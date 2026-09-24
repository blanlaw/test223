# Análisis del proyecto TMS (Vanguard) y requisitos para convertirlo en un SaaS

> **Alcance:** análisis de solo lectura de `vanguard-tms-backend-main` (.NET 8) y `vanguard-tms-frontend-main` (Angular 15).
> **Fecha:** 2026-09-23
> **Objetivo:** documentar qué hace hoy el sistema (módulos, integraciones, configuración, tablas, flujos) y qué hace falta para reconstruirlo como SaaS multi-empresa, en otro entorno y con otras tecnologías.
>
> Este documento **no reproduce ningún secreto**. Solo indica dónde están.

---

## Índice

1. [Resumen ejecutivo](#1-resumen-ejecutivo)
2. [Qué es el producto (dominio)](#2-qué-es-el-producto-dominio)
3. [Inventario técnico actual](#3-inventario-técnico-actual)
4. [Arquitectura y flujo de datos](#4-arquitectura-y-flujo-de-datos)
5. [Modelo de datos (todas las tablas)](#5-modelo-de-datos-todas-las-tablas)
6. [Tablas maestras y configuración funcional (CONFIG)](#6-tablas-maestras-y-configuración-funcional)
7. [Backend: API y endpoints](#7-backend-api-y-endpoints)
8. [Lógica de negocio clave](#8-lógica-de-negocio-clave)
9. [Integraciones externas](#9-integraciones-externas)
10. [Procesos en segundo plano (jobs y colas)](#10-procesos-en-segundo-plano)
11. [Frontend](#11-frontend)
12. [Seguridad, roles y permisos](#12-seguridad-roles-y-permisos)
13. [Configuración y despliegue](#13-configuración-y-despliegue)
14. [Bugs y deuda técnica detectados](#14-bugs-y-deuda-técnica-detectados)
15. [Lógica fijada en el código para Vanguard](#15-lógica-fijada-en-el-código-para-vanguard)
16. [Qué debes tener en cuenta para el SaaS](#16-qué-debes-tener-en-cuenta-para-el-saas)
17. [Stack propuesto para la nueva versión](#17-stack-propuesto-para-la-nueva-versión)
18. [Modelo de datos objetivo (SaaS)](#18-modelo-de-datos-objetivo-saas)
19. [Funcionalidad faltante (backlog de producto)](#19-funcionalidad-faltante)
20. [Hoja de ruta sugerida](#20-hoja-de-ruta-sugerida)
21. [Checklist de migración de datos y funcionalidades](#21-checklist-de-migración)
22. [Acciones urgentes en el sistema actual](#22-acciones-urgentes-en-el-sistema-actual)

---

## 1. Resumen ejecutivo

- **Qué hace hoy.** Es un TMS de **monitoreo de órdenes de transporte (OT) de contenedores para agroexportación en Perú**. El recorrido de una OT es: almacén de vacío → garita → packing → puerto. Las OT se importan desde el ERP **Nisira** y se asignan a un transportista, un vehículo (tracto y carreta) y un chofer. Se envían a un **proveedor GPS (Navitel)**, que devuelve posiciones "consolidadas" con la geocerca actual, el siguiente hito, la distancia y el ETA. El sistema muestra el avance en tiempo real (lista y mapa en Google Maps), el semáforo de cumplimiento de citas y los reportes (Excel/CSV por correo), y guarda la auditoría.
- **Cómo está construido.**
  - Backend: 1 API principal y 3 microservicios (Receptor, ProcesadorReceptor y TareasMantenimiento).
  - 3 servicios **simulados** del proveedor GPS (`PROVGPS.*`) y 2 herramientas de prueba.
  - Datos: MySQL 8, Redis, RabbitMQ, un servidor Socket.IO externo (no está en el repo) y Seq para logs.
  - Frontend: Angular 15 sobre la plantilla Velzon.
- **Nivel de "SaaS" actual: ninguno.** Hay **un despliegue y una base de datos por cliente** ("inquilino"). `TenantId` es un único valor de configuración y ninguna tabla tiene columna de tenant. El único aislamiento interno es por *Operador Logístico (OPL)*, y está aplicado de forma parcial.
- **Riesgos serios:**
  - Contraseñas en MD5 sin sal.
  - Inyección SQL en `ORDER BY`.
  - Secretos expuestos por la API (`/Cache/config`).
  - La misma clave JWT en todos los servicios.
  - Endpoints de prueba (`DevTest`) y Swagger activos en producción.
  - RabbitMQ configurado de forma que **pierde mensajes**.
  - Una fecha fija en el frontend (`2026-12-31`) que **romperá la asignación de OT a partir del 1 de enero de 2027**.
- **Recomendación.** Reescribir sobre un **monolito modular multi-tenant**. Conservar como activo el **conocimiento del dominio** (el modelo OT → ruta de geocercas → hitos → semáforo) y los contratos de integración, no el código.

---

## 2. Qué es el producto (dominio)

| Concepto | Significado en el sistema |
|---|---|
| **OT** (Orden de Transporte) | Pedido comercial que viene del ERP (serie-número, booking, cliente final, fecha de cita, sucursal, OPL, naviera). Es la unidad que se monitorea. |
| **OPL** (Operador Logístico) | Empresa que opera la OT, por ejemplo un agente de aduanas. Tiene usuarios propios (rol 50) y solo debe ver sus OT. Cada OPL apunta a un proveedor GPS. |
| **Transportista / Vehículo / Chofer** | Maestros de flota. El vehículo es *tracto* (001) o *carreta* (002) y tiene IMEI del GPS. Se comparten entre OPL mediante la lista `OLids`. |
| **Geocerca** | Polígono o círculo: almacén de vacío, almacén de retorno, packing, puerto, descanso, etc. |
| **Ruta** | Secuencia ordenada de geocercas (`RutaPos`) con tipo (INICIO / INTERMEDIO / FIN), dirección (IDA / VUELTA), hito, tipo de control y un flag "dinámico". **No es una polilínea vial.** |
| **Hito** (`OTHito`) | Punto de control de la ruta con hora planificada (la cita), ETA, hora real y semáforo (rojo / amarillo / verde). |
| **Proveedor GPS** | Empresa externa (Navitel, Comsatel, Protegecorp…) que recibe la OT con su ruta y devuelve posiciones con los cálculos ya hechos. |
| **Receptor** | API del TMS a la que el proveedor GPS envía la "data consolidada". |
| **Packing list** | Datos de planta: inspección SENASA, precintos, carguío, rampa, nave. Se cruzan con la OT para los KPI de estadía. |

**Ciclo de vida de una OT:**

```
ERP Nisira ──importar──► INACTIVA ──► ACTIVA (visible para el OPL) ──► ASIGNADA (vehículo + chofer + ruta)
                                                                        │  └─► se envía al proveedor GPS (crear OT + inicio)
                                                                        ▼
                                                  posiciones GPS → hitos (tin / ETA / semáforo)
                                                                        ▼
                                          FINALIZADA / REPROGRAMADA / CANCELADA ──► fin del envío al proveedor GPS
```

---

## 3. Inventario técnico actual

### 3.1 Backend (`vanguard-tms-backend-main`, solución `TMS.sln`, .NET 8)

| Proyecto | Tipo | Función | Puerto dev |
|---|---|---|---|
| `TMS.API.Backend` | Web API | API principal de la webapp: CRUD, OT, reportes, caché, archivos | 9014 |
| `TMS.API.Receptor` | Web API | Recibe la data consolidada del proveedor GPS (JWT), la registra y la encola | 9013 |
| `TMS.API.ProcesadorReceptor` | Worker y API | Consume la cola y actualiza OT, hitos, cálculos, historial, Redis y socket | 9016 |
| `TMS.API.TareasMantenimiento` | Worker y API | Scheduler, notificaciones por email, reportes, purgas y sincronizaciones (ERP, packing, GPS) | 9015 |
| `TMS.Shared` | Librería | Entidades, DTOs, repositorios (EF y Dapper), servicios, utilidades y settings | — |
| `PROVGPS.I1` | Web API (**mock**) | Simula el colector de tramas GPS del proveedor | 9023 |
| `PROVGPS.I3` | Web API (**mock**) | Simula la API del proveedor GPS que llama el TMS (auth, geocercas, OT, inicio y fin) | 9025 |
| `PROVGPS.ProcesadorColector` | Worker (**mock**) | Simula el motor del proveedor: punto en polígono, ETA, tramas → Receptor | 9024 |
| `TMS.Tester.WinApp` | WPF | Simulador de vehículos (CSV → I1) y pruebas de carga | — |
| `TMS.Tester.DevTests` | Consola | Experimentos (no son tests automatizados) | — |

**Infraestructura** (en el repo **no hay docker-compose**):

| Servicio | Puerto dev |
|---|---|
| MySQL 8.4 | 9010 |
| Redis | 9021 |
| RabbitMQ | 9017 |
| Seq | 9020 / 8081 |
| Socket Server (Node Socket.IO, **no incluido**) | 9012 |
| Servicio `api-integracion-sql:5000` (packing list, **no incluido**) | — |

**Paquetes NuGet principales:**
- Datos: EF Core 8.0.8 + Pomelo MySQL 8.0.2, Dapper 2.1.35, DbUp 5.
- Autenticación: JwtBearer 8.
- Caché: StackExchange.Redis.
- Mensajería y tiempo real: RabbitMQ.Client 6.8.1, SocketIOClient 3.1.2.
- Logging: Serilog con Seq.
- Archivos y correo: ClosedXML (Excel), CsvHelper, MailKit.
- Otros: AutoMapper 13, ObjectsComparer, Humanizer, Swashbuckle, HealthChecks (MySQL / Redis).

### 3.2 Frontend (`vanguard-tms-frontend-main`)

- **Base:** Angular 15.0 y TypeScript 4.8, sobre la **plantilla Velzon** (Bootstrap 5.3, ng-bootstrap 14). El `package.json` sigue llamándose `velzon`.
- **Mapas:** `@angular/google-maps`, con la API de Google Maps cargada en `index.html` (drawing, marker). Leaflet está instalado pero no se usa.
- **Librerías de la aplicación:**
  - tiempo real: `socket.io-client` 4.7;
  - Excel: `xlsx` / `xlsx-js-style` (solo exporta);
  - UI y formularios: `sweetalert2`, `ng-select`, `flatpickr`, `ngx-mask`;
  - fechas: `moment`, `humanize-duration`;
  - traducciones: `ngx-translate`, que en la práctica no se usa.
- **Configuración en tiempo de ejecución:** `src/assets/config/appconfig.json` (`urlApi`, `urlRt`, `urlRtPath`, `tenantId`, `tenantName`, `pageSize`…). **`tenantId` no se usa.**
- **Despliegue:** Docker (node:latest → nginx:latest, puerto 9011). `dockerfile-dev` está roto.
- **Tamaño:** unas **17 pantallas reales**. El resto son demos de Velzon y unas 25 dependencias sin uso.

---

## 4. Arquitectura y flujo de datos

```
                ┌──────────────────────────── WEBAPP (Angular) ────────────────────────────┐
                │ REST (JWT en header)                              Socket.IO (server_send_ots)│
                └───────┬─────────────────────────────────────────────────────▲────────────┘
                        ▼                                                     │
              ┌───────────────────┐   HTTP /tarea    ┌──────────────────────┐ │
  ERP Nisira◄─┤  TMS.API.Backend  ├─────────────────►│ TareasMantenimiento  │ │
  (HTTP)      │   :9014           │                  │ jobs + cola memoria  │─┼─► SMTP (reportes, notificaciones)
              └──┬──────┬─────────┘                  └──┬───────────────────┘ │   ERP Nisira / PackingList / APIs GPS
                 │      │ URL_601..606 (auth, geocerca, │                     │
                 │      │ OT crear, inicio, fin)        │                     │
                 │      ▼                               │                     │
                 │  ┌─────────────────────┐             │             ┌───────┴──────┐
                 │  │ PROVEEDOR GPS       │             │             │ Socket Server│
                 │  │ (Navitel real /     │             │             │  :9012 (ext) │
                 │  │  mocks PROVGPS.*)   │             │             └───────▲──────┘
                 │  └────────┬────────────┘             │                     │ client_broadcast_updatedots
                 │           │ POST /auth/login + /Consolidated (JWT)         │
                 │           ▼                          │                     │
                 │  ┌─────────────────────┐  RabbitMQ   │   ┌─────────────────┴────┐
                 │  │  TMS.API.Receptor   ├──queue.receptor─►│ ProcesadorReceptor   │
                 │  │  :9013              │             │   │ :9016                │
                 │  └─────────────────────┘             │   └──────────┬───────────┘
                 ▼                                      ▼              ▼
          ┌─────────────────────── MySQL "tms" (1 BD por cliente) + Redis (caché) ───────────────────────┐
          └──────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Puntos clave del flujo GPS:**

1. Al pasar la OT a **ASIGNADA**, el Backend envía al proveedor GPS `orderid`, `tenantid`, `receiverurl`, `vid` (IMEI), `plate`, `rid1` (serie-número) y `route[]` (las geocercas en orden). Después llama a *start*.
2. Las geocercas se envían al proveedor cada vez que se crean o editan, **solo al primer proveedor activo**.
3. **El proveedor es quien calcula** la geocerca actual, el siguiente hito, la distancia y el ETA. El TMS solo recibe y guarda.
4. El proveedor hace `POST /Consolidated` con este cuerpo:
   - cabecera: `provgpsid`, `tenantid`, `seq`, `t`;
   - `data[]`, con uno o más puntos por OT (`vid`, `plate`, `orderid`, `lat`, `lon`, `vel`, `alt`, `tgps`, `tsend`, `geofence{id,tin,tout}`, `calcs{geofenceid,dir,type,d,et}`).
5. El Receptor guarda la petición en `HistoricRequest` y en un archivo JSON, la encola y responde 200.
6. ProcesadorReceptor:
   - actualiza `OT.RPointsJson` (los `tin` de cada punto), `OTCalcs1` (la última posición), `OTHito` (FHReal, FHEta, semáforo `i01` / cumplimiento `i03`) e inserta en `OTPosHistorial`;
   - finaliza la OT si llegó al fin;
   - recalcula la caché de tiempo real en Redis y emite al socket.
7. El frontend recibe `server_send_ots` y **vuelve a pedir todo por REST** (`cache/ottiemporeal`, `cache/otfinalizadas`, `cache/ottiemporealmapa`).
8. Además, TareasMantenimiento consulta cada 30 s otras dos API del proveedor: **PuntoRutaMulti** (eventos inzone/outzone) y **VanguardStatus** (estado de movimiento y última transmisión).

---

## 5. Modelo de datos (todas las tablas)

**Motor:** MySQL 8 (utf8mb4_0900_ai_ci). **Migraciones:** DbUp embebido al arrancar la API (`TMS.API.Backend/DBMigrations/001`, `002`; el `003` **no está embebido**).

**Convenciones comunes** (`AuditableEntity`), en casi todas las tablas:

| Columna | Significado |
|---|---|
| `cid` | creado por |
| `uid` | actualizado por |
| `isdel` | borrado lógico |
| `cat` | creado en (unix ms, BIGINT) |
| `uat` | actualizado en (unix ms, BIGINT) |

- Las PK son **VARCHAR(36) con GUID**. No existe **ninguna foreign key** declarada.
- Muchas tablas tienen `AggregatedData` (JSON) y `Notes`.

> ⚠️ **Desfase entre esquema y código.** Las entidades C# tienen columnas que **no existen en los scripts SQL**. Por tanto, la BD de producción fue modificada a mano y **los scripts no reproducen la BD real**. Antes de migrar, saca un `mysqldump --no-data` de producción. Columnas detectadas:
> - `OT`: `Chofer2Id`, `PGPS_Calculos1`, `PGPS_Calculos2`, `HistoricSerie`, `SerieNumeroUltimo`, `Reprogramado`, `EncFrio`, `FechaFrio`, `IncPacking`.
> - `Chofer`, `Vehiculo`, `Transportista`: `OLids` (JSON).
> - `Ruta`: `TipoRuta`.
>
> Además, las tablas `PackingList`, `PuntoRutaMulti`, `VanguardStatus`, `ERPNisiraOTSnapshot` y `HitoricDeleteVTC` no tienen DDL en el repo, y el `003` define `OTPosHistorial` con columnas distintas a las de la entidad.

### 5.1 Seguridad y usuarios

| Tabla | PK | Columnas relevantes | Notas |
|---|---|---|---|
| `AppRole` | `AppRoleId` INT | Name, IsActive, IsSuperAdmin, RoleInfo | Seeds: 5 SERVICE, 10 SUPER-ADMIN, 20 ADMINISTRADOR-TMS, 30 ADMINISTRADOR, 40 MONITOR, 50 OPERADOR-LOGISTICO, 100 RECEPTOR-TMS, 200 VISOR. El código usa además **210, 220 y 230**, que no están en los seeds. |
| `AppUser` | `AppUserId` | AppRoleId, UserName, **HashedPassword (MD5)**, FirstName, LastName, FN, IdentificationNumber, Email, IsActive, **OperadorLogisticoId**, ResetGuid, ResetGuidValidUntil, LastLogin | Un usuario tiene un rol y, opcionalmente, un OPL. |
| `HistoricLogin` | BIGINT (unix ms) | FechaHora, usuario, rol, OPL, Resultado, ErrorMessage | Logins de la webapp. |
| `HistoricLoginReceptor` | BIGINT | ídem | Logins de los proveedores GPS en el Receptor. |
| `Person` | GUID | FirstName, LastName, Payment | **Tabla de demo de la plantilla.** Eliminar. |

### 5.2 Maestros de negocio

| Tabla | Columnas relevantes |
|---|---|
| `OPL` | OPLId, Nombre, **ProveedorGPSId** (se importa desde el ERP) |
| `Cliente` | ClienteId, Nombre (cliente final, del ERP) |
| `Sucursal` | SucursalId, Nombre (del ERP) |
| `Transportista` | RazonSocial, NroIdentificacion (RUC), GerenteGeneral, contactos, teléfonos, emails, dirección, Estado, `OLids` |
| `Chofer` | IsTemp, Nombres, Apellidos, FechaNacimiento, TipoDoc, NroIdentificacion, CatBrevete, NroBrevete, VctoBrevete, Estado, `OLids` |
| `Vehiculo` | IsTemp, Estado, **NroPlaca**, **IMEIGps**, Propietario, TipoVehTMS (001 tracto / 002 carreta), CatVehicular, TipoCarroceria, ColorBase, TipoTransm, TipoCombust, Marca, Modelo, AnioFab, VIN, serie, motor, pesos, carga útil, puertas, ruedas, ejes, cilindros, dimensiones, **VctoRevTecnica**, `OLids` |
| `ProveedorGPS` | Nombre, IsActive, **LogicaId** (conector), contactos, **AuthPayload** (credenciales en claro), `URL_001_COLECTOR`, `URL_601_AUTH`, `URL_602_GEOCERCA_CREARMODIFICAR`, `URL_604_OT_CREARMODIFICAR`, `URL_605_OT_INICIO`, `URL_606_OT_FIN`, **LastToken**, Instrucciones |
| `TablasMaestras` | PK compuesta (TableName, Code), Value1–3, ParentTableName, ParentCode, Enabled, Order, TMAdd, TMEdit. Guarda **catálogos y también la configuración del sistema**. |

### 5.3 Geografía y rutas

| Tabla | Columnas relevantes |
|---|---|
| `Geofence` | Status, **Sent** (enviada al proveedor GPS), Name, **ShortName** (único), Address, Lat, Lng, Alt, **Rad**, **Type** (POLYGON / CIRCLE), CatCode (GEOCERCACAT), **JPoints** (JSON de vértices) |
| `Ruta` | Status, IsDefault, Name, GeofenceCount, (TipoRuta) |
| `RutaPos` | PK (RutaId, Pos), Tipo (INICIO_RUTA / PUNTO_INTERMEDIO / FIN_RUTA), Direccion (IDA / VUELTA), Calcular, GeofenceId, Categoria, NombreHito (H001 Alm. vacío, H002 Packing, H003 Puerto…), TipoControlHito (TC001 cita alm. vacío, TC002 cita packing, TC003 cita puerto), **Dinamico** (se reemplaza por el almacén de retiro o retorno de la OT) |
| `GeoUbigeo` | Ubigeos de Perú con geometría (se usa `ST_Contains` para saber el distrito de una posición) |

### 5.4 Operación (OT y tracking)

| Tabla | Descripción |
|---|---|
| `OT` | **Tabla central.** Contiene: <br>• **Datos del ERP:** ERPId (idunico), IdDocumento, Serie, Numero, SerieNumero, Fecha, SucursalId, Sucursal, NBooking, FechaCita, ClienteId, Cliente, OperadorLogId, OperadorLog y Naviera. <br>• **Estado:** ACTIVA / INACTIVA / ASIGNADA / REPROGRAMADA / CANCELADA / FINALIZADA. <br>• **Fechas:** FHCarga, FHPosicionInicial (cita packing), FHDespacho, FHRecojoContenedor, FHRetiroContenedor, FHEntregaContenedor, FHPuertoDescarga, FHRutaInicio, FHRutaFin y FHFinalizada. <br>• **Asignación:** TransportistaId, Vehiculo1Id (tracto), Vehiculo2Id (carreta), ChoferId, AlmacenRetiroGeofenceId, AlmacenRetornoGeofenceId, NContenedor y RutaId. <br>• **Ruta y estado GPS:** **RPointsJson** (copia de la ruta con el `tin` de cada punto), PGPS_UltAct y PGPS_UltError. <br>• **Notas:** Notes (del ERP) y Notes2 (del OPL). |
| `OTHito` | PK (OTId, Pos). Copia del punto de ruta con FHPlan1 (cita), FHEta, FHReal, `i01` (semáforo: 1 rojo, 2 amarillo, 3 verde), `i03` (cumplió o no) y HasData. |
| `OTCalcs1` | 1:1 con la OT: última posición (lat, lng, vel, alt, ts), última geocerca y su hora de ingreso, geocerca destino, distancia en km y minutos al destino, dirección, LastPos y TotalPos (avance). |
| `OTPosHistorial` | Una fila por punto GPS recibido (time series), con índice (OTId, TProc). **Sin purga implementada.** Es la tabla que más crece. |
| `PuntoRutaMulti` | Eventos inzone/outzone por geocerca, traídos de la API del proveedor cada 30 s (Plate, OrderId, Rid, GeofenceName, Type, Time, Lat, Lng). No tiene índice único. |
| `VanguardStatus` | Estado del vehículo según el proveedor (LastTransmitionTime, MovementStatus, GpsUpdated). El upsert por OrderId **duplica las filas con OrderId nulo cada 30 s**. |
| `packinglist` | Datos de planta por `nro_pedido`: estado, ronda, rampa, nave, naviera, puerto destino, inspector y precinto SENASA, y unas 15 marcas de tiempo (ingreso a planta, conformidades, carguío, entrega al chofer, límite de salida…). |
| `ERPNisiraOTSnapshot` | Snapshot de las OT del ERP (idunico como PK), CamposCompletos, **FechaProgramada** (historial JSON de cambios de fecha) y FechaSincronizacion. |

### 5.5 Archivos y auditoría

| Tabla | Descripción |
|---|---|
| `FileGroup` / `FileItem` | Metadatos de los archivos: TableName, TableId, Tag1, Tag2, OriginalName, DestinationName, SizeInMB y MimeType. Los archivos se guardan en **disco local**, en `uploads/common`. |
| `HistoricChange` | Diff JSON por entidad (TypeId 1 crear, 2 modificar, 3 eliminar, 10 contraseña, 20 archivo). La PK es unix ms, lo que provoca colisiones. |
| `HistoricRequest` | Cada llamada externa (ERP, GPS, Receptor): URL, verbo, origen, parámetros, estado, duración, número de registros y un archivo JSON con el cuerpo. **Guarda también las credenciales en `Parametros`.** |
| `HitoricDeleteVTC` | Snapshot JSON de vehículos, choferes y transportistas eliminados. El nombre tiene una errata ("Hitoric"). |
| Histórico de OT eliminadas | Lo genera el stored procedure `EliminarOT`. |

**Stored procedures:** `EliminarOT(p_SerieNumero)` y `ObtenerOTDatos(p_SerieNumero)`.

---

## 6. Tablas maestras y configuración funcional

`TablasMaestras` mezcla **catálogos** y **configuración del sistema**:

| TableName | Filas | Uso |
|---|---|---|
| `CONFIG` | 53 | Configuración global (ver detalle abajo) |
| `TAREASPROGRAMADAS` | 8 | Jobs en JSON (`CodigoTarea`, `Activo`, `EsRecurrente`, `Fijo_HoraLocal`, `Rec_FrecuenciaMin`, `Emails`) |
| `GEOCERCACAT` | 7 | GC010 Alm. de vacío, GC020 Alm. de retorno, GC030 Descanso, … |
| `NOMBREHITO` | 5 | H001 Alm. vacío, H002 Packing, H003 Puerto, H999 Otros |
| `TIPOCONTROLHITO` | 5 | TC001 / TC002 / TC003: fechas de cita que controla cada hito |
| `RUTAPUNTOTIPO` / `RUTAPUNTODIR` | 3 / 2 | INICIO / INTERMEDIO / FIN — IDA / VUELTA |
| `TIPODOCUMENTO` / `TIPOBREVETE` | 5 / 10 | DNI, CE, pasaporte… / categorías de licencia en Perú |
| `VEHCAT`, `VEHTIPOCARR`, `VEHCOLORBASE`, `VEHTIPOCOMBUST`, `VEHTIPOTRANSM`, `VEHTIPOTMS` | 67 / 96 / 21 / 21 / 4 / 2 | Catálogos vehiculares (MTC Perú) |

**Claves de `CONFIG`** (solo los nombres):

| Grupo | Claves |
|---|---|
| ERP | `ERP_NOMBRE`, `ERP_001_AUTH`, `ERP_001_AUTH_PAYLOAD` ⚠️ credencial, `ERP_607_ORDENES`, `ERP_608_CLIENTES`, `ERP_609_EMPRESASHIJAS`, `ERP_610_OPLOGISTICOS`, `ERP_PACKINGLIST_URL` |
| APIs GPS extra | `API_PUNTORUTAMULTI_URL`, `API_VANGUARDSTATUS_URL`, `API_EXT_REINTENTOS` |
| Email | `EMAIL_HOST`, `EMAIL_PORT`, `EMAIL_PASSWORD` ⚠️, `EMAIL_AUTENTICADO`, `EMAIL_SENDEREMAIL`, `EMAIL_SENDERNAME`, `EMAIL_SUBJECT_PREFIX`, `ERROR_EMAILS`, `ENVIARNOTIFICACIONES`, `ENVIARNOTIFICACIONESERROR` |
| Notificaciones | `NOTIF_{USER,TRANSPORTISTA,CHOFER,VEHICULO,CARRETA,PROVEEDORGPS,OT}_{NEW,EDIT,...}`, `NOTIF_VEHICULO_REVISIONTECNICA`, `NOTIF_OT_IMPORT`, `NOTIF_OT_ESTADOMASIVO_UPDATE`. El valor son los roles destinatarios, por ejemplo `30_40` o `30_SELF`. |
| Mapa | `MAPA_GOOGLE_ID`, `MAPA_INIT_LAT`, `MAPA_INIT_LNG`, `MAPA_INIT_ZOOM`, `MAPA_COLOR_OTREAL_{ETA_OK,ETA_WARNING,ETA_PROBLEM,DIR_IDA,DIR_VUELTA}` |
| Tiempo real | `OT_TREAL_SEMAFORO_MINUTOS` (30), `OT_TREAL_FINALIZADAS_HORAS` (12) |
| Retención | `REMOVE_HISTORIC_{LOGINWEB,LOGINRECEPTOR,REQUESTS}_DAYS` (15), `REMOVE_HISTORIC_OTPOSHISTORIAL_DAYS` (90, **no implementado**) |
| Jobs | `TAREASPROGRAMADAS_FREQ_MINS`, `JOB_FECHA_PROGRAMADA_FREQ_MINS`, `JOB_PACKING_LIST_FREQ_MINS`, `JOB_PUNTORUTAMULTI_FREQ_SECS`, `JOB_VANGUARDSTATUS_FREQ_SECS` |
| Otros | `INQUILINO_WEBURL` |

> ⚠️ Los seeds SQL incluyen **credenciales reales o de prueba** (contraseña SMTP y token del ERP). Hay que rotarlas y sacarlas del repo.

---

## 7. Backend: API y endpoints

Todas las rutas tienen la forma `/{Controller}`. Todo exige JWT salvo lo indicado. Los roles se expresan como IDs numéricos en `[Authorize(Roles="…")]`.

| Controller | Endpoints principales | Roles |
|---|---|---|
| `Auth` | `POST /Auth/login` (anónimo) | — |
| `AppUser` | CRUD, `PUT /{id}/newpassword`, `GET historiclogin`, `GET historicloginreceptor` | 10, 20, 30 |
| `AppRole` | `GET`, `GET /{id}` | 10, 20, 30 |
| `TablasMaestras` | `GET`, `GET /{t}/{c}`, `POST`, `PUT /{t}/{c}` | 10, 20, 30 |
| `ProveedorGPS` | `GET`, `GET /{id}`, `POST`, `PUT` (devuelve credenciales) | 10, 20, 30 |
| `Transportista` | CRUD, `report`, `resumen` | 10–50, 210–230 |
| `Chofer` / `Vehiculo` | `GET`, `gettemporal` (borrador), `PUT` (guardar), `DELETE` (con snapshot), `update-masivo`, `delete-masivo`, `report`, `resumen` | 10–50, 200–230 |
| `Geofence` | CRUD (sincroniza con el proveedor GPS), `GPX`, `uisearch`, `getallforproveedorgps` | 10–50, 220, 230 |
| `Ruta` | CRUD, `/{id}/GPX`, `/{id}/getallforproveedorgps` | 10–50, 220, 230 |
| `TempOT` | `GET` (OT del ERP frente a la BD), `filtrado`, `/{serieNumero}`, `POST` (importar), `copy`; `integracion/*` (anónimo con **X-Api-Key**) | 10–50, 220, 230 |
| `OT` | `GET` (filtrado por el OPL del token), `/{id}`, `PUT /{id}` (máquina de estados), `actualizar-estado` (masivo), `actualizar-obs`, `update-masivo`, `update-rpoints`, `refreshcache-ottreal`, `OTConHitos`, `DELETE /delete/{id}` (SP), `historic-delete`; `integracion/getall` e `integracion/tracking` (**X-Api-Key**) | varía según el método |
| `OTPosHistorial` | `GET` (agrupado y enriquecido con ubigeo y geocerca), `/{otId}` | 10–50, 220, 230 |
| `Cache` | Unos 25 GET: catálogos para combos, `ottiemporeal`, `otfinalizadas`, `ottiemporealmapa`, `ottrackingtiemporeal`, `tracking`, `retransmisionsemanal`, `unidades-cerca-packing`, `programa-embarques`, `actividades-camion-dentro-packing`, `citarecojovacio`, `citapuerto`, `ncontenedor`, `cumplimientocita`, `retrasocita`, `avancepedidos`, `encendidofrio`. **`/Cache/config` expone secretos.** | casi todos |
| `Report` | `POST R01` (alertas, Excel), `R02` (análisis, CSV), `R03` (alertas de despacho, Excel). Se ejecutan en TareasMantenimiento y se envían por email. | 10–40 |
| `FileManager` | `uploadsinglefile`, `GET /{id}`, `DELETE /{id}`, `groups` | casi todos |
| `HistoricChange`, `HistoricRequest`, `HitoricDeleteVTC`, `GeoUbigeo` | Consultas de auditoría y ubigeo | varía |
| `DevTest` | `configrapida` (**reescribe URLs de integración**), `simulacion1`, `createtestdata`, `uploadfileforprocessing` (CSV de vehículos y choferes) | 10–30, **activo en producción** |
| `Person`, `Ping`, `/health`, `/swagger` | Demo / sondeo / salud / documentación (Swagger **siempre activo**) | — |

**Receptor:**
- `POST /Auth/login` (MD5, rol 100, JWT de 30 min).
- `POST /Consolidated` (JWT, rate limit **global** de 20 peticiones por minuto).

**TareasMantenimiento:**
- `POST /Tarea` (**anónimo**).
- `/Person/*` (demo).

---

## 8. Lógica de negocio clave

### 8.1 Importación desde el ERP
- `GET TempOT` llama al ERP Nisira: primero `POST ERP_001_AUTH` para obtener el token y luego `GET ERP_607_ORDENES` **con body JSON**. Consulta 30 días hacia atrás y compara con la BD por `SerieNumero`.
- `POST TempOT` ejecuta `ImportarOTs`:
  - crea las OT en estado INACTIVA;
  - hace upsert de OPL (asignándoles el proveedor GPS por defecto), Cliente y Sucursal;
  - si el flete es aéreo (`idflete="002"`), hace upsert del Transportista.
- Reimportar solo se permite si la OT está ACTIVA o INACTIVA, y la devuelve a INACTIVA.

### 8.2 Máquina de estados de la OT (`OTService.Update`)

| Destino | Desde | Validaciones y efectos |
|---|---|---|
| INACTIVA | INACTIVA, ACTIVA, ASIGNADA, FINALIZADA | Recalcula la ruta. |
| ACTIVA | ídem | Exige un OPL con proveedor GPS activo y con credenciales. |
| ASIGNADA | ACTIVA, ASIGNADA, FINALIZADA | Exige además un tracto ACTIVO con IMEI. **Envía la OT y el inicio al proveedor GPS**, refresca la caché y notifica por socket. |
| REPROGRAMADA / CANCELADA / FINALIZADA | cualquiera | Fija `FHFinalizada` y envía el fin al proveedor GPS. **No hay salida de REPROGRAMADA ni de CANCELADA.** |

- Reabrir una OT FINALIZADA exige el rol 30. Es un bug: el mensaje de error dice "10".
- Cada cambio de la fecha de cita incrementa `Reprogramado` (R → 2R → 3R).
- **No es transaccional.** Si falla el proveedor GPS, el cambio de estado queda guardado de todas formas.

### 8.3 Ruta → hitos
- Al asignar la ruta a una OT, `RutaPos` se **copia** a `OT.RPointsJson`, y se crean `OTHito` para los puntos con `Calcular=true`.
- Los puntos dinámicos se sustituyen así: INICIO con TC001 toma el almacén de retiro de la OT, y FIN con TC003 toma el almacén de retorno.
- Hora planificada de cada hito:
  - TC001 = `FHRecojoContenedor`
  - TC002 = `FHPosicionInicial`
  - TC003 = `FHEntregaContenedor`
- Semáforo: compara el ETA con la hora planificada usando `OT_TREAL_SEMAFORO_MINUTOS`.
- Cambiar la ruta **borra los `tin`** ya registrados.

### 8.4 Dashboards y KPI (en `CacheService`)
- **Estado TMS** según los `tin` de las posiciones 1 a 5: por ingresar a alm. vacío → en ruta → en planta → en ruta a puerto → llegó a puerto.
- **Cumplimiento de cita:** hora de garita frente a la cita (con la cita −5 h fijadas en el código).
- **Estadía en packing:** base de 6 h si el OPL es MAERSK o TPP y la ruta contiene "PDP"; 8 h en los demás casos. Si se supera, marca "URGENTE DAR SALIDA".
- **Tiempos de planta** calculados desde el packing list: inspección, carguío, SENASA y guía.
- Unidades cerca de packing (distritos de Ica fijados en el código), retransmisión GPS por OPL, encendido de frío, avance de pedidos.

### 8.5 Reportes
- **R01** Alertas: Excel con ClosedXML, una hoja por número de hitos, celdas coloreadas por semáforo.
- **R02** Análisis: OT con sus hitos en CSV, máximo 32 días.
- **R03** Alertas de despacho: OT cuyo hito de packing no tiene hora real.
- Se generan en TareasMantenimiento y se envían por email al usuario que los pide o, si son programados, a la lista de correos configurada.

### 8.6 Maestros de flota
- **Patrón "borrador":** `gettemporal` crea el registro con `IsTemp=1`, y `PUT` lo consolida.
- **Deduplicación:** vehículo por placa + IMEI; chofer por DNI + brevete; transportista por RUC. Si ya existe, se agrega el OPL a `OLids` en lugar de duplicar el registro.
- **Borrado:** lógico, con snapshot en `HitoricDeleteVTC`.
- **Documentos:** los adjuntos (FileManager con "updaters") recalculan `AggregatedData`.

---

## 9. Integraciones externas

| Integración | Dirección | Mecanismo | Configuración | Observaciones |
|---|---|---|---|---|
| **ERP Nisira** (Perú) | TMS → ERP | HTTP: auth con token y luego GET con body | `CONFIG.ERP_*` | Trae OT, OPL, clientes, sucursales y "empresas hijas". El payload de autenticación queda en logs y en `HistoricRequest`. Sin capa de abstracción. |
| **Proveedor GPS** (Navitel; también se mencionan Comsatel y Protegecorp) | TMS → GPS | HTTP 601 auth, 602 geocerca, 604 crear/modificar OT, 605 inicio, 606 fin | Tabla `ProveedorGPS` | Token cacheado en `LastToken`, reintentos (`API_EXT_REINTENTOS`) y reautenticación ante 401. |
| **Proveedor GPS → Receptor** | GPS → TMS | `POST /auth/login` y `POST /Consolidated` | Usuario con rol 100 | Contrato "Interface 2". Autenticación con usuario y contraseña en MD5. |
| **Proveedor GPS: APIs extra** | TMS → GPS (polling) | GET cada 30 s | `API_PUNTORUTAMULTI_URL`, `API_VANGUARDSTATUS_URL` | Usan el token del `DefaultProveedorGPSId`. Solo soportan un proveedor. |
| **Packing List** | TMS → servicio SQL | GET sin autenticación | `ERP_PACKINGLIST_URL` (por defecto `api-integracion-sql:5000`) | Puente a la BD de planta. Fechas parseadas con cultura es-PE. |
| **SMTP** | TMS → correo | MailKit | `CONFIG.EMAIL_*` | Acepta cualquier certificado TLS. Envía contraseñas en claro. |
| **Socket.IO** | Backend/Procesador → servidor → webapp | WebSocket | `SocketServer_Port` | El servidor Node **no está en el repo**. |
| **Google Maps** | Webapp | JS API (drawing, marker, AdvancedMarker) | Key en `index.html`, `MAPA_GOOGLE_ID` | El coste escala con el número de tenants. |
| **API de integración** (terceros) | Terceros → TMS | `X-Api-Key` estática | `IntegrationApiKey` | Lectura de OT y tracking, e **importación**. Los endpoints de importación probablemente fallan (NullReference). |
| **Seq** | Logs | Serilog | `serverUrl` | — |

**Contratos que hay que conservar** (DTOs en `TMS.Shared/DTOs/Queues/*`):
- `OTItemForProveedorGPSDto`: `orderid`, `tenantid`, `receiverurl`, `senddata`, `start`, `end`, `rid1..3`, `vid`, `plate`, `route[]{i, geofenceid, dir, type, calculate}`.
- `GeofenceItemTransferDto`.
- `ReceptorDataConsolidadaRequestDto`: `provgpsid`, `tenantid`, `seq`, `t`, `data[]` (con `geofence` y `calcs`).
- `ColectorVehicleGPSRequestDto` (formato de retransmisión Navitel V2).

---

## 10. Procesos en segundo plano

### Colas RabbitMQ
- Se usa el exchange por defecto. Hay dos colas: `queue.receptor` y `queue.mockcolector`.
- **No son durables**, los mensajes no son persistentes, no hay publisher confirms, no hay DLX ni reintentos, y se aplica `Nack(requeue:false)` → **cualquier error transitorio descarta posiciones**.
- El canal (`IModel`) se comparte entre hilos, y no es thread-safe.
- `DbContext` singleton en ProcesadorReceptor: no hay concurrencia ni idempotencia (`seq` no se verifica).

### TareasMantenimiento

| Job | Frecuencia | Qué hace |
|---|---|---|
| `TimerForJobsWorker` | `TAREASPROGRAMADAS_FREQ_MINS` | Dispara las tareas TC00 (limpieza), TC01–03 (purgar históricos), TD01 (importar OPL desde el ERP), TR01–03 (reportes). **No tiene try/catch**: un fallo tumba el servicio. Usa la hora del contenedor (UTC), así que los jobs de hora fija se desfasan 5 h respecto a Lima. |
| `JobFechaProgramada` | 5 min | Snapshot de las OT del ERP en `ERPNisiraOTSnapshot`, con historial de fechas programadas. |
| `JobPackingList` | 10 min | Sincroniza `packinglist`. |
| `JobPuntoRutaMulti` | 30 s | Eventos inzone/outzone del proveedor GPS. |
| `JobVanguardStatus` | 30 s | Estado de movimiento de cada vehículo. |
| `SimpleConsumerWorker` | continuo | Cola **en memoria** con: NOTIFICACION, BORRAR_HISTORIAL, OBTENER_DATOS_EXTERNOS, REPORTE, LIMPIEZA_INTERNA. Se pierde al reiniciar. |

Si hay varias instancias, **todos los jobs se duplican**: no hay lock distribuido.

---

## 11. Frontend

### Pantallas reales

| Módulo | Ruta | Descripción |
|---|---|---|
| OT – lista | `/OTLista` (inicio) | Filtros por fechas, sucursal, cliente, OPL, transportista, almacenes, estado y texto. Columnas de hitos (alm. vacío, garita, packing, VTA, puerto). Cambio de estado masivo, reportes R01–R03 y exportación a Excel (solo la página visible). |
| OT – editar | `/otEditar/:id` | Asignación de flota, fechas de cita, ruta, contenedor y notas. Aplica las reglas de estado en el cliente y muestra la auditoría de cambios. "Crear OT" **no funciona**: solo hay GET y PUT. |
| Importar OT | `/ImportarOTLista` | Lista las OT del ERP y permite importar con checkbox. |
| OT en tiempo real | `/OTTRealLista` | Tabla que se refresca con cada evento de socket. Muestra avance, siguiente hito, ETA, tiempo restante y semáforo, y permite editar observaciones. |
| Mapa en tiempo real | `/OTTRealMapa` | Google Maps con un pin por vehículo coloreado según el ETA y banderas de hitos. **No dibuja recorrido ni polilínea.** Borra y vuelve a crear todos los marcadores en cada evento. |
| Geocercas | `/GeocercaLista` | Dibujo de polígono o círculo (DrawingManager), exportación GPX. |
| Rutas | `/RutasLista`, `/rutaEditar/:id` | Secuencia de geocercas con tipo, dirección, hito y control. |
| Transportistas, Choferes, Vehículos | `…Lista`, `…Editar/:id` | CRUD, resumen por OPL, exportación a Excel, documentos adjuntos y foto. |
| Usuarios / Roles | `/listaUsuarios`, `/listaRoles` | Usuarios con rol en una lista fija. Los roles son de solo lectura. |
| Tablas maestras | `/listaTablasMaestras` | CRUD de catálogos y de la configuración. |
| Proveedores GPS | `/listaProveedorGPS` | URLs y credenciales, **visibles en claro**. |
| Históricos | Login, Login Receptor, Requests | Auditoría. |

Hay un dashboard, pero está vacío y sin ruta.

### Problemas del frontend

- **Autenticación y permisos:**
  - El JWT y el perfil se guardan en `localStorage`.
  - No se controla la expiración del token.
  - El 403 se trata como logout.
  - Los roles se evalúan con IDs numéricos repartidos por las plantillas.
  - No hay guards por ruta, así que cualquier usuario autenticado puede abrir cualquier URL.
- **Código:**
  - 17 copias de `list-sortable.directive.ts`.
  - 591 `console.log`.
  - Restos de Velzon (auth demo, fake-backend, menús, i18n de 8 idiomas sin claves reales, unas 25 dependencias sin uso), con un bundle de unos 8 MB.
- **Contenido sin escapar:** HTML sin escapar en las ventanas de información del mapa (riesgo de XSS).
- **Lógica frágil:** la lógica de hitos busca textos en los nombres de las geocercas ("GARITA", "PACKING", "(VTA)").
- **🔴 Fecha fija `2026-12-31T23:59:00`** en `src/app/pages/ot/ot-editar/ot-editar.component.ts` (líneas 216, 241 y 354), usada como `FHEntregaContenedor` por defecto. **Desde enero de 2027 la validación recojo < cita < entrega fallará.**

---

## 12. Seguridad, roles y permisos

**Modelo actual:**
- RBAC fijo por ID de rol.
- Los datos se segmentan por `OperadorLogisticoId` del token, **solo en OT y reportes**.
- Los roles 220 y 230 (transportista) no filtran nada: el claim `transportistaid` siempre va vacío.

**Vulnerabilidades encontradas** (hay que evitarlas en la nueva versión y mitigarlas ya en la actual):

| # | Hallazgo | Severidad |
|---|---|---|
| 1 | Contraseñas en **MD5 sin sal** (web y Receptor) | Crítica |
| 2 | **Inyección SQL** en `ORDER BY` (`sortBy` y `sortDirection` interpolados) en unos 11 repositorios, por ejemplo `TMS.Shared/Persistence/AppUserRepository.cs:172` | Crítica |
| 3 | `/Cache/config` y `/Cache/tm?tableName=CONFIG` devuelven la contraseña SMTP y la credencial del ERP a **cualquier rol**. ProveedorGPS devuelve `AuthPayload` y `LastToken`. `HistoricRequest` guarda credenciales. | Crítica |
| 4 | **La misma clave JWT, issuer y audience** en Backend, I3, Receptor y ProcesadorReceptor: un token de un servicio sirve en todos | Alta |
| 5 | El rol 30 puede crear usuarios con rol 10 o 20 y resetear la contraseña de un SUPER-ADMIN (escalada de privilegios) | Alta |
| 6 | `DevTestController` y Swagger activos en producción. `/DevTest/configrapida` reescribe URLs de integración | Alta |
| 7 | TareasMantenimiento es anónimo (`POST /Tarea`), y el mock `/i3/order` también | Alta |
| 8 | Aislamiento entre OPL incompleto: caché, mapa, historial de posiciones, flota, archivos y geocercas no se filtran | Alta |
| 9 | Subida de archivos sin validar tipo ni tamaño, con riesgo de *path traversal* por `TableName` y `TableId` | Alta |
| 10 | API key estática y compartida, comparada sin tiempo constante y sin rate limit | Media |
| 11 | Contraseñas enviadas en claro por email; SMTP que acepta cualquier certificado; HTML de los correos sin codificar | Media |
| 12 | CORS `AllowAnyOrigin`, `/health` anónimo, `exception.Message` devuelto al cliente | Media |
| 13 | Secretos en `appsettings*.json`, en los seeds SQL y en `index.html` (API key de Google Maps) | Media |
| 14 | Sin rate limit en el login, sin bloqueo, sin 2FA, sin refresh token; JWT de 7 días sin revocación | Media |
| 15 | Los roles OPL y 210 pueden **borrar registros de auditoría** | Media |

---

## 13. Configuración y despliegue

**`AppSettings` en los servicios .NET:**
- Identidad del despliegue: `Id`, `Name`, `TenantId`, `ReceiverUrl`, `DefaultProveedorGPSId`, `WebAppUrl`.
- Base de datos y caché: `DB_Server`, `DB_Name`, `DB_Port`, `DB_User`, `DB_Pwd`, `Redis_ConStr`.
- Puertos: `SocketServer_Port`, `Receptor_Port`, `TareasMantenimiento_Port`.
- Otros: `TM_SimpleQueue_InitialConsumers`, `ServiceAppUserId`, `ServiceUserName`, `ServicePassword`, `IntegrationApiKey`, `RMQ_*`.

**Otras secciones y claves:**
- `JWT`: `SecretKey`, `Issuer`, `Audience`, `TokenDurationInMinutes` (10080 = 7 días).
- `Serilog`: File, Console y Seq.
- Simuladores: `AuthUser` y `AuthPassword` (I3), `Receptor_*` y los tiempos de trama (ProcesadorColector).

**Docker y despliegue:**
- Los Dockerfiles son multi-stage con `sdk:8.0` y `aspnet:8.0`, pero **esperan un contexto `./net/`** del monorepo original, así que no construyen tal cual.
- No tienen `EXPOSE`, no hay compose en el repo y no hay volúmenes para `uploads/` ni `logs/`.
- La URL entre servicios depende de `IsProduction()`: en producción usa el nombre DNS de compose y en cualquier otro entorno usa `localhost`.
- **Modelo de despliegue implícito:** un stack completo por cliente, con subdominio `*.tmscontrolvg.com`, un `appconfig.json` y una BD propios.

---

## 14. Bugs y deuda técnica detectados

1. **Fecha fija 2026-12-31** en el editor de OT → fallo a partir del 1 de enero de 2027.
2. `003-otposhistorial.sql` no está embebido, y su DDL no coincide con la entidad. **Los scripts SQL no reproducen la BD real** (ver §5).
3. Paginación incorrecta para usuarios OPL: las OT INACTIVA se filtran en memoria *después* de paginar.
4. `R01` y `R03`: el bucle `for (i = Count-1; i == 0; i--)` tiene la condición invertida y casi nunca se ejecuta.
5. Endpoints `TempOT/integracion/*`: NullReference por el usuario "INTEGRACION" → probablemente fallan siempre.
6. `PersonController`: el servicio no está registrado en DI.
7. `SocketService`: `async void` con `throw`, lo que puede tumbar el proceso.
8. `TimerForJobsWorker` sin try/catch; `SimpleTaskQueue` con el semáforo iniciado en 1 (el primer mensaje llega vacío).
9. `VanguardStatus` crece sin límite; `OTPosHistorial` no tiene purga; `PuntoRutaMulti` no tiene índice único.
10. Pérdida de mensajes en RabbitMQ; el mock ProcesadorColector usa la hora del servidor como `tgps`.
11. Tiempo real **sin caché efectiva**: cada GET lanza 4 consultas pesadas (UNION y `ST_Contains`), y cada mensaje GPS recalcula todas las OT.
12. PK basadas en unix ms, con colisiones; fechas que mezclan hora local y UTC con −5 h fijas en el código.
13. Las geocercas se envían solo al primer proveedor activo, no al proveedor del OPL.
14. Los archivos del Receptor nunca se purgan, porque la purga corre en otro contenedor.
15. `GPSUtils.DistanceBetweenPoints` puede devolver NaN (`Acos` sin acotar); el ETA calculado como distancia / velocidad es inestable a baja velocidad.
16. No hay tests automatizados, no hay CI y `Leer.txt` está desactualizado (menciona Postgres).

---

## 15. Lógica fijada en el código para Vanguard

Todo lo siguiente debe pasar a **configuración por tenant** o a **plugins**:

| Elemento | Qué está fijado |
|---|---|
| JWT | Issuer `vanguard/software` y audience `vanguard/customers` |
| Despliegue | Dominio `prodcompany.tmscontrolvg.com`, tenant `prodcompany` |
| Usuarios | Usuario de servicio `service_tmsmanager`; usuario del Receptor `navitel` |
| ERP | Nisira (DTOs `_Nisira`, GET con body, `idflete "002"`) |
| Regional | Cultura `es-PE`, −5 h fijas, ubigeos y distritos de Ica, catálogos MTC y licencias de Perú |
| Estadía en packing | OPL "MAERSK" o "TPP" y ruta "PDP" → 6 h (8 h en los demás casos) |
| Hitos | Posiciones de ruta 1 a 5 con significado fijo; códigos TC001–TC003 y H002; detección por nombre de geocerca ("GARITA", "PACKING", "VTA") |
| Nombres | `ActualizarRuta_Vanguard`, tabla `VanguardStatus` |
| Packing | SENASA y packing list (agroexportación) |
| Proveedor GPS | Un único `DefaultProveedorGPSId` y un único `Receptor_TenantId` (con un TODO en el código que lo reconoce) |

---

## 16. Qué debes tener en cuenta para el SaaS

### 16.1 Multi-tenancy (lo más importante)

- **Decide el modelo de aislamiento.**
  - Recomendado: **una BD compartida con `tenant_id` en cada tabla y Row-Level Security (PostgreSQL RLS)**.
  - Para clientes enterprise que lo exijan, ofrece "BD dedicada" como plan premium, con el mismo esquema.
- **Jerarquía de tenancy:** `Tenant` (empresa cliente del SaaS, por ejemplo un exportador) → `Organización/OPL` → `Usuario`. Hoy el OPL hace de "sub-tenant"; mantén ese concepto, pero **aplicado en todas las consultas**.
- **Resolución del tenant:**
  - por subdominio (`acme.tutms.com`) o, para marca blanca, por dominio personalizado;
  - el tenant va en el claim del token;
  - **nunca se confía** en un `tenantid` que venga en el body.
- **Recursos que hay que prefijar por tenant:**
  - caché (claves Redis `t:{tenant}:…`);
  - colas y eventos (`tenant_id` en el mensaje y en la clave de routing);
  - almacenamiento de archivos (`s3://bucket/{tenant}/…`);
  - salas de WebSocket (`room: tenant:{id}:opl:{id}`);
  - logs y métricas (`tenant_id` como atributo);
  - jobs programados (por tenant y con su zona horaria).
- **Maestros compartidos o propios:** hoy vehículos, choferes y transportistas se comparten entre OPL con `OLids`. En el SaaS decide si un transportista es global a la plataforma (un marketplace) o propio de cada tenant. Recomendación: propio del tenant, con una tabla puente `opl_asset_access`.

### 16.2 Identidad y acceso
- Usar un **IdP gestionado** (Auth0, Clerk, WorkOS, Keycloak o Zitadel) con OIDC, MFA, SSO (SAML / Azure AD para clientes corporativos), invitaciones y recuperación de contraseña.
- Contraseñas con **Argon2id** o bcrypt si se gestionan localmente. Migrar desde MD5 forzando un reseteo o re-hasheando al siguiente login.
- **RBAC y permisos granulares:** `permission` (por ejemplo `ot.assign`, `ot.view`, `fleet.edit`, `reports.run`), roles editables por el administrador del tenant y roles de sistema (el soporte de la plataforma) separados de los roles del tenant.
- **Tokens:** vida corta (15 min) con refresh token rotativo y cookie httpOnly en la web, o BFF. Claves asimétricas (RS256 / EdDSA) con JWKS.
- **Integraciones máquina a máquina:** API keys por tenant y por integración (con hash, alcances, rotación y rate limit) o OAuth client credentials. Webhooks entrantes firmados con **HMAC**.

### 16.3 Integraciones como plugins (conectores)
- **Interfaz `GpsProviderAdapter`:** `authenticate`, `upsertGeofences`, `upsertOrder`, `startTracking`, `stopTracking`, `ingest(payload)` → evento normalizado. Adaptadores: Navitel (el contrato actual), Comsatel, Protegecorp, Wialon, Samsara, Geotab, Traccar, y **app móvil propia**.
- **Interfaz `ErpAdapter`:** `listOrders`, `listPartners`, `listBranches`. Adaptadores: Nisira, SAP B1, Odoo, CSV/Excel, API REST genérica.
- **Configuración por tenant y por OPL:** credenciales cifradas en un **vault** (AWS Secrets Manager, GCP Secret Manager, HashiCorp Vault o, como mínimo, cifrado a nivel de columna con KMS). **Nunca devolverlas por API** (solo "configurado / no configurado").
- **Motor de cálculo propio.** Hoy el proveedor GPS calcula la geocerca y el ETA. En el SaaS conviene **calcular dentro de la plataforma** a partir de la posición cruda, con PostGIS: punto en polígono, ETA con un motor de rutas (OSRM, Valhalla o Google Routes) y detección de entrada y salida. Así no dependes de que cada proveedor implemente la "data consolidada", y puedes aceptar cualquier fuente de posiciones (GPS, app, AVL).
- **Idempotencia:** clave `(tenant, provider, device, t_gps)` o `seq`. Outbox en la ingesta.
- **Observabilidad de integraciones:** conserva el concepto de `HistoricRequest` (es valioso), pero **sin credenciales**, con retención y con el cuerpo en object storage.

### 16.4 Datos y escalabilidad
- **Posiciones GPS en una base de series temporales:** TimescaleDB (hypertables, compresión, retención y *continuous aggregates*) o ClickHouse si el volumen es muy alto.
  - Ejemplo de volumen: 500 vehículos con una posición cada 15 s dan unos 2,9 M filas al día por tenant.
- **Estado caliente** (la última posición y el ETA de cada OT activa) en Redis o en una tabla `order_tracking_state`, actualizado **de forma incremental**. Hoy se recalculan todas las OT en cada mensaje.
- **Tiempo real:** enviar **deltas** por WebSocket (o SSE) a la sala del tenant y del OPL, no el aviso de "vuelve a pedir todo".
- **Mensajería fiable:** colas durables (RabbitMQ quorum, NATS JetStream, SQS o Kafka / Redpanda si el volumen es alto), DLQ, reintentos con backoff, publisher confirms y consumidores idempotentes.
- **Jobs:** scheduler distribuido con lock (BullMQ, Temporal, Hangfire o Quartz en cluster), con zona horaria por tenant.
- **Archivos:** object storage (S3 / R2 / GCS) con URLs firmadas, validación de tipo y tamaño, y antivirus opcional.
- **Migraciones versionadas** (Prisma, Drizzle, Flyway o Alembic) en CI, **no al arrancar la app**.
- **Foreign keys reales**, `timestamptz` en UTC y zona horaria por tenant solo para mostrar. Sin ajustes de −5 h en el código.

### 16.5 Producto SaaS (lo que no existe hoy)
- Onboarding self-service: registro de la empresa, verificación, asistente inicial (sucursales, OPL, geocercas base, proveedor GPS) e importación masiva desde Excel o CSV.
- **Billing:** planes por vehículos activos, OT al mes o usuarios, con límites y cuotas, Stripe (o Culqi / Mercado Pago para LatAm), facturación electrónica (SUNAT si facturas en Perú), periodo de prueba, upgrade y downgrade.
- **Consola de super-admin de la plataforma:** tenants, planes, uso, impersonación auditada, feature flags y estado de las integraciones.
- **Branding por tenant:** logo, colores, dominio y remitente de email.
- **Notificaciones configurables:** email, WhatsApp Business API, SMS y push, con plantillas por tenant y reglas de alerta (desvío, detención prolongada, ETA en rojo, sin transmisión, documento vencido).
- **Cumplimiento:** Ley 29733 de protección de datos personales (Perú) y GDPR si hay clientes en la UE; DPA con los clientes; retención configurable; exportación y borrado de datos por tenant.
- **Registro de auditoría inmutable** (append-only), que el tenant no pueda borrar.
- **SLA y operación:** backups con PITR, DR, página de estado, monitoreo (OpenTelemetry, Grafana, Sentry) y alertas.

### 16.6 Generalizar el dominio
Hoy el producto sirve solo para contenedores de agroexportación. Para crecer:
- Hitos genéricos y configurables: una **plantilla de flujo** por tipo de operación (contenedor de exportación, distribución urbana, carga general, última milla).
- Campos personalizados por tenant (JSONB con esquema) para lo específico: SENASA, booking, naviera, precinto.
- **Reglas de SLA** configurables en lugar de "MAERSK/TPP/PDP = 6 h": un motor de reglas simple (condición → umbral).
- Rutas reales: polilínea planificada, corredor de tolerancia para detectar desvíos, paradas y ventanas horarias.
- Internacionalización (es y en como mínimo), varias monedas, catálogos vehiculares por país y zona horaria por tenant.

---

## 17. Stack propuesto para la nueva versión

Hay dos opciones razonables. Elige según tu equipo.

### Opción A — TypeScript de punta a punta (recomendada si el equipo es web/JS)

| Capa | Tecnología |
|---|---|
| Frontend web | **Next.js 15 (React) + TypeScript**, Tailwind y shadcn/ui, TanStack Query y TanStack Table |
| Mapas | **MapLibre GL** con tiles (MapTiler o Stadia) → evita el coste de Google por tenant. Dibujo de geocercas con Mapbox GL Draw o Terra Draw. Google Maps como opción del plan |
| Backend | **NestJS** (monolito modular) o Fastify; Zod o class-validator; OpenAPI |
| ORM y migraciones | Prisma o Drizzle |
| BD | **PostgreSQL 16 + PostGIS + TimescaleDB**, con RLS por `tenant_id` |
| Caché, pub/sub y colas | Redis (Valkey) + **BullMQ**; NATS JetStream o RabbitMQ (quorum) para la ingesta GPS |
| Tiempo real | Socket.IO con adaptador Redis, o WebSocket/SSE con salas por tenant |
| Auth | Clerk, Auth0 o WorkOS (SSO B2B); o Keycloak / Zitadel si quieres self-host |
| App móvil del chofer | **React Native (Expo)**: GPS en segundo plano, POD con foto y firma, trabajo offline |
| Archivos | S3 / Cloudflare R2 |
| Reportes | ExcelJS y CSV en workers, con dashboards embebidos (Metabase o Superset) o propios |
| Observabilidad | OpenTelemetry, Grafana (Loki, Tempo, Prometheus) o Datadog; Sentry |
| Infraestructura | Docker; AWS ECS Fargate, Fly.io o Kubernetes (EKS / GKE) cuando crezca; Terraform; GitHub Actions |

### Opción B — Mantener .NET, pero bien hecho (si tu equipo es C#)

- **ASP.NET Core 9** como monolito modular, con EF Core + Npgsql y NetTopologySuite (PostGIS).
- **Finbuckle.MultiTenant** o filtros globales de EF por tenant.
- Duende IdentityServer o Keycloak para la identidad.
- MassTransit sobre RabbitMQ o Azure Service Bus (outbox, reintentos, DLQ).
- Hangfire o Quartz en cluster para los jobs; SignalR con backplane Redis para el tiempo real.
- Frontend en React/Next.js (sugerido) o Angular 19 con standalone components.

### Opción C — Python (si el equipo es Python o hay mucha analítica e IA)

- FastAPI con SQLAlchemy 2 async, Alembic, GeoAlchemy2 y PostGIS/Timescale.
- Celery o Arq para tareas; frontend Next.js.
- Ventaja: ETA predictivo con ML y analítica en el mismo lenguaje.

**Principios comunes, sea cual sea el stack:**
1. **Monolito modular primero.** Separar en microservicios solo lo que escala distinto: la *ingesta GPS* y los *workers*. Hoy hay seis servicios con estado compartido y ninguno de los beneficios de los microservicios.
2. **Arquitectura por módulos de dominio:** `identity`, `tenancy`, `masters` (flota y partners), `geo` (geocercas, rutas), `orders` (OT, hitos), `tracking` (ingesta, estado, historial), `integrations` (conectores), `notifications`, `reporting`, `billing`.
3. **Contratos de eventos internos:** `order.assigned`, `position.received`, `geofence.entered`, `milestone.reached`, `eta.updated`, `order.completed`. Las notificaciones, los reportes, los webhooks salientes y la caché consumen esos eventos.
4. **API pública versionada** (`/v1`) con webhooks salientes para los clientes: eso es lo que hace que un TMS SaaS se integre y crezca.

---

## 18. Modelo de datos objetivo (SaaS)

Es un borrador en PostgreSQL. Todas las tablas llevan `tenant_id uuid not null` (excepto las de plataforma), RLS, `created_at` y `updated_at` como `timestamptz`, `created_by`, `updated_by` y `deleted_at` (borrado lógico).

```
-- Plataforma
tenants(id, slug, name, plan_id, status, timezone, locale, country, branding jsonb, settings jsonb)
plans(id, name, limits jsonb, price) ; subscriptions(tenant_id, plan_id, status, period_end, provider_ref)
usage_counters(tenant_id, metric, period, value)            -- vehículos activos, OT/mes, posiciones
feature_flags(tenant_id, key, enabled)

-- Identidad
users(id, email, name, status, idp_subject)                 -- global
memberships(tenant_id, user_id, organization_id NULL, role_id)
roles(id, tenant_id NULL, name, is_system) ; role_permissions(role_id, permission)
api_keys(id, tenant_id, name, hash, scopes, last_used_at, expires_at)

-- Organización (antes OPL / Cliente / Sucursal)
organizations(id, tenant_id, type[OPL|CUSTOMER|CARRIER|BRANCH], name, tax_id, parent_id, gps_integration_id)
contacts(id, tenant_id, organization_id, name, email, phone)

-- Flota
carriers(id, tenant_id, organization_id, legal_name, tax_id, status)          -- Transportista
vehicles(id, tenant_id, carrier_id, plate, kind[TRACTOR|TRAILER|TRUCK|VAN], device_id, specs jsonb, status, tech_review_due)
devices(id, tenant_id, integration_id, external_id /*IMEI*/, vehicle_id)
drivers(id, tenant_id, carrier_id, first_name, last_name, doc_type, doc_number, license_class, license_number, license_due, phone, status)
asset_access(tenant_id, organization_id, asset_type, asset_id)                -- reemplaza OLids
documents(id, tenant_id, entity_type, entity_id, doc_type, file_key, expires_at)  -- con alertas de vencimiento

-- Geo
geofences(id, tenant_id, name, short_name, category_id, geom geography, radius_m NULL, address, synced_at jsonb)
route_templates(id, tenant_id, name, is_default, planned_path geography NULL, tolerance_m)
route_template_stops(route_template_id, seq, geofence_id NULL, dynamic_slot, stop_type, direction, milestone_type_id, compute)

-- Órdenes
order_types(id, tenant_id, name, milestone_flow jsonb, custom_fields_schema jsonb)
orders(id, tenant_id, order_type_id, external_ref, source[ERP|MANUAL|API|CSV], status, customer_org_id, opl_org_id,
       branch_org_id, booking, container_no, custom_fields jsonb, route_template_id, reschedule_count, ...)
order_assignments(order_id, carrier_id, tractor_id, trailer_id, driver_id, assigned_at)
order_stops(order_id, seq, geofence_id, milestone_type_id, planned_at, eta_at, arrived_at, departed_at, status_color, sla_met)
order_status_history(order_id, from, to, at, by, reason)

-- Tracking
positions (hypertable)(tenant_id, device_id, order_id NULL, t_gps, t_recv, lat, lng, geom, speed, heading, alt, ignition, raw jsonb)
geofence_events(tenant_id, order_id, device_id, geofence_id, type[ENTER|EXIT], at)
order_tracking_state(order_id, last_position, current_stop_seq, next_stop_id, dist_km, eta_at, progress, updated_at)
alerts(id, tenant_id, order_id, rule_id, type, severity, opened_at, closed_at, ack_by)
alert_rules(id, tenant_id, type, conditions jsonb, channels jsonb)

-- Integraciones
integrations(id, tenant_id, kind[GPS|ERP|WMS|PACKING], provider, config jsonb, secret_ref, status)
integration_logs(id, tenant_id, integration_id, direction, endpoint, status, duration_ms, records, body_key, error)
inbound_dedup(tenant_id, integration_id, idempotency_key, received_at)

-- Transversal
audit_log (append-only)(tenant_id, actor, action, entity_type, entity_id, diff jsonb, at, ip)
notifications(tenant_id, channel, template, to, status, sent_at)
catalogs(tenant_id NULL, catalog, code, label, parent_code, sort, enabled)    -- NULL = catálogo global de la plataforma
settings(tenant_id, key, value jsonb)                                          -- reemplaza CONFIG, sin secretos
```

**Equivalencias con el sistema actual:**

| Actual | Nuevo |
|---|---|
| `OT` | `orders` + `order_assignments` |
| `OTHito` | `order_stops` |
| `OTCalcs1` | `order_tracking_state` |
| `OTPosHistorial` | `positions` |
| `PuntoRutaMulti` | `geofence_events` |
| `Ruta` / `RutaPos` | `route_templates` / `route_template_stops` |
| `TablasMaestras` | `catalogs` + `settings` + `integrations` |
| `ProveedorGPS` | `integrations(kind=GPS)` |
| `OPL` / `Cliente` / `Sucursal` | `organizations` |
| `HistoricRequest` | `integration_logs` |
| `HistoricChange` / `HitoricDeleteVTC` | `audit_log` |
| `ERPNisiraOTSnapshot` | tabla de staging de importación |
| `packinglist` | `custom_fields` / tabla del módulo "packing" como add-on |

---

## 19. Funcionalidad faltante

Backlog de producto, priorizado:

**Imprescindible para lanzar el SaaS**
- [ ] Multi-tenant con RLS, onboarding y administración de usuarios con invitación, recuperación de contraseña, MFA y roles editables.
- [ ] Creación manual de OT (hoy no funciona) e importación por **Excel/CSV** además del ERP.
- [ ] Conector GPS genérico, con Navitel como primer adaptador; motor propio de geocercas y ETA.
- [ ] Mapa en tiempo real con **actualización incremental**, recorrido histórico (replay) y polilínea de la ruta.
- [ ] Alertas configurables: sin transmisión, detención, desvío, ETA en rojo, exceso de velocidad y vencimiento de brevete o revisión técnica (SOAT, etc.).
- [ ] Dashboard real con KPI: cumplimiento de citas por OPL y transportista, tiempos por hito, estadía en planta y utilización de flota.
- [ ] Billing y planes; consola de super-admin.
- [ ] Auditoría inmutable y registro de integraciones sin secretos.

**Diferenciadores (siguiente fase)**
- [ ] **App móvil del chofer:** hoja de ruta, check-in por hito, POD (foto y firma), incidencias, chat, y GPS del teléfono como fuente alternativa.
- [ ] **Portal de cliente / consignatario:** enlace público de tracking por booking o contenedor, ETA y documentos.
- [ ] **Portal de transportista:** aceptar o rechazar OT, subir documentos, ver su flota y sus OT.
- [ ] Notificaciones por WhatsApp Business o SMS para el chofer y el cliente.
- [ ] Planificación: asignación sugerida de unidades, optimización de rutas (VRP) y ventanas horarias.
- [ ] Gestión documental con vencimientos (brevete, SOAT, revisión técnica, póliza).
- [ ] Costos y liquidación: tarifas por ruta, fletes, pagos a transportistas y facturación al cliente.
- [ ] Demurrage/detention y control de contenedores (retiro y devolución de vacío).
- [ ] API pública y webhooks; marketplace de conectores (ERP, WMS, navieras, portales portuarios).
- [ ] ETA predictivo con ML a partir del historial de posiciones.
- [ ] Informes programables y exportables desde la UI, no solo por email.

---

## 20. Hoja de ruta sugerida

| Fase | Duración estimada | Entregables |
|---|---|---|
| **0. Estabilizar lo actual** | 1–2 semanas | Corregir la fecha 2026-12-31, rotar secretos, cerrar DevTest/Swagger/`/Cache/config`, lista blanca en `ORDER BY`, colas durables. Hacer `mysqldump --no-data` de producción. |
| **1. Descubrimiento y diseño** | 2–3 semanas | ADRs (stack, modelo de tenancy, IdP, ingesta GPS), modelo de datos final, contratos de API y eventos, diseño UX (Figma), definición de planes. |
| **2. Plataforma base** | 4–6 semanas | Tenancy con RLS, IdP, RBAC, auditoría, settings, catálogos, CI/CD, entornos, observabilidad, billing básico. |
| **3. Núcleo TMS (paridad)** | 6–8 semanas | Organizaciones, flota, geocercas y rutas (MapLibre), órdenes con estados e hitos, importación CSV y conector Nisira, reportes R01/R02/R03 equivalentes. |
| **4. Tracking** | 4–6 semanas | Ingesta GPS (adaptador Navitel con el mismo contrato `/Consolidated` para que el proveedor no cambie nada), motor de geocercas y ETA, TimescaleDB, tiempo real incremental, alertas. |
| **5. Migración del primer cliente** | 2–3 semanas | ETL desde MySQL con validación, operación en paralelo (*shadow*) junto al sistema viejo y corte. |
| **6. Crecimiento** | continuo | App del chofer, portal del cliente, WhatsApp, API pública y webhooks, más conectores, analítica. |

---

## 21. Checklist de migración

**Datos (ETL MySQL → PostgreSQL):**
- [ ] Obtener el **esquema real de producción** (los scripts del repo no coinciden con él).
- [ ] Mapear los IDs GUID tal cual (conservar los UUID) y añadir `tenant_id` al tenant del cliente migrado.
- [ ] Convertir `cat` y `uat` (unix ms) a `timestamptz`; revisar las fechas guardadas como `VARCHAR` en la OT (`Fecha`, `FechaCita`).
- [ ] Convertir `Geofence.JPoints` y `Rad` a `geography` (polígono o punto + radio).
- [ ] `RutaPos` → `route_template_stops`; `OT.RPointsJson` y `OTHito` → `order_stops`.
- [ ] `OLids` (JSON) → `asset_access`.
- [ ] `OTPosHistorial` → `positions` (hypertable), considerando el volumen y la retención de 90 días.
- [ ] `TablasMaestras` → `catalogs` (VEH*, TIPO*, GEOCERCACAT, NOMBREHITO…) y `settings` (CONFIG sin secretos). Las credenciales van a `integrations` y al vault.
- [ ] Usuarios: migrar sin contraseña y forzar un reseteo (el MD5 no se reutiliza).
- [ ] Archivos de `uploads/common` → S3, y `FileItem` → `documents`.

**Funcionalidades que hay que alcanzar (paridad):**
- [ ] Importación de OT desde el ERP con comparación con la BD e importación selectiva.
- [ ] Máquina de estados de la OT con las mismas validaciones (OPL con proveedor GPS, tracto con IMEI, geocercas distintas, recojo < cita < entrega).
- [ ] Rutas con puntos dinámicos (almacén de retiro y de retorno) y tipos de control de hito.
- [ ] Semáforo de ETA según `OT_TREAL_SEMAFORO_MINUTOS`; OT finalizadas visibles N horas.
- [ ] KPI actuales: programa de embarques, actividades en packing, cumplimiento de citas, retrasos, avance de pedidos, encendido de frío, retransmisión GPS, unidades cerca del packing.
- [ ] Reportes R01, R02 y R03.
- [ ] Notificaciones NOTIF_* por rol.
- [ ] Deduplicación de flota (placa + IMEI, DNI + brevete, RUC) y borrador de alta.
- [ ] Endpoints de integración para terceros (`integracion/getall`, `integracion/tracking`) → API pública v1.
- [ ] Contrato del Receptor compatible (`/auth/login` + `/Consolidated`) para no obligar a Navitel a cambiar.
- [ ] Jobs: snapshot de fechas programadas del ERP, packing list, eventos PuntoRutaMulti y estado del vehículo.

---

## 22. Acciones urgentes en el sistema actual

Mientras se construye la nueva versión, el sistema actual sigue en producción:

1. 🔴 **Antes del 31 de diciembre de 2026:** reemplazar la fecha fija `2026-12-31T23:59:00` en `vanguard-tms-frontend-main/src/app/pages/ot/ot-editar/ot-editar.component.ts` (líneas 216, 241 y 354) por `null` o por una fecha calculada.
2. 🔴 **Rotar todos los secretos** que están en el repo y en los seeds: clave JWT, ServicePassword, IntegrationApiKey, contraseña SMTP, token del ERP, credenciales de RabbitMQ, credenciales de I3 y del Receptor, y la API key de Google Maps (restringirla por dominio).
3. 🔴 Bloquear `/Cache/config`, `/Cache/tm?tableName=CONFIG` y `/DevTest/*`, y desactivar Swagger en producción.
4. 🔴 Aplicar una lista blanca de columnas a `sortBy` y `sortDirection` en todos los repositorios.
5. 🟠 Poner autenticación en TareasMantenimiento y usar claves JWT distintas por servicio.
6. 🟠 Hacer las colas RabbitMQ durables, con mensajes persistentes y una DLQ.
7. 🟠 Implementar la purga de `OTPosHistorial` y corregir el upsert de `VanguardStatus`.
8. 🟡 Añadir try/catch en `TimerForJobsWorker` y definir `TZ=America/Lima` en los contenedores.
