# Plan de tareas y horas — Cambios flujo 261020 (panel.propietario)

**Base de cálculo:** 1 desarrollador senior Bitrix, lunes a viernes, 8 h/día (40 h/semana). Inicio: lunes 26/10/2026. Horas = valor probable (promedio de mín–máx del Excel `Estimacion_Cambios_Flujo_261020_v2.xlsx`).
**Fecha objetivo 31/10/2026:** solo caben 40 h (26–30 oct). No se llega; abajo se indica qué entra en esa semana y la fecha real de término.

---

## Fase 1 — Cambios que afectan el flujo principal (282 h dev + 76 h QA/integración = 358 h)

### Bloque 0 — Fundaciones (66 h) · prerrequisito de todo lo demás

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 0.1 | Enum central de estados + motivo de cierre | Modificación | 12 |
| 0.2 | Tabla de transiciones estado × rol (reemplaza los `if` de GridColumnBuilder) | Modificación | 14 |
| 0.6 | Unificar roles IFS/GCB + validar rol en acciones AJAX | Modificación | 8 |
| 0.3 | Servicio de notificaciones (IM + push al portal; sin correo) | Desarrollo | 10 |
| 0.4 | Webhook inbound genérico con eventos tipados + API key en settings | Modificación | 12 |
| 0.5 | Entidad genérica de propuestas de fecha (tipo + nº intento) | Modificación | 10 |

### Bloque 1 — Registro y validación (13 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 1.1 | Mover evaluación inicial de SAC a Arquitecto | Modificación | 3 |
| 1.2 | Recibir subsanación por webhook + estado SUBSANADA | Desarrollo | 5 |
| 1.3 | Modal "Revisar Subsanación" (SAC) con ¿Cumple Requerimientos? | Modificación | 5 |

### Bloque 2 — Visita inicial (24 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 2.1 | Rechazo de fechas de visita + loop de re-propuesta + historial | Modificación | 10 |
| 2.2 | Confirmar disponibilidad del arquitecto antes de asignar | Modificación | 5 |
| 2.3 | Informe de campo (formulario + adjuntos + HL) | Desarrollo | 9 |

### Bloque 3 — Garantía (39 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 3.1 | Cálculo automático de garantía en servidor | Modificación | 10 |
| 3.2 | ¿Se asumirá? con aprobación del Aprobador | Desarrollo | 10 |
| 3.3 | Dictamen con ¿Proveedor Asume Garantía? / ¿Procede con el Informe? | Modificación | 7 |
| 3.4 | Cierre con respuesta formal al cliente (reutilizable en 3 puntos) | Desarrollo | 12 |

### Bloque 4 — Ramal proveedor (38 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 4.1 | Coordinación con proveedor + ¿Realizará Labor? + señal | Desarrollo | 12 |
| 4.2 | ¿Necesita Visita de Proveedor? + ciclo de fechas vía SAC | Desarrollo | 14 |
| 4.3 | Visita conjunta + informe con proveedor + ¿Procede garantía? | Desarrollo | 12 |

### Bloque 5 — OC / OS (8 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 5.1 | Recibir OS + coordinar fechas de servicio hacia el plan | Modificación | 4 |
| 5.2 | "Ambos OC y OS": join antes del plan | Modificación | 4 |

### Bloque 6 (parte Fase 1) — Almacén (10 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 6.2 | Cotizar OC antes de solicitar aprobación (Logística) | Modificación | 10 |

### Bloque 8 — Recepción (6 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 8.1 | Confirmar fecha de entrega + entradas desde traslado/compra directa | Modificación | 6 |

### Bloque 9 — Plan y programación (12 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 9.1 | Separar "Programar Trabajo" como punto de reentrada | Modificación | 7 |
| 9.2 | SAC notifica fechas + loop de rechazo + tope de intentos | Modificación | 5 |

### Bloque 10 — Ejecución y acta (51 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 10.1 | Registro visual + elaborar acta + ¿Se culminó trabajo? | Modificación | 7 |
| 10.2 | Acta de no concluido + reprogramación + historial de ejecuciones (1:N) | Desarrollo | 15 |
| 10.3 | Solicitar firma + firma/rechazo desde portal + versionado de actas | Desarrollo | 19 |
| 10.4 | Validar rechazo + ¿Procede el Rechazo? | Desarrollo | 10 |

