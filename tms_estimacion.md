# Estimación – Etapa 1: Colector de posiciones GPS y punto de integración de proveedores

> **Fecha:** 2026-09-25 · *Cifras referenciales; hay que validarlas.*
>
> **Supuestos:**
> - **Equipo:** 2 desarrolladores senior en Perú.
> - **Jornada:** 4 h diarias, de lunes a viernes (≈ 87 h al mes por desarrollador).
> - **Sueldo:** S/ 4.000 – 5.000 al mes por desarrollador. Asumido **en soles**.
> - **Unidad de esfuerzo:** horas-desarrollador.
>
> **Roles:**
> - **D1:** senior backend / integraciones (líder técnico)
> - **D2:** senior full-stack (frontend fuerte)

---

## 1. Alcance de la etapa

| Incluye ✅ | No incluye ❌ (etapas siguientes) |
|---|---|
| Empresas (multi-tenant), usuarios, roles y permisos | Órdenes de transporte completas con hitos y citas |
| Transportistas, vehículos, conductores, dispositivos GPS, proveedores GPS | ETA con motor de rutas y semáforo de citas |
| Recepción de posiciones: API, adaptador Navitel, equipos directos, consulta a APIs | Reportes avanzados y dashboards de KPI |
| Procesamiento: validación, deduplicación, geocercas, paradas, sin señal | Billing y planes |
| Mapa en vivo, recorrido por tramos, reproducción | App móvil del chofer |
| Exposición: API pública, webhooks, enlace de tracking | Conectores ERP (Nisira, etc.) |
| Portal de proveedores y portal de desarrolladores, calidad por proveedor | |

---

## 2. Tareas y estimación (en horas)

### Fase 0 – Preparación

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 0.1 | Especificación del formato de retransmisión (OpenAPI) | D1 | 32 |
| 0.2 | Arquitectura, modelo de datos, ADRs | D1 | 24 |
| 0.3 | Repositorios, CI/CD, entornos dev/staging/prod | D2 | 32 |
| 0.4 | Wireframes de las pantallas clave | D2 | 24 |
| | **Subtotal** | | **112** |

### Fase 1 – Plataforma base

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 1.1 | Multi-tenant (empresas) con Row-Level Security | D1 | 40 |
| 1.2 | Autenticación: login, recuperación, invitaciones (IdP) | D2 | 32 |
| 1.3 | Roles y permisos (RBAC) | D1 | 32 |
| 1.4 | Gestión de usuarios (API + UI) | D2 | 24 |
| 1.5 | Auditoría de acciones | D1 | 16 |
| 1.6 | Configuración de la empresa (datos, zona horaria, logo) | D2 | 16 |
| | **Subtotal** | | **160** |

### Fase 2 – Maestros

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 2.1 | Transportistas (CRUD) | D2 | 24 |
| 2.2 | Vehículos (CRUD: tracto, carreta, placa) | D2 | 24 |
| 2.3 | Conductores (CRUD) | D2 | 24 |
| 2.4 | Dispositivos GPS (IMEI, proveedor) y vínculo con el vehículo | D1 | 24 |
| 2.5 | Proveedores GPS: registro y credenciales de API | D1 | 24 |
| 2.6 | Importación masiva CSV/Excel (vehículos, conductores) | D2 | 32 |
| 2.7 | Documentos con vencimiento (brevete, SOAT, revisión técnica) | D2 | 24 |
| 2.8 | Autorización del transportista para compartir sus unidades (consentimiento) | D1 | 24 |
| | **Subtotal** | | **200** |

### Fase 3 – Recepción de datos (el colector)

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
| 3.9 | Consulta periódica (pull) a APIs de proveedores | D1 | 24 |
| 3.10 | Pasarela Traccar para equipos directos e integración | D2 | 40 |
| 3.11 | Pruebas de carga de la ingesta (k6) | D2 | 24 |
| | **Subtotal** | | **304** |

### Fase 4 – Procesamiento

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 4.1 | Geocercas: CRUD y dibujo en el mapa | D2 | 40 |
| 4.2 | Motor de entrada y salida de geocercas (PostGIS) | D1 | 32 |
| 4.3 | Asignación unidad ↔ viaje/orden con ventana de tiempo (permisos de visibilidad) | D1 | 32 |
| 4.4 | Detección de paradas y de "sin transmisión" | D1 | 24 |
| 4.5 | Alertas básicas por email: sin señal, velocidad, geocerca | D1 | 32 |
| | **Subtotal** | | **160** |

