# Estado del desarrollo frente al FRD v5.1

**Fecha:** 28 de septiembre de 2026 · **Documento de requerimientos:** `WAI_Master_FRD_v5.1_Borrador_Integral.docx`
(revisado completo: 602 párrafos, 19 tablas; sin imágenes, comentarios ni cambios con control).
**Base:** validación requisito por requisito del 27-sep (3 revisores con evidencia en el código)
más lo construido después (T10, cierre del punto 9 y proveedor WhatsApp WaSender).
**Portal DEV:** https://srv1974730.hstgr.cloud · **Backlog:** `tasks.md`

## Leyenda

| Símbolo | Significado |
|---|---|
| ✅ | Hecho |
| 🔑 | Hecho; falta la credencial del proveedor para encenderlo |
| ⚠️ | Parcial (se indica qué falta) |
| ❌ | No hecho |
| ⚖️ | Depende de un dato o de una decisión de Wideline |

## Cuadro por requisito

| # | FRD | Requisito | Estado | Tareas | Dónde validar | Qué falta |
|---|---|---|---|---|---|---|
| 1 | §3.2 | KPIs: adopción, tickets, correos, recordatorios, no resueltas, más usadas | ⚠️ | T6.8, T8.8 | /kpis | Adopción por área y algunos conteos (T11.4) |
| 2 | §4 | Alcance Fase 1: omnicanal, consulta, reportes, borradores, claims, cotizaciones, recordatorios | ✅ | M1–T10 | Todo el portal | — |
| 3 | §5 | 8 restricciones: no tocar CargoWise, no agendar por terceros, firma IA, solo monday escribe, aislamiento, aprobación humana, logs | ✅ 7/8 · ⚠️ 1 | T7.11, T8.9, T9.1 | /admin/auditoria | Acuse automático del correo entrante sin aprobación ⚖️ |
| 4 | §6.1–6.2 | Usuarios internos por área, rol y jerarquía; externos solo su empresa | ✅ | T6.20, T10.1 | /admin/equipo, /admin/perfiles | Lista real del personal ⚖️ |
| 5 | §6.3 | Canal WhatsApp | 🔑 · ⚠️ | T7.5, WaSender | WhatsApp | Envío real probado ✅; falta el secreto del webhook para recibir; internos por WhatsApp (T11.2) |
| 6 | §6.3 | Canal Teams | 🔑 | T3.2 | Teams | Azure Bot |
| 7 | §6.3 | Canal correo (entrada y salida) | 🔑 | T8.1, T10.3 | /correo | Permisos de Graph |
| 8 | §6.3 | Voz (audios) | 🔑 | T7.7 | Audio por WhatsApp | Azure OpenAI (STT) |
| 9 | §6.3 | SharePoint/OneDrive como fuente de documentos | ❌ | — | — | No existe; definir qué documentos ⚖️ |
| 10 | §7 | Arquitectura por capas | ✅ | ADR 0001–0007, T7.15 | — | Embeddings reales de la base de conocimiento 🔑 |
| 11 | §8 | Matriz de permisos editable | ✅ | S2.4, T10.1 | /admin/permisos | — |
| 12 | §9 | Asistente personalizado (nombre, avatar) | ✅ | T7.11 | /preferencias | Logo de Wideline ⚖️ |
| 13 | §9 | Firma «Artificial Intelligence Assistant on behalf of [Usuario]» | ✅ | T7.11, T9.1 | /asistente | — |
| 14 | §9 | Contactos corporativos y de WhatsApp autorizados | ✅ · 🔑 | T9.2 | /contactos | Permiso de Graph `User.ReadBasic.All` |
| 15 | §9 | Borradores, resúmenes, propuestas | ✅ · ⚠️ | T4, T6.13 | /correo | Borrador de follow-up automático (T11.1) |
| 16 | §10.1 | Búsqueda de correo | ⚠️ | T7.1 | /correo | Bug: por chat se pierde el remitente; extracto y enlace (T11.1) |
| 17 | §10.2 | Propuesta de respuesta con aprobación | 🔑 | T6.13 | /correo | Usar «pendientes» y elegir el tipo (T11.1) |
| 18 | §10.3 | Urgente / importante con reglas del usuario y feedback | ⚠️ | T7.2 | /correo → Reglas | Bug: el chat ignora las reglas; alerta inmediata; reglas por cc (T11.1) |
| 19 | §10.4 | Resumen diario programado | ⚠️ | T7.3 | /correo → Hora | Por correo; secciones de claims y críticas (T11.1, T11.3) |
| 20 | §10.5 | Alertas: respuesta de hilo, keyword, invitación | 🔑 · ⚠️ | T7.4 | /correo | «Avísame cuando responda» por chat (T11.3) |
| 21 | §10.6 | Calendario con confirmación y link de Teams | 🔑 | T7.13 | /calendario | `Calendars.ReadWrite` |
| 22 | §10.7 | Reglas de correo por lenguaje natural | 🔑 | T7.12 | /correo | Modificar y borrar reglas |
| 23 | §11 | Recordatorios: ETA, ausencia, seguimiento, canal, husos | ⚠️ | T6.6, T6.7, T6.16 | /recordatorios | Bug «si la tarea sigue abierta»; números en palabras; idioma (T11.3) |
| 24 | §12 | CargoWise solo lectura, capa cliente (L0) | ✅ | ADR 0006, T2.1, T10.2 | /embarques | — |
| 25 | §12 | Capas N1/N2/N3 diferenciadas | ⚠️ ⚖️ | S2.4, T10.1 | — | Hoy solo difieren en financieros; decisión de Wideline |
| 26 | §12.1 | Cliente consulta, filtra, exporta y ve gráficos | ✅ · ⚠️ | T6.9, T8.5 | /embarques | Gráficos en pantalla para el cliente (T11.5) |
| 27 | §13.1 | Estado de cuenta, deuda y pagos | 🔑 | T8.2 | /estado-de-cuenta | Dataset de Power BI y su mapeo ⚖️ |
| 28 | §13.2 | Organigrama | ✅ | T7.14 | /organigrama | Personal real (hoy vacío) ⚖️ |
| 29 | §13.3 | Itinerarios / INTTRA | 🔑 | T8.4, T8.8 | /itinerarios | API de INTTRA ⚖️ |
| 30 | §13.4 | Tarifas contractuales | ✅ | T8.3, T10.3 | /tarifas | Tarifas reales ⚖️ |
| 31 | §14 | Claims: recepción, ticket monday, alerta, acuse, evidencia | ✅ · 🔑 | T6.3, T6.4, T7.8, T8.1, T8.5 | /reclamos | Tipo «costo» y estatus «en revisión» (T11.4); token de monday |
| 32 | §15 | Cotizaciones: voz, faltantes, pricing, Wisor | ✅ · 🔑 | T7.9, T8.4 | /cotizaciones | API de Wisor ⚖️ |
| 33 | §16 | Reportes PDF/Excel, gráficos, comparativos, estadísticos | ✅ | T8.5 | /analitica, «Exportar comparativo» | — |
| 34 | §16 | Clientes inactivos (3 y 6 meses) | ✅ | T7.10 | Correo a comercial | — |
| 35 | §16 | Caída o subida abrupta de profit; alertas a management | ❌ · ⚖️ | — | — | Sin datos financieros; detección por cliente y alertas (T11.4) |
| 36 | §17 | Auditoría append-only (canal, instrucción, audio, respuesta, zona horaria) | ✅ | T1, T10.5 | /admin/auditoria | Campo «directo o con IA» y referencia a archivos (T11.4) |
| 37 | Anexo A | Lista de usuarios por departamento | ⚖️ | T6.20 (estructura) | /admin/equipo | Nombres reales de Wideline |
| 38 | Anexo B | Matriz de integraciones | ✅ · 🔑 | T6.19 | /admin/integraciones | Credenciales por plataforma |
| 39 | Anexo C | 8 casos de uso de punta a punta | ✅ 7/8 · ❌ 1 | T8.7 | /asistente | Caso 1: correo por audio de WhatsApp para un interno (T11.2) |
| 40 | Anexo D | Pendientes para desarrollo (APIs, arquitectura, auth, logs, costos) | ✅ · ⚖️ | ADR 0001–0007 | `docs/decisions/` | Costos reales con modelos reales 🔑 |

