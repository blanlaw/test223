# Estimación: TMS SaaS general e integrable, desarrollado desde cero

> **Complementa a:** `ANALISIS-TMS-SAAS.md` (análisis del TMS actual de Vanguard).
> **Fecha:** 2026-09-23
> **Pregunta que responde:** si el TMS se construye **desde cero** como SaaS **genérico** (no a medida), capaz de integrarse con muchas empresas de transporte en **ambos sentidos** (ellas se conectan a nuestra API y nosotros a las suyas), ¿cuánto tarda, qué recursos necesita y cuánto cuesta la mano de obra de un desarrollador senior?
>
> ⚠️ **Todas las cifras son estimaciones de referencia** a septiembre de 2026, basadas en el alcance descrito aquí. Las tarifas varían según el país, la modalidad de contratación y el perfil. Verifícalas en tu mercado antes de cotizar.

---

## Índice

1. [Resumen rápido](#1-resumen-rápido)
2. [Qué se construye: alcance del producto](#2-qué-se-construye-alcance-del-producto)
3. [La integración en ambos sentidos](#3-la-integración-en-ambos-sentidos)
4. [Arquitectura y tecnologías](#4-arquitectura-y-tecnologías)
5. [Estimación de esfuerzo por módulo](#5-estimación-de-esfuerzo-por-módulo)
6. [Escenarios de alcance y tiempos](#6-escenarios-de-alcance-y-tiempos)
7. [Recursos humanos: equipo sugerido](#7-recursos-humanos-equipo-sugerido)
8. [Costo de mano de obra](#8-costo-de-mano-de-obra)
9. [Costos de infraestructura y servicios](#9-costos-de-infraestructura-y-servicios)
10. [Otros costos únicos](#10-otros-costos-únicos)
11. [Costo total estimado](#11-costo-total-estimado)
12. [Cronograma por fases](#12-cronograma-por-fases)
13. [Riesgos que mueven la estimación](#13-riesgos-que-mueven-la-estimación)
14. [Cómo reducir tiempo y costo](#14-cómo-reducir-tiempo-y-costo)
15. [Si vas a cotizarlo a un cliente](#15-si-vas-a-cotizarlo-a-un-cliente)

---

## 1. Resumen rápido

| Escenario | Qué incluye | Esfuerzo (con contingencia) | Tiempo con 1 senior solo | Tiempo con equipo | Mano de obra (referencia LatAm) |
|---|---|---|---|---|---|
| **MVP** | Validar con 1–3 transportistas | ~10–11 meses-persona | 11–13 meses | **4–5 meses** (2–3 personas) | **USD 45k – 95k** |
| **V1 comercial** ⭐ | SaaS vendible con API pública y webhooks | ~22 meses-persona | 20–24 meses (no recomendado) | **6–8 meses** (4–5 personas) | **USD 100k – 200k** |
| **Plataforma completa** | App del chofer, portales, catálogo de conectores, optimización | ~42 meses-persona | no viable | **10–13 meses** (6–7 personas) | **USD 190k – 380k** |

> Si se contrata con tarifas de EE. UU. o Europa, multiplica la mano de obra por **2 a 3**.
> **Recomendación:** apuntar a la **V1 comercial** con un equipo pequeño (1 tech lead senior + 2 desarrolladores + apoyo parcial de UX, QA y DevOps) y **empezar a vender en el mes 4–5 con el MVP**.

---

## 2. Qué se construye: alcance del producto

Es un TMS **multi-tenant** donde cada empresa cliente (transportista, operador logístico, generador de carga) tiene su espacio aislado. El producto se configura en lugar de programarse para cada cliente:

- **Tipos de operación configurables:** cada tipo es una plantilla de hitos, por ejemplo contenedor de exportación, distribución, carga general o última milla. Reemplaza la lógica actual, fijada para agroexportación.
- **Campos personalizados por tenant**, para datos como booking, precinto o SENASA.
- **Reglas de SLA y alertas configurables.** Hoy están en el código, por ejemplo "MAERSK = 6 h".
- **Conectores intercambiables** para GPS, ERP y sistemas de otras empresas.
- **API pública y webhooks** como parte central del producto, no como añadido.

---

## 3. La integración en ambos sentidos

Es la pieza que convierte el TMS en **plataforma**. Tiene que diseñarse desde el día 1.

### 3.1 Entrante: otras empresas se conectan a nuestra API

| Capacidad | Detalle |
|---|---|
| **API REST pública v1** | Especificación OpenAPI 3.1 con versionado (`/v1`). Recursos: órdenes, paradas e hitos, vehículos, conductores, posiciones, eventos, documentos (POD), geocercas. |
| **Autenticación B2B** | OAuth 2.0 *client credentials* por integración, o API keys con alcances (`orders:read`, `positions:write`…), rotación y expiración. |
| **Ingesta de posiciones y eventos** | Endpoint masivo (`POST /v1/positions:batch`), con idempotencia (`Idempotency-Key` o `device + t_gps`) y rate limit por cliente. |
| **Webhooks entrantes de proveedores** | Firma HMAC, verificación y reintentos. |
| **Sandbox** | Tenant de pruebas con datos ficticios y credenciales separadas. |
| **Portal de desarrolladores** | Documentación interactiva, guías, ejemplos (cURL, JS, Python, C#), colección Postman y registro de llamadas por integración. |

### 3.2 Saliente: nosotros nos conectamos a otras empresas

| Capacidad | Detalle |
|---|---|
| **Webhooks salientes** | Los clientes se suscriben a eventos (`order.created`, `order.assigned`, `milestone.reached`, `eta.updated`, `order.completed`, `alert.opened`). Se firman con HMAC y tienen reintentos con backoff, cola de mensajes fallidos (DLQ), reenvío manual y panel de entregas. |
| **Framework de conectores** | Interfaz común: `authenticate`, `pullOrders`, `pushOrder`, `upsertGeofences`, `startTracking`, `stopTracking`, `ingest`. Cada integración es un **adaptador**. |
| **Conectores iniciales** | GPS: Navitel (el contrato actual) y un GPS genérico (el formato propio de la API). ERP: Nisira y un conector genérico por CSV, Excel o SFTP. |
| **Conectores posteriores** | Wialon, Traccar, Samsara o Geotab; ERP: SAP B1, Odoo; navieras y portales portuarios; EDI (204/214) si se apunta a EE. UU. |
| **Mapeo de datos** | Plantillas de transformación por tenant para campos, códigos y estados, sin escribir código nuevo. |
| **Credenciales** | Guardadas en un vault y **nunca devueltas por la API**. |
| **Observabilidad de integraciones** | Registro de cada llamada (sin secretos), métricas de éxito y latencia, alertas cuando un conector falla. |

**Por qué pesa en la estimación:** la integración bidireccional equivale a construir **un producto dentro del producto**. Representa aproximadamente el **25–30 % del esfuerzo total**.

---

## 4. Arquitectura y tecnologías

Es la propuesta del documento de análisis, resumida.

| Capa | Tecnología sugerida |
|---|---|
| Frontend web | Next.js + React + TypeScript, Tailwind, TanStack Query/Table |
| Mapas | MapLibre GL + MapTiler/Stadia (evita el costo de Google Maps por cada tenant) |
| Backend | NestJS (TypeScript) como **monolito modular**; alternativa: .NET 9 si el equipo es de C# |
| Base de datos | PostgreSQL 16 + **PostGIS** (geocercas) + **TimescaleDB** (posiciones GPS), con Row-Level Security por tenant |
| Colas y jobs | Redis + BullMQ para jobs; NATS JetStream o RabbitMQ (quorum) para la ingesta GPS |
| Tiempo real | WebSocket (Socket.IO con Redis) con salas por tenant |
| Identidad | Servicio gestionado (Clerk, Auth0 o WorkOS), o Keycloak/Zitadel si se aloja en servidores propios |
| Archivos | S3 / Cloudflare R2 |
| Pagos | Stripe (o Culqi / Mercado Pago en Perú) |
| Notificaciones | Email (Resend/SES), WhatsApp Business API |
| App del chofer | React Native (Expo) |
| Observabilidad | OpenTelemetry + Grafana o Datadog, Sentry |
| Infraestructura | Docker, AWS (ECS Fargate + RDS) o similar, Terraform, GitHub Actions |

---

## 5. Estimación de esfuerzo por módulo

**Unidad:** semanas de **un desarrollador senior a tiempo completo**. Incluye desarrollo, pruebas automatizadas del módulo y revisión de código.

| # | Módulo | MVP | V1 | Completa | Contenido |
|---|---|---:|---:|---:|---|
| 1 | Descubrimiento, arquitectura, ADRs, modelo de datos | 2 | 3 | 3 | Decisiones técnicas, contratos de API y eventos |
| 2 | Plataforma base multi-tenant | 4 | 6 | 7 | Tenants, RLS, identidad, usuarios e invitaciones, RBAC con permisos, auditoría, configuración, catálogos |
| 3 | DevOps: CI/CD, IaC, entornos, observabilidad | 2 | 3 | 4 | dev/staging/prod, backups, alertas |
| 4 | Maestros | 3 | 5 | 6 | Organizaciones (clientes, OPL, transportistas, sucursales), vehículos, dispositivos, conductores, documentos con vencimiento, importación CSV/Excel |
| 5 | Geo: geocercas y plantillas de ruta | 3 | 4 | 5 | Dibujo en mapa, PostGIS, rutas con paradas y puntos dinámicos, corredor para detectar desvíos |
| 6 | Órdenes de transporte | 4 | 6 | 7 | Tipos de operación, estados, asignación, paradas e hitos, reprogramación, historial, creación manual e importación |
| 7 | **Plataforma de integración** | 4 | 8 | 10 | API pública v1, OAuth/API keys, webhooks entrantes y salientes, idempotencia, sandbox, portal de desarrolladores |
| 8 | Framework de conectores y conectores | 2 | 5 | 10 | Navitel, GPS genérico, Nisira y CSV/SFTP en V1; más GPS y ERP en la versión completa (1–2 semanas por conector) |
| 9 | Tracking | 5 | 8 | 9 | Ingesta de posiciones, cola fiable, series temporales, motor de geocercas (entrada/salida), ETA, estado en tiempo real, retención |
| 10 | Tiempo real y mapa en vivo | 2 | 4 | 5 | Actualización incremental, recorrido histórico, polilínea de la ruta |
| 11 | Alertas y notificaciones | 1 | 4 | 5 | Reglas configurables, email, WhatsApp, centro de notificaciones |
| 12 | Dashboards y reportes | 2 | 4 | 6 | KPI de cumplimiento, tiempos por hito, exportación Excel/CSV, reportes programados |
| 13 | Billing, planes y consola de super-admin | 1 | 5 | 6 | Planes, límites y uso, pagos, facturas, impersonación auditada, feature flags |
| 14 | Onboarding self-service y branding | 0 | 2 | 3 | Registro, asistente inicial, logo y colores, dominio propio |
| 15 | QA, seguridad y rendimiento | 2 | 5 | 8 | Pruebas E2E, pruebas de carga de la ingesta, hardening, corrección de hallazgos del pentest |
| 16 | Documentación y lanzamiento | 1 | 2 | 3 | Documentación de usuario y API, runbooks |
| 17 | Portal del cliente / consignatario | — | — | 3 | Enlace público de tracking, ETA, documentos |
| 18 | Portal del transportista | — | — | 4 | Aceptar o rechazar órdenes, subir documentos, ver flota |
| 19 | **App móvil del chofer** | — | — | 10 | Hoja de ruta, check-in por hito, POD (foto y firma), modo offline, GPS del teléfono, publicación en tiendas |
| 20 | Planificación y optimización | — | — | 6 | Sugerencia de asignación, rutas con ventanas horarias |
| 21 | Costos y liquidación | — | — | 6 | Tarifas, fletes, pagos a transportistas |
| | **Subtotal (semanas-senior)** | **38** | **74** | **126** | |
| | Contingencia | +20 % | +25 % | +30 % | Imprevistos, cambios de alcance, integraciones difíciles |
| | **Total (semanas-senior)** | **~46** | **~92** | **~164** | |
| | **Total (meses-persona, 4,33 semanas/mes)** | **~10,5** | **~21–22** | **~38–42** | |

> El diseño UX/UI, la gestión de producto y el QA dedicado se estiman **aparte** (ver §7 y §8), porque no los hace el desarrollador senior.

---

## 6. Escenarios de alcance y tiempos

### Escenario A — MVP (validar el mercado)
- **Alcance:** módulos 1 a 16 en su versión mínima. Multi-tenant, maestros, geocercas y rutas, órdenes, **API pública básica con webhooks**, 1 conector GPS, tracking y mapa en vivo, reportes básicos. Sin billing automático (se factura a mano).
- **Tiempo:**
  - 1 senior solo: **11–13 meses**.
  - 2 seniors, o 1 senior + 1 semi-senior, con apoyo de UX: **5–6 meses**.
  - 3 personas: **4–5 meses**.

### Escenario B — V1 comercial ⭐ (recomendado)
- **Alcance:** todo lo del MVP más la plataforma de integración completa (sandbox, portal de desarrolladores, catálogo inicial de conectores), alertas, billing, onboarding, branding, seguridad y pentest.
- **Tiempo:**
  - 1 senior solo: **20–24 meses**. No se recomienda: llega tarde al mercado y concentra todo el riesgo en una persona.
  - Equipo de 4–5 personas: **6–8 meses**.

### Escenario C — Plataforma completa
- **Alcance:** V1 más app del chofer, portales de cliente y de transportista, más conectores, optimización y costos.
- **Tiempo:** equipo de 6–7 personas, **10–13 meses**. Lo habitual es llegar aquí de forma incremental después de lanzar la V1.

> **Por qué un equipo no reduce el tiempo en proporción directa:** a más personas, más coordinación. Por eso 22 meses-persona con 4 personas equivalen a unos 6–8 meses de calendario, no a 5,5.

---

## 7. Recursos humanos: equipo sugerido

### Para la V1 (6–8 meses)

| Rol | Dedicación | Responsabilidad |
|---|---|---|
| **Tech lead / arquitecto senior** (full-stack, backend fuerte) | 100 % | Arquitectura, multi-tenancy, integración y tracking, revisión de código |
| **Desarrollador senior backend / integraciones** | 100 % | API pública, webhooks, conectores, ingesta GPS |
| **Desarrollador semi-senior o senior frontend** | 100 % | Web app, mapas, dashboards, portal de desarrolladores |
| **Diseñador UX/UI** | 30–50 % los primeros 3 meses, luego 20 % | Flujos, sistema de diseño, prototipos |
| **QA / automatización** | 50 % desde el mes 3 | Pruebas E2E, pruebas de carga, pruebas de regresión |
| **DevOps / SRE** | 20–30 % | Infraestructura como código, CI/CD, monitoreo, backups (el tech lead lo puede cubrir si tiene experiencia) |
| **Product owner / experto en logística** | 30–50 % | Prioridades, validación con transportistas, criterios de aceptación. Puede ser el dueño del negocio. |

**Equivalente aproximado:** unas 4,5 personas a tiempo completo.

### Para la versión completa
Añadir **1 desarrollador móvil** (React Native), **1 desarrollador full-stack** más y subir QA a 100 %.

### Si solo hay presupuesto para una persona
Se puede hacer con **1 senior full-stack**, con estas condiciones:
- Limitar el alcance al **MVP**.
- Usar al máximo servicios ya hechos: identidad, pagos, mapas, email.
- Contratar diseño por proyecto (unas 3–4 semanas al inicio).
- Hacer un pentest externo antes del lanzamiento.
- Asumir el **riesgo de dependencia de una sola persona** (bus factor): documentación y ADRs obligatorios desde el día 1.

---

## 8. Costo de mano de obra

### 8.1 Tarifas de referencia de un desarrollador senior

Rangos orientativos, a septiembre de 2026, para un perfil full-stack o backend con experiencia en SaaS, integraciones y datos geoespaciales.

| Modalidad / mercado | Mensual | Por hora (160 h/mes) |
|---|---:|---:|
| **Perú – planilla** (sueldo bruto) | S/ 12.000 – 20.000 (≈ USD 3.200 – 5.400) | — |
| **Perú – planilla, costo total para la empresa**\* | ≈ USD 4.500 – 7.800 | ≈ USD 28 – 49 |
| **LatAm – contratista / freelance** | USD 4.500 – 8.000 | USD 28 – 50 |
| **LatAm – a través de agencia** | USD 7.000 – 11.000 | USD 45 – 70 |
| **EE. UU. / Europa occidental – contratista** | USD 12.000 – 24.000 | USD 75 – 150 |

\* En régimen general peruano, el costo de un trabajador en planilla es aproximadamente un **40–50 % mayor** que su sueldo bruto: gratificaciones (2 sueldos al año), CTS (1 sueldo), vacaciones y EsSalud (9 %). Confírmalo con tu contador según el régimen de tu empresa.

Otros perfiles, como referencia (LatAm, contratista, mensual):

| Perfil | Mensual |
|---|---:|
| Semi-senior | USD 2.500 – 4.500 |
| Diseñador UX/UI senior | USD 3.000 – 5.500 |
| QA automatización | USD 2.500 – 4.500 |
| DevOps senior | USD 4.500 – 8.000 |

### 8.2 Costo de un desarrollador senior según el escenario

Fórmula: **meses-persona × tarifa mensual**.

| Escenario | Meses-persona | @ USD 4.500/mes | @ USD 6.500/mes | @ USD 9.000/mes | @ USD 18.000/mes (EE. UU.) |
|---|---:|---:|---:|---:|---:|
| **MVP** | 10,5 | USD 47.000 | USD 68.000 | USD 95.000 | USD 189.000 |
| **V1 comercial** | 22 | USD 99.000 | USD 143.000 | USD 198.000 | USD 396.000 |
| **Completa** | 42 | USD 189.000 | USD 273.000 | USD 378.000 | USD 756.000 |

**En horas** (V1, 22 meses-persona ≈ 3.520 horas):

| Tarifa por hora | Costo |
|---:|---:|
| USD 30 | ≈ USD 106.000 |
| USD 40 | ≈ USD 141.000 |
| USD 50 | ≈ USD 176.000 |

### 8.3 Costo del equipo completo recomendado (V1, 7 meses, LatAm)

| Rol | Dedicación | Tarifa mensual | Meses | Subtotal |
|---|---:|---:|---:|---:|
| Tech lead senior | 100 % | USD 7.500 | 7 | USD 52.500 |
| Senior backend / integraciones | 100 % | USD 6.500 | 7 | USD 45.500 |
| Frontend semi-senior o senior | 100 % | USD 5.000 | 7 | USD 35.000 |
| UX/UI | ~35 % promedio | USD 4.500 | 7 | USD 11.000 |
| QA automatización | 50 % (desde el mes 3) | USD 3.500 | 5 | USD 8.750 |
| DevOps | 25 % | USD 6.500 | 7 | USD 11.400 |
| **Total mano de obra V1** | | | | **≈ USD 164.000** |

Con tarifas bajas del rango el total ronda **USD 120.000**; con tarifas de agencia, **USD 230.000**. No incluye al product owner (normalmente es el dueño del negocio).

---

## 9. Costos de infraestructura y servicios

Estimación mensual en USD. Depende mucho del volumen de posiciones GPS.

| Concepto | Desarrollo y staging | Producción inicial (≤ 10 tenants, ≤ 500 vehículos) | Crecimiento (≤ 100 tenants, ≤ 5.000 vehículos) |
|---|---:|---:|---:|
| Cómputo (contenedores API, workers, ingesta) | 50 – 120 | 150 – 400 | 800 – 2.500 |
| PostgreSQL + PostGIS + Timescale (gestionado) | 30 – 80 | 150 – 400 | 600 – 2.000 |
| Redis y cola de mensajes | 15 – 40 | 50 – 150 | 150 – 500 |
| Almacenamiento de archivos y backups | 5 – 20 | 20 – 60 | 100 – 400 |
| Mapas (tiles MapLibre / MapTiler) | 0 – 25 | 25 – 150 | 150 – 800 |
| Identidad (Clerk / Auth0 / WorkOS) | 0 | 0 – 150 | 200 – 1.000 |
| Email transaccional | 0 – 20 | 20 – 50 | 50 – 200 |
| WhatsApp Business API | — | según uso (≈ USD 0,01–0,08 por conversación o plantilla) | según uso |
| Observabilidad (Sentry, Grafana/Datadog) | 0 – 30 | 30 – 150 | 200 – 1.000 |
| CDN, dominio, certificados, DNS | 5 – 20 | 20 – 50 | 50 – 150 |
| **Total aproximado** | **USD 100 – 350** | **USD 450 – 1.600** | **USD 2.300 – 8.500** |

**Herramientas del equipo** (por mes): GitHub, Linear o Jira, Figma, Postman, Google Workspace. Suman aproximadamente **USD 100 – 300**.

---

## 10. Otros costos únicos

| Concepto | Rango estimado (USD) |
|---|---:|
| Pentest externo antes del lanzamiento | 3.000 – 10.000 |
| Asesoría legal: términos y condiciones, política de privacidad (Ley 29733), contrato de encargo de tratamiento de datos (DPA), SLA | 1.500 – 5.000 |
| Marca, landing page y material comercial | 1.500 – 6.000 |
| Cuentas de prueba con proveedores GPS y ERP, y dispositivos GPS de prueba | 500 – 2.000 |
| Integración con facturación electrónica (SUNAT, vía proveedor OSE/PSE), si se factura desde la plataforma | 1.000 – 3.000 + costo mensual del proveedor |
| Publicación en tiendas de apps (solo si hay app móvil) | ~125 (Apple USD 99/año + Google USD 25 único) |

---

## 11. Costo total estimado

Estimación para el **escenario B (V1 comercial)**, con equipo LatAm y 7 meses de desarrollo:

| Rubro | Bajo | Medio | Alto |
|---|---:|---:|---:|
| Mano de obra del equipo | 120.000 | 164.000 | 230.000 |
| Infraestructura y herramientas durante el desarrollo (7 meses) | 1.500 | 3.000 | 5.000 |
| Costos únicos (pentest, legal, marca, pruebas) | 6.500 | 13.000 | 23.000 |
| **Total para lanzar la V1** | **≈ USD 128.000** | **≈ USD 180.000** | **≈ USD 258.000** |
| **Operación mensual después del lanzamiento** (infraestructura + 1–2 personas de mantenimiento y evolución) | 5.000 | 9.000 | 15.000 |

**Solo con un desarrollador senior** (MVP, aproximadamente 12 meses):

| Rubro | Rango |
|---|---:|
| Mano de obra | USD 47.000 – 95.000 |
| Diseño por proyecto | USD 4.000 – 8.000 |
| Pentest, legal e infraestructura | USD 8.000 – 18.000 |
| **Total** | **≈ USD 59.000 – 121.000** |

---

## 12. Cronograma por fases

V1 con un equipo de 4–5 personas.

```
Mes:            1     2     3     4     5     6     7     8
Descubrimiento ████
Plataforma base   ██████████
DevOps/CI       ██████                                  (continuo)
Maestros              ████████
Geo + rutas               ████████
Órdenes                     ██████████
Integración API/WH            ████████████████
Conectores                          ██████████████
Tracking + ETA                  ████████████████
Mapa en vivo                            ████████
Alertas/Notif.                               ██████
Dashboards                                   ████████
Billing/Admin                                  ████████
QA/Seguridad                  ▒▒▒▒▒▒▒▒▒▒▒▒▒▒████████████
Piloto 1er cliente                      ▲ MVP (mes 4–5)
Lanzamiento V1                                         ▲ (mes 7–8)
```

**Hitos:**

| Mes | Hito |
|---|---|
| 1 | Arquitectura, contratos de API y diseño aprobados |
| 3 | Plataforma base, maestros y órdenes funcionando en staging |
| 4–5 | **MVP**: primer cliente piloto con tracking real y API funcionando |
| 6 | Portal de desarrolladores, conectores y billing |
| 7–8 | Pentest resuelto y **lanzamiento comercial de la V1** |

---

## 13. Riesgos que mueven la estimación

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Proveedores GPS y ERP con API mala, sin documentación o sin entorno de pruebas | +1–3 semanas por conector | Priorizar pocos conectores; ofrecer primero **nuestra** API estándar para que ellos se adapten |
| Alcance "a medida" que se cuela por cada cliente nuevo | Retrasos continuos | Todo lo específico va a configuración o campos personalizados; sin forks por cliente |
| Volumen de posiciones GPS mayor al previsto | Mayor costo de infraestructura y rediseño | Series temporales desde el día 1, pruebas de carga en el mes 3 |
| Una sola persona desarrollando | Riesgo de dependencia, lentitud | Documentación y ADRs, código revisado, pruebas automatizadas |
| Requisitos legales (datos personales, facturación) | Retrasos al lanzar | Asesoría legal temprana |
| Migración de datos del TMS actual (esquema desalineado) | +2–4 semanas | Volcar el esquema real de producción al inicio |
| ETA preciso (tráfico, paradas) | Expectativas del cliente | Empezar con ETA por distancia y velocidad promedio; pasar luego a un motor de rutas o ML |

---

## 14. Cómo reducir tiempo y costo

1. **Comprar en vez de construir:**
   - identidad: Clerk, Auth0 o WorkOS (ahorra unas 3–4 semanas);
   - pagos: Stripe (unas 2–3);
   - mapas: MapTiler;
   - email: Resend;
   - webhooks salientes: Svix o Hookdeck (unas 2–3);
   - portal de API: Mintlify, Redocly o Scalar (1–2).
2. **Reutilizar el conocimiento del TMS actual.** El modelo de órdenes, rutas de geocercas, hitos y semáforo, las reglas de validación y el contrato con Navitel ya están probados en producción (ver `ANALISIS-TMS-SAAS.md`). Eso reduce el descubrimiento y los errores de dominio.
3. **Monolito modular.** Nada de microservicios al inicio: ahorra DevOps y depuración.
4. **Lanzar con el MVP** y financiar la V1 con los primeros clientes.
5. **Plantillas en lugar de desarrollo a medida.** Los tipos de operación, campos personalizados y reglas configurables evitan desarrollar algo nuevo por cada cliente.
6. **Contratar en LatAm** con un tech lead senior fuerte que revise el trabajo del resto del equipo.

---

## 15. Si vas a cotizarlo a un cliente

- Las cifras anteriores son **costo** de mano de obra. Si lo vas a vender como proyecto, suma:
  - gestión de proyecto (10–15 %);
  - margen de la empresa (20–40 %);
  - garantía o soporte posterior (por ejemplo, 3 meses).
- Cotiza **por fases con entregables** (MVP → V1 → módulos adicionales), no como un precio cerrado del sistema completo: el riesgo de alcance es alto.
- Define por escrito **qué conectores incluye** el precio. Cada conector adicional se cotiza aparte, con una referencia de 1–2 semanas-senior cada uno.
- Aclara la **propiedad del código** y el **modelo de negocio**: ¿tú desarrollas y el cliente es dueño del producto, o el SaaS es tuyo y el cliente paga una suscripción?
- Ejemplo de precio de venta para la V1, partiendo de un costo medio de USD 180.000: con gestión y un margen del 30 %, el precio quedaría en **USD 250.000 – 270.000**. Es solo una referencia; ajústala a tu mercado y relación con el cliente.