### Fase 5 – Mapa

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 5.1 | Mapa en vivo (MapLibre + WebSocket por empresa, marcadores en movimiento) | D2 | 40 |
| 5.2 | Recorrido por tramos: API, SQL y dibujo (huecos de señal, color por velocidad) | D2 | 32 |
| 5.3 | Paradas e información de cada punto | D2 | 16 |
| 5.4 | Reproducción del viaje (línea de tiempo) | D2 | 32 |
| 5.5 | Lista de unidades con estado (en línea / sin señal) y filtros | D2 | 24 |
| | **Subtotal** | | **144** |

### Fase 6 – Exposición e integración

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 6.1 | API pública de consulta (posición, recorrido, eventos) | D1 | 32 |
| 6.2 | Webhooks salientes: suscripción, firma HMAC, reintentos, registro | D1 | 40 |
| 6.3 | API keys / OAuth para clientes, con alcances | D1 | 24 |
| 6.4 | Enlace público de tracking | D2 | 24 |
| 6.5 | Portal de desarrolladores: documentación, sandbox, guías | D2 | 32 |
| 6.6 | Portal de proveedores: alta, credenciales, certificación automática | D2 | 32 |
| | **Subtotal** | | **184** |

### Fase 7 – Calidad, operación y piloto

| # | Tarea | Resp. | Horas |
|---|---|:---:|---:|
| 7.1 | Panel de calidad por proveedor (latencia, % transmitiendo) | D2 | 24 |
| 7.2 | Observabilidad (logs, métricas, alertas) y backups | D1 | 24 |
| 7.3 | Pruebas E2E y de regresión | D2 | 32 |
| 7.4 | Hardening de seguridad (OWASP) | D1 | 24 |
| 7.5 | Documentación y despliegue a producción | D1 | 16 |
| 7.6 | Documentación y despliegue a producción | D2 | 16 |
| 7.7 | Piloto con un proveedor real y ajustes | D1 | 24 |
| 7.8 | Piloto con un proveedor real y ajustes | D2 | 24 |
| | **Subtotal** | | **184** |

---

## 3. Resumen de esfuerzo y tiempo

| Concepto | D1 | D2 | Total |
|---|---:|---:|---:|
| Horas estimadas | 760 | 688 | **1.448** |
| Contingencia (+20 %) | 152 | 138 | 290 |
| **Horas con contingencia** | **912** | **826** | **1.738** |
| Jornadas de 4 h (lunes a viernes) | 228 | 207 | — |
| Semanas (20 h por semana) | 46 | 41 | — |
| **Duración en calendario** (en paralelo, incluye feriados) | | | **≈ 11 meses** |

> Con jornada de 8 h la misma etapa tomaría **≈ 5 meses**. Con 4 h diarias el calendario se duplica, aunque el costo total por hora es similar.

---

## 4. Cronograma (meses)

| Fase | M1 | M2 | M3 | M4 | M5 | M6 | M7 | M8 | M9 | M10 | M11 |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 0. Preparación | ██ | | | | | | | | | | |
| 1. Plataforma base | █ | ██ | | | | | | | | | |
| 2. Maestros | | ██ | ██ | ██ | | | | | | | |
| 3. Recepción | | | ██ | ██ | ██ | ██ | | | | | |
| 4. Procesamiento | | | | | ██ | ██ | ██ | | | | |
| 5. Mapa | | | | | ██ | ██ | ██ | ██ | | | |
| 6. Exposición | | | | | | | ██ | ██ | ██ | | |
| 7. Calidad y piloto | | | | | | | | ██ | ██ | ██ | |
| Contingencia | | | | | | | | | | | ██ |

### Entregables por mes

| Mes | Entregable |
|---|---|
| 2 | Plataforma base: empresas, usuarios, roles; especificación de la API publicada |
| 4 | Maestros completos; primera posición recibida y guardada |
| 6 | Colector completo (Navitel, Traccar, pull) |
| 7 | Mapa en vivo, geocercas, alertas |
| 8 | Recorrido por tramos y reproducción del viaje |
| 9 | API pública, webhooks, portales de proveedores y de desarrolladores |
| 10 | Piloto con un proveedor real |
| 11 | Ajustes finales y **producción** |