## Resumen

| Estado | Cantidad | De qué depende |
|---|---|---|
| ✅ Hecho | 18 | — |
| 🔑 Hecho, falta credencial | 12 | Wideline: Graph, Teams, monday, Azure OpenAI, WaSender (secreto del webhook) |
| ⚠️ Parcial | 13 | Nosotros: T11.1–T11.5 |
| ❌ No hecho | 3 | SharePoint ⚖️, profit ⚖️, caso 1 por voz (T11.2) |
| ⚖️ Depende de Wideline | 10 | Datos, decisiones y personal real |

Algunas filas combinan dos estados (por ejemplo «✅ · 🔑»), por eso la suma no coincide con 40.

## Próximos pasos

**Nuestros (T11, ver `tasks.md`):**
- **T11.1:** bugs de correo (búsqueda por chat, reglas en el chat), alerta de urgentes, follow-up automático y secciones del resumen.
- **T11.2:** WhatsApp para usuarios internos, que cierra el caso 1 del Anexo C.
- **T11.3:** recordatorios («si la tarea sigue abierta», números en palabras), correo como canal de avisos y avisos en el idioma del usuario.
- **T11.4:** auditoría «directo o con IA», KPIs por área, detección de incremento abrupto, alertas a management, export de claims y KPIs, tipo «costo» y estatus nuevos.
- **T11.5:** front de lo anterior y gráficos para clientes.

**De Wideline:**
- Permisos de Graph (correo, calendario, contactos).
- Secreto del webhook de WaSender.
- Tokens de monday y Azure OpenAI.
- Dataset de Power BI.
- API o canal por correo para INTTRA y Wisor.
- Tarifas reales y lista del personal (Anexo A).
- Logo de marca.
- Decisiones: capas N1/N2/N3 y acuse automático del correo.

## Notas del documento

- El FRD dice v5.1 en la portada y en el nombre, pero v5.0 en la tabla de datos y en el pie de página.
- El propio FRD se declara borrador: no incluye la matriz final de permisos, los casos de uso exhaustivos ni los criterios QA. Esos puntos los debe cerrar Wideline.