### Bloque 11 — Cierre SAC (15 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 11.1 | Ingresar acta (SAC) + Acta Aprobada? + cierre con acta rechazada | Modificación | 5 |
| 11.2 | Resumen/informe + cerrar trabajo + Drive + file físico | Modificación | 5 |
| 11.3 | Culminar + notificar al cliente | Modificación | 5 |

### Transversal (76 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| T2 | Contratos e integración con Spring (lado Bitrix) | Integración | 20 |
| T1 | QA / regresión E2E (loops, 4 cierres, reprogramaciones) | QA | 56 |

**Subtotal Fase 1: 358 h**

---

## Fase 2 — Almacén avanzado y aprobación por montos (90 h)

| ID | Tarea | Tipo | Horas |
|---|---|---|---|
| 6.1 | Stock multi-almacén + traslado interno | Desarrollo | 17 |
| 6.3 | Compra directa por SAC + factura a Tesorería | Desarrollo | 19 |
| 6.4 | Compra acumulada (una OC para varias incidencias) — *confirmar con negocio* | Desarrollo | 25 |
| 7.1 | Tabla de umbrales + monto en la solicitud | Desarrollo | 12 |
| 7.2 | Ruteo al aprobador por monto + rama de rechazo | Modificación | 10 |
| 7.3 | 2º nivel Encargado de costo antes de crear OC/OS | Modificación | 7 |

**Subtotal Fase 2: 90 h**

---

## TOTAL: 448 h = 56 días hábiles = 11,2 semanas (1 dev)

---

## Calendario proyectado (1 dev, L–V 8 h, desde 26/10/2026)

| Hito | Horas acum. | Fecha de término |
|---|---|---|
| Semana del 26–30 oct (fecha objetivo) | 40 | Solo 0.1, 0.2, 0.6 y parte de 0.3. **Nada visible para negocio.** |
| Bloque 0 Fundaciones | 66 | jue 05/11 |
| Bloque 1 Validación | 79 | vie 06/11 |
| Bloque 2 Visita inicial | 103 | mié 11/11 |
| Bloque 3 Garantía | 142 | mié 18/11 |
| Bloque 4 Ramal proveedor | 180 | mié 25/11 |
| Bloques 5, 6.2, 8, 9 | 216 | mar 01/12 |
| Bloque 10 Ejecución y acta | 267 | jue 10/12 |
| Bloque 11 Cierre | 282 | lun 14/12 |
| T2 Integración Spring | 302 | mié 16/12 |
| T1 QA | 358 | **vie 25/12 → Fase 1 lista** (con feriados 08/12 y 25/12: mié 30/12) |
| Fase 2 (6.1, 6.3, 6.4, 7.x) | 448 | **lun 11/01/2027** (con feriados 08/12, 25/12, 01/01: jue 14/01/2027) |

### Con 2 desarrolladores (en paralelo desde el bloque 1; el bloque 0 es secuencial)

| Hito | Fecha de término |
|---|---|
| Fase 1 | ≈ vie 04/12/2026 |
| Fase 1 + Fase 2 | ≈ vie 11/12/2026 |

---

## Supuestos

- No incluye el trabajo del lado Spring Boot / portal del propietario (rechazo de fechas, firma de acta, subsanación, cálculo de garantía con fecha de entrega). Si ese equipo se retrasa, 2.1, 4.2, 9.2, 10.3 y T2 se bloquean.
- No incluye correo electrónico en notificaciones (+5 h si se requiere).
- No incluye corrección de los 19 hallazgos de código ajenos al cambio de flujo (hoja "Hallazgos código"); los de severidad Alta que bloquean el flujo (GCB:309, reprogramación inalcanzable, acta rechazada → CONCLUIDO) ya están absorbidos en 0.2, 10.2 y 10.4.
- Las 15 preguntas a negocio pendientes pueden mover ±15 % las horas de los bloques 3, 4, 6 y 7.
- Feriados considerados solo como nota (08/12, 25/12, 01/01); no se descontaron vacaciones ni soporte a producción.