---

## 5. Costo de 2 desarrolladores (4 h diarias, S/ 4.000 – 5.000 cada uno)

Tipo de cambio referencial: **S/ 3,70 por USD**.

### 5.1 Por mes

| Modalidad | Por desarrollador / mes | Valor hora aproximado | **2 desarrolladores / mes** | USD / mes |
|---|---:|---:|---:|---:|
| Recibo por honorarios – bajo | S/ 4.000 | S/ 46 | **S/ 8.000** | ≈ 2.160 |
| **Recibo por honorarios – medio** ⭐ | S/ 4.500 | S/ 52 | **S/ 9.000** | ≈ 2.430 |
| Recibo por honorarios – alto | S/ 5.000 | S/ 57 | **S/ 10.000** | ≈ 2.700 |
| Planilla (bruto S/ 4.500; costo empresa ≈ +45 %)\* | S/ 6.525 | S/ 75 | **S/ 13.050** | ≈ 3.530 |

\* En planilla, con **4 h diarias exactas** corresponden los beneficios completos: gratificaciones, CTS, vacaciones y EsSalud. El régimen de tiempo parcial solo aplica con **menos** de 4 h diarias en promedio. Validar con el contador.

### 5.2 Por la etapa completa (11 meses)

| Modalidad | Total en soles | Total en USD |
|---|---:|---:|
| Recibo por honorarios – bajo (S/ 4.000) | S/ 88.000 | ≈ 23.800 |
| **Recibo por honorarios – medio (S/ 4.500)** ⭐ | **S/ 99.000** | **≈ 26.750** |
| Recibo por honorarios – alto (S/ 5.000) | S/ 110.000 | ≈ 29.700 |
| Planilla (bruto S/ 4.500) | S/ 143.550 | ≈ 38.800 |

---

## 6. Otros costos de la etapa (11 meses)

| Concepto | Mensual | Total etapa |
|---|---:|---:|
| Infraestructura dev/staging | USD 100 – 250 | USD 1.100 – 2.750 |
| Infraestructura de producción (últimos 2 meses) | USD 150 – 400 | USD 300 – 800 |
| Herramientas (repositorio, tablero de tareas, Figma, etc.) | USD 50 – 150 | USD 550 – 1.650 |
| Servicios: IdP, mapas, email (tier inicial) | USD 0 – 100 | USD 0 – 1.100 |
| Legal: privacidad, contratos con proveedores, registro de banco de datos (Ley 29733) | — | USD 1.000 – 3.000 |
| Pentest externo (opcional) | — | USD 3.000 – 8.000 |

---

## 7. Total estimado de la etapa

| Rubro | Bajo | Medio ⭐ | Alto | Planilla |
|---|---:|---:|---:|---:|
| 2 desarrolladores × 11 meses | S/ 88.000 | S/ 99.000 | S/ 110.000 | S/ 143.550 |
| Infraestructura, herramientas y servicios | S/ 7.000 | S/ 13.000 | S/ 20.000 | S/ 13.000 |
| Legal | S/ 3.700 | S/ 7.400 | S/ 11.100 | S/ 7.400 |
| Pentest | — | — | S/ 29.600 | — |
| **Total** | **S/ 98.700** | **S/ 119.400** | **S/ 170.700** | **S/ 163.950** |
| **Total en USD** | **≈ 26.700** | **≈ 32.300** | **≈ 46.100** | **≈ 44.300** |
| **Operación mensual posterior** (infraestructura + mantenimiento parcial) | S/ 4.000 | S/ 6.000 | S/ 9.000 | S/ 7.500 |

---

## 8. Riesgos de trabajar a 4 h diarias

| Riesgo | Mitigación |
|---|---|
| El calendario se alarga (≈ 11 meses frente a ≈ 5) | Lanzar un MVP interno en el mes 6–7 (colector + mapa en vivo) |
| Menor disponibilidad para incidencias o para el piloto | Definir un horario fijo común y una guardia acordada durante el piloto |
| Los desarrolladores pueden tener otros trabajos en paralelo | Contrato con dedicación y horario claros, y entregables mensuales |
| Sueldo S/ 4.000 – 5.000 por 4 h puede ser bajo para un senior con experiencia en GPS, PostGIS y colas | Validar el perfil con una prueba técnica; considerar semi-senior + revisión de un senior |
