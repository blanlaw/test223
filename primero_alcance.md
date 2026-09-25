# Alcance 1 – Centralización de datos GPS (Fases 0 a 3)

> **Fecha:** 2026-09-25 · *Cifras referenciales; hay que validarlas.*
>
> **Supuestos:**
> - **Equipo:** 2 desarrolladores senior, 4 h diarias, lunes a viernes.
> - **Sueldo:** S/ 4.000 – 5.000 al mes por desarrollador.
> - **Tipo de cambio referencial:** S/ 3,70 por USD.
>
> **Roles:**
> - **D1:** senior backend / integraciones (líder técnico)
> - **D2:** senior full-stack
>
> **Objetivo:** recibir, validar, identificar y guardar las posiciones (lat, lng, velocidad, rumbo, hora, etc.) que envían los proveedores GPS y los equipos, con la base de empresas, usuarios y maestros que se necesita para ello.

---

## 1. Alcance

| Incluye ✅ | Queda para el siguiente alcance ❌ |
|---|---|
| Empresas, usuarios, roles y permisos | Mapa en vivo, recorrido por tramos, reproducción |
| Transportistas, vehículos, conductores, dispositivos, proveedores GPS | Geocercas, paradas, alertas |
| API de recepción de posiciones | API pública de consulta, webhooks, enlace de tracking |
| Adaptador Navitel V2, pasarela Traccar, consulta a APIs de proveedores | Portales de proveedores y de desarrolladores |
| Validación, deduplicación, cuarentena de unidades desconocidas | Órdenes de transporte, hitos, ETA |
| Almacenamiento histórico (TimescaleDB) y última posición | Reportes y dashboards |
| Pantalla de verificación de posiciones recibidas (tabla) | |

---

## 2. Tareas

### Fase 0 – Preparación

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 0.1 | Especificación del formato de retransmisión (OpenAPI) | D1 | 32 |
| 0.2 | Arquitectura, modelo de datos, ADRs | D1 | 24 |
| 0.3 | Repositorios, CI/CD, entornos dev/staging/prod | D2 | 32 |
| 0.4 | Wireframes de las pantallas | D2 | 24 |
| | **Subtotal** | | **112** |

### Fase 1 – Plataforma base

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 1.1 | Multi-tenant (empresas) con Row-Level Security | D1 | 40 |
| 1.2 | Autenticación: login, recuperación, invitaciones | D2 | 32 |
| 1.3 | Roles y permisos (RBAC) | D1 | 32 |
| 1.4 | Gestión de usuarios (API + UI) | D2 | 24 |
| 1.5 | Auditoría de acciones | D2 | 16 |
| 1.6 | Configuración de la empresa | D2 | 16 |
| | **Subtotal** | | **160** |

### Fase 2 – Maestros

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 2.1 | Transportistas (CRUD) | D2 | 24 |
| 2.2 | Vehículos (CRUD) | D2 | 24 |
| 2.3 | Conductores (CRUD) | D2 | 24 |
| 2.4 | Dispositivos GPS (IMEI, proveedor) y vínculo con el vehículo | D1 | 24 |
| 2.5 | Proveedores GPS: registro y credenciales de API | D2 | 24 |
| 2.6 | Importación masiva CSV/Excel | D2 | 32 |
| 2.7 | Documentos con vencimiento | D2 | 24 |
| 2.8 | Autorización del transportista para compartir sus unidades | D1 | 24 |
| | **Subtotal** | | **200** |

### Fase 3 – Recepción y centralización de datos

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 3.1 | API de ingesta `POST /v1/positions:batch`: autenticación por proveedor, validación, respuesta 202 | D1 | 40 |
| 3.2 | Rate limit por proveedor, idempotencia y deduplicación | D1 | 24 |
| 3.3 | Cola durable, DLQ y reintentos | D1 | 32 |
| 3.4 | Worker de calidad del dato: coordenadas, fechas, saltos, puntos desordenados | D1 | 32 |
| 3.5 | Identificación de la unidad y cuarentena de unidades no vinculadas | D1 | 24 |
| 3.6 | Almacenamiento en TimescaleDB (compresión, retención) y guardado del mensaje original | D1 | 24 |
| 3.7 | Última posición en Redis | D1 | 8 |
| 3.8 | Adaptador de retransmisión Navitel V2 | D1 | 32 |
| 3.9 | Consulta periódica (pull) a APIs de proveedores | D2 | 24 |
| 3.10 | Pasarela Traccar para equipos directos | D2 | 40 |
| 3.11 | Pruebas de carga de la ingesta (k6) | D2 | 24 |
| 3.12 | Pantalla de verificación: últimas posiciones por unidad y mensajes recibidos | D2 | 24 |
| | **Subtotal** | | **328** |

---

## 3. Resumen de esfuerzo y tiempo

| Concepto | D1 | D2 | Total |
|---|---:|---:|---:|
| Horas estimadas | 392 | 408 | **800** |
| Contingencia (+20 %) | 78 | 82 | 160 |
| **Horas con contingencia** | **470** | **490** | **960** |
| Jornadas de 4 h | 118 | 123 | — |
| Semanas (20 h por semana) | 24 | 25 | — |
| **Duración en calendario** (incluye feriados) | | | **≈ 6 meses** |

---

## 4. Cronograma

| Fase | M1 | M2 | M3 | M4 | M5 | M6 |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| 0. Preparación | ██ | | | | | |
| 1. Plataforma base | █ | ██ | | | | |
| 2. Maestros | | ██ | ██ | █ | | |
| 3. Recepción y centralización | | | █ | ██ | ██ | ██ |
| Pruebas de carga, ajustes, contingencia | | | | | █ | ██ |

| Mes | Entregable |
|---|---|
| 1 | Especificación de la API de retransmisión, arquitectura, entornos listos |
| 2 | Empresas, usuarios, roles y permisos funcionando |
| 3 | Transportistas, vehículos, conductores, importación masiva |
| 4 | Dispositivos, proveedores, autorización; **primeras posiciones recibidas y guardadas** |
| 5 | Cola, validación de calidad, TimescaleDB, adaptador Navitel |
| 6 | Traccar, pull, pruebas de carga, pantalla de verificación → **entrega del Alcance 1** |

---

## 5. Criterios de aceptación

| Criterio | Meta |
|---|---|
| Unidades transmitiendo a la vez | ≥ 500 unidades cada 30 s, sin pérdida de datos |
| Latencia desde la recepción hasta que el dato queda guardado | < 5 s (p95) |
| Duplicados | 0 guardados (se reenvía un lote y no se duplica) |
| Caída del procesador | Al volver, no se pierde ningún mensaje de la cola |
| Unidad desconocida | Queda en cuarentena y es visible en la pantalla de verificación |
| Fuentes funcionando | Navitel (retransmisión), 1 equipo directo vía Traccar, 1 API consultada por pull |
| Aislamiento | Un proveedor solo puede enviar datos de sus unidades vinculadas; una empresa solo ve lo suyo |

---

## 6. Costo

### 6.1 Mano de obra (2 desarrolladores)

| Modalidad | Por desarrollador / mes | 2 desarrolladores / mes | **6 meses** | USD |
|---|---:|---:|---:|---:|
| Recibo por honorarios – bajo | S/ 4.000 | S/ 8.000 | **S/ 48.000** | ≈ 13.000 |
| **Recibo por honorarios – medio** ⭐ | S/ 4.500 | S/ 9.000 | **S/ 54.000** | ≈ 14.600 |
| Recibo por honorarios – alto | S/ 5.000 | S/ 10.000 | **S/ 60.000** | ≈ 16.200 |
| Planilla (bruto S/ 4.500; costo empresa ≈ +45 %) | S/ 6.525 | S/ 13.050 | **S/ 78.300** | ≈ 21.200 |

### 6.2 Otros costos (6 meses)

| Concepto | Bajo | Medio | Alto |
|---|---:|---:|---:|
| Infraestructura dev/staging, herramientas, servicios | S/ 3.300 | S/ 6.700 | S/ 11.100 |
| Legal: contratos con proveedores, privacidad (Ley 29733) | S/ 3.700 | S/ 7.400 | S/ 11.100 |

### 6.3 Total del Alcance 1

| Rubro | Bajo | Medio ⭐ | Alto | Planilla |
|---|---:|---:|---:|---:|
| Mano de obra | S/ 48.000 | S/ 54.000 | S/ 60.000 | S/ 78.300 |
| Infraestructura, herramientas, servicios | S/ 3.300 | S/ 6.700 | S/ 11.100 | S/ 6.700 |
| Legal | S/ 3.700 | S/ 7.400 | S/ 11.100 | S/ 7.400 |
| **Total** | **S/ 55.000** | **S/ 68.100** | **S/ 82.200** | **S/ 92.400** |
| **Total en USD** | **≈ 14.900** | **≈ 18.400** | **≈ 22.200** | **≈ 25.000** |

---

## 7. Siguiente alcance sugerido

| Alcance | Contenido | Tiempo | Mano de obra (medio) |
|---|---|---:|---:|
| **Alcance 2 – Visualización y exposición** | Mapa en vivo, tramos, reproducción, geocercas, paradas, alertas, API pública, webhooks, portales, piloto (Fases 4 a 7) | ≈ 5 meses | ≈ S/ 45.000 |
| **Alcance 3 – TMS operativo** | Órdenes, hitos, ETA, semáforo, reportes, dashboards, conector ERP | ≈ 7–8 meses | ≈ S/ 63.000 – 72.000 |
