# Cumplimiento del FRD v5.1, punto por punto

**Fecha:** 29 de septiembre de 2026
**Documento:** `WAI_Master_FRD_v5.1_Borrador_Integral.docx`. Se recorre en su mismo orden, cada viñeta y cada fila de tabla.
**Base:** validación del 27-sep con evidencia en el código, más lo construido después (T10, cierre del punto 9 y WhatsApp vía WaSender).
**Portal DEV:** https://srv1974730.hstgr.cloud · **Tareas:** `tasks.md` · **Resumen por módulo:** `docs/estado-frd-v5.1-2026-09-28.md`

| Símbolo | Significado |
|---|---|
| ✅ | Cumple / hecho |
| 🔑 | Hecho; falta la credencial del proveedor para encenderlo |
| ⚠️ | Parcial (se indica qué falta) |
| ❌ | No hecho |
| ⚖️ | Depende de un dato o de una decisión de Wideline |
| — | Informativo, no es un requisito construible |

---

## 1. Resumen ejecutivo

| # | Punto del FRD | Estado | Cómo se cumple / qué falta |
|---|---|---|---|
| 1.1 | No solo un chatbot: capa inteligente omnicanal sobre el ecosistema existente | ✅ | Monolito modular con canales WhatsApp, Teams, correo, portal y voz sobre CargoWise, M365, monday y Power BI (ADR 0001) |
| 1.2 | Interactuar con internos y externos por lenguaje natural, texto, voz y canales corporativos | ✅ · ⚠️ | Texto y voz por WhatsApp, Teams y portal. Falta que los **internos** usen WhatsApp (T11.2) |
| 1.3 | Fase 1 enfocada en experiencia de usuario, no en automatización profunda | ✅ | Nada automatiza decisiones; todo sugiere y pide aprobación |
| 1.4 | Consultar, generar borradores, resumir, alertar, recordar, organizar y generar reportes | ✅ | Embarques, correo, recordatorios, reportes PDF/Excel, analítica |
| 1.5 | Transparencia: las comunicaciones indican intervención de AI Assistant | ✅ | Firma obligatoria en toda salida (`common/signature.py`) |
| 1.6 | Seguridad, trazabilidad, logs, separación por cliente y usuario | ✅ | RLS por cliente y por dueño, auditoría append-only, permisos por nivel y perfil |
| 1.7 | No modificar sistemas core, salvo monday para tareas, tickets y estatus | ✅ | CargoWise solo lectura. Las escrituras a Graph (borradores, reglas, calendario) van con aprobación, como permite el Anexo B |
| 1.8 | Principio: sugiere, organiza, consulta, genera borradores y pide aprobación; nada sensible sin validación humana | ✅ · ⚠️ | Todo con botón de aprobación. Excepción: el acuse automático del correo entrante ⚖️ |
| 1.9 | La arquitectura final la propone el equipo de desarrollo | ✅ | ADR 0001 a 0007 en `docs/decisions/` |

## 2. Estructura documental recomendada

| # | Documento | Estado | Observación |
|---|---|---|---|
| 2.A | Executive Vision & Master Concept | — | Es el FRD actual (Wideline) |
| 2.B | FRD | — | Es el FRD actual (Wideline) |
| 2.C | Use Cases & Operational Flows | ⚖️ | Segunda etapa, a cargo de Wideline. Los 8 casos del Anexo C sí están implementados |
| 2.D | Technical Architecture Document | ✅ | ADR 0001-0007, `docs/architecture/` |
| 2.E | Roles & Permissions Matrix | ✅ · ⚖️ | Implementada y editable en el sistema (/admin/permisos, /admin/perfiles). Falta el documento formal de Wideline |
| 2.F | Integrations Matrix | ✅ | `docs/architecture/matriz-integraciones.md` y /admin/integraciones |
| 2.G | AI Governance & Compliance | ⚠️ | Cubierto en parte por ADR 0002 (modelos y presupuesto) y ADR 0007 (sensibilidad). Es un documento posterior |

## 3. Objetivos de negocio y KPIs

### 3.1 Objetivos cuantificables

| # | Objetivo | Estado | Observación |
|---|---|---|---|
| 3.1.1 | Reducir tiempos de respuesta 80-90 % | ⚠️ ⚖️ | Las respuestas del sistema tardan de 17 ms a 1,2 s (prueba de carga). Falta la línea base actual de Wideline para medir la reducción |
| 3.1.2 | Reducir carga administrativa 20-30 % | ⚖️ | Se mide en el piloto |
| 3.1.3 | Claims: mejor organización, trazabilidad y seguimiento | ✅ | Ticket en monday, prioridad, responsable, evidencia, seguimiento |
| 3.1.4 | Customer Experience: mejora radical | ⚖️ | Se mide en el piloto |
| 3.1.5 | Correos sin respuesta: reducción mayor al 50 % | ⚠️ | Hay priorización, resumen y alertas. Falta el follow-up automático de pendientes (T11.1) |

### 3.2 KPIs iniciales (/kpis)

| # | KPI | Estado | Qué falta |
|---|---|---|---|
| 3.2.1 | Adopción: activos, frecuencia, recurrencia, **por área** | ⚠️ | Corta por nivel y organización, no por área (T11.4) |
| 3.2.2 | Tickets gestionados | ✅ · ⚠️ | Reclamos y cotizaciones sí; los seguimientos no se cuentan (T11.4) |
| 3.2.3 | Correos resumidos, procesados y con respuesta sugerida | ⚠️ | Faltan el resumen programado y el correo entrante (T11.4) |
| 3.2.4 | Recordatorios automáticos | ⚠️ | Faltan los avisos de estatus y los resúmenes (T11.4) |
| 3.2.5 | Solicitudes no resueltas por WAI | ✅ | — |
| 3.2.6 | Funcionalidades más utilizadas | ✅ | — |

## 4. Alcance de Fase 1

| # | Punto | Estado | Observación |
|---|---|---|---|
| 4.1 | Orientada a experiencia de cliente y usuario interno | ✅ | — |
| 4.2 | Comunicación omnicanal: email, WhatsApp Business, Teams, voz y plataformas autorizadas | ✅ · 🔑 | Portal ✅ y WhatsApp real vía WaSender ✅ (envío probado). Email, Teams y voz construidos; esperan credencial 🔑 |
| 4.3 | Consulta y organización de información existente | ✅ | 3.050 embarques reales de CargoWise |
| 4.4 | Reportes PDF y Excel | ✅ | Embarques, comparativo de periodo, estado de cuenta |
| 4.5 | Borradores, propuestas de respuesta, summaries y alertas | ✅ · ⚠️ | /correo. Falta la alerta inmediata de correos urgentes (T11.1) |
| 4.6 | Claims externos únicamente | ✅ | — |
| 4.7 | Recepción y estructuración de cotizaciones | ✅ | /cotizaciones |
| 4.8 | Recordatorios inteligentes y workflows ligeros | ✅ · ⚠️ | /recordatorios. Falta el caso «si la tarea sigue abierta mañana» (T11.3) |
| 4.9 | Sin automatización profunda de decisiones críticas | ✅ | — |
| 4.10 | Sin modificación automática de sistemas core | ✅ | — |

### Regla de complejidad de Fase 1 (subtítulo dentro del §4)

| # | Punto | Estado | Observación |
|---|---|---|---|
| 4.R1 | Informar si una funcionalidad es muy compleja o implica tiempos extra | ✅ | Informado: SharePoint, Power BI (mapeo), INTTRA, Wisor y profit |
| 4.R2 | Wideline decide si queda en Fase 1 o pasa a Fase 2/3 | ⚖️ | Pendiente de decisión de Wideline |

## 5. Principios y restricciones críticas

| # | Principio | Estado | Evidencia |
|---|---|---|---|
| 5.1 | No modificar CargoWise | ✅ | Export de solo lectura → data mart; la app solo tiene SELECT |
| 5.2 | No comprometer agendas de terceros | ✅ | El calendario solo actúa sobre la agenda propia y con confirmación |
| 5.3 | No suplantar identidad humana | ✅ | Firma «AI Assistant on behalf of…» obligatoria |
| 5.4 | No modificar sistemas críticos automáticamente | ✅ | — |
| 5.5 | Excepción limitada en monday (tareas, tickets, estatus) | ✅ · 🔑 | Cliente GraphQL real; falta el token |
| 5.6 | No exponer información entre clientes | ✅ | RLS fail-closed y suite adversarial |
| 5.7 | Sin decisiones críticas autónomas | ✅ · ⚠️ | Excepción del acuse automático del correo entrante ⚖️ |
| 5.8 | Logs obligatorios de consulta, instrucción, output y acción sugerida | ✅ | `audit.event` append-only |

## 6. Tipos de usuarios y canales

### 6.1 Tipos de usuarios

| # | Tipo | Estado | Observación |
|---|---|---|---|
| 6.1.1 | Externo: solo su empresa o scope | ✅ | Habilitación comercial por empresa (/admin/clientes) |
| 6.1.2 | Interno: segmentado por departamento, rol y jerarquía | ✅ | Área, cargo, jefe, nivel y perfil (/admin/equipo, /admin/perfiles) |

### 6.2 Segmentación interna

| # | Punto | Estado | Observación |
|---|---|---|---|
| 6.2.1 | Basada en PBX y departamentos: Dirección, CS, Operaciones, Pricing, Comercial, Finanzas, Claims, Administración/RH, IT, futuros | ✅ | Los 9 departamentos más «Otros» |
| 6.2.2 | Lista nominal como anexo | ⚖️ | Faltan los nombres reales |

### 6.3 Canales e integraciones

| # | Canal | Uso esperado | Estado | Qué falta |
|---|---|---|---|---|
| 6.3.1 | Microsoft 365 / Outlook | Correos, búsqueda, resumen, calendar, contactos, reglas, borradores | 🔑 · ⚠️ | Permisos de Graph. Bug en la búsqueda por chat (T11.1) |
| 6.3.2 | Teams | Mensajes, alertas, reminders, consultas | 🔑 | Azure Bot |
| 6.3.3 | WhatsApp Business | Principal para clientes e internos, voz, alertas, seguimiento | 🔑 · ⚠️ | Clientes ✅ vía WaSender (falta el secreto del webhook). **Internos ❌** (T11.2) |
| 6.3.4 | Correo electrónico | Comunicación formal, borradores, seguimiento | 🔑 | Graph |
| 6.3.5 | Mensajes de voz | Instrucciones, solicitudes, búsquedas, cotizaciones, reminders | 🔑 | Azure OpenAI (STT); hoy solo entran por WhatsApp |
| 6.3.6 | monday.com | Tickets, claims, tareas, estatus, workflows | 🔑 | Token e ids de columnas |
| 6.3.7 | CargoWise | Embarques, eventos, milestones | ✅ | Actualización horaria desde OneDrive: falta corregir la contraseña del archivo de credenciales |
| 6.3.8 | Power BI | Estados de cuenta, deuda, pagos, reportes financieros | ⚠️ · ❌ ⚖️ | Funciona en fake. Falta el dataset y su mapeo |
| 6.3.9 | INTTRA | Itinerarios y servicios | ⚠️ ⚖️ | Funciona en fake. Falta la API |
| 6.3.10 | Wisor | Solicitudes de cotización | ⚠️ ⚖️ | Funciona en fake. Falta la API |
| 6.3.11 | SharePoint / OneDrive | Documentos, reportes, anexos, archivos | ❌ ⚖️ | No existe como fuente de documentos; definir qué documentos |

## 7. Arquitectura conceptual

| # | Capa | Estado | Observación |
|---|---|---|---|
| 7.1 | Usuarios (internos y externos) | ✅ | — |
| 7.2 | Canales | ✅ · 🔑 | — |
| 7.3 | Orchestration Layer (NLP, voz a texto, RAG, reglas, logs) | ✅ · 🔑 | Los embeddings reales para RAG esperan Azure |
| 7.4 | Control y gobernanza (permisos, logs, trazabilidad, AI disclosure) | ✅ | — |
| 7.5 | Sistemas fuente | ⚠️ | Ver 6.3 |
| 7.6 | Outputs (respuestas, reportes, borradores, alertas, PDF, Excel) | ✅ | — |

## 8. Mapa de módulos funcionales

| # | Módulo | Estado | Pantalla |
|---|---|---|---|
| 8.1 | AI Executive Assistants | ✅ | /asistente, /preferencias |
| 8.2 | Microsoft 365 Intelligence Layer | 🔑 · ⚠️ | /correo, /calendario, /contactos |
| 8.3 | CargoWise Intelligence Layer (solo lectura) | ✅ | /embarques |
| 8.4 | Claims externos | ✅ · 🔑 | /reclamos |
| 8.5 | Cotizaciones inteligentes | ✅ · 🔑 | /cotizaciones |
| 8.6 | Reporting & Analytics | ✅ · ⚠️ | /analitica, /kpis |
| 8.7 | Recordatorios y workflow engine | ✅ · ⚠️ | /recordatorios |
| 8.8 | Permissions Matrix (documento posterior) | ✅ | /admin/permisos, /admin/perfiles |
| 8.9 | Integrations Matrix (documento posterior) | ✅ | /admin/integraciones |

## 9. AI Executive Assistants

| # | Punto | Estado | Observación |
|---|---|---|---|
| 9.1 | Asistente AI personalizado para cada interno autorizado | ✅ | — |
| 9.2 | Nombre configurable por el propio usuario | ✅ | /preferencias |
| 9.3 | Avatar configurable | ✅ | 6 avatares de marca |
| 9.4 | Branding corporativo de Wideline | ✅ · ⚖️ | Paleta y firma. Falta el logo oficial |
| 9.5 | Usar la cuenta corporativa real del usuario | 🔑 | Graph delegado |
| 9.6 | Firma «Artificial Intelligence Assistant on behalf of [Usuario]» | ✅ | En todas las respuestas a internos |
| 9.7 | No enviar correos como humanos sin referencia AI | ✅ | — |
| 9.8 | Acceder a contactos corporativos @wideline.biz | ✅ · 🔑 | /contactos; Graph `User.ReadBasic.All` |
| 9.9 | Acceder a contactos autorizados de WhatsApp | ✅ | /contactos |
| 9.10 | Generar borradores, follow-ups, resúmenes, reminders y propuestas | ✅ · ⚠️ | Falta el follow-up automático (T11.1) |
| 9.11 | Operar con permisos y logs | ✅ | — |
| 9.T1 | Tabla: Borradores, permitido con aprobación | ✅ | — |
| 9.T2 | Tabla: Firma AI obligatoria | ✅ | — |
| 9.T3 | Tabla: Enviar como humano, no permitido | ✅ | — |
| 9.T4 | Tabla: Contactos, según permisos | ✅ | Módulo `contacts` en la matriz |
| 9.T5 | Tabla: Correos, borrador y aprobación | ✅ · ⚠️ | Excepción del acuse automático ⚖️ |

## 10. Microsoft 365 Intelligence Layer

### 10.1 Búsqueda inteligente de correos

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 10.1.1 | Búsqueda desde cualquier canal autorizado | ✅ · 🔑 | — |
| 10.1.2 | Por voz o texto | 🔑 | STT real |
| 10.1.3 | Por palabras clave, título, texto, shipment, remitente, destinatario, fechas o referencias | ⚠️ | En el portal sí. **Bug:** por chat se pierde el remitente (T11.1) |
| 10.1.4 | Devolver por el mismo canal como resumen, screenshot, anexo, extracto o enlace | ⚠️ | Hoy solo lista. Faltan extracto y enlace (T11.1) |
| 10.1.5 | Ejemplo: audio por WhatsApp «Búscame el correo de Juan Pérez…» | ❌ | Requiere internos por WhatsApp (T11.2) y el arreglo de la búsqueda (T11.1) |

### 10.2 Propuestas de respuesta

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 10.2.1 | Propuesta contextual al recuperar un correo | 🔑 | — |
| 10.2.2 | Considerar hilo, contexto, tono, shipment, pendientes y datos | ✅ · ⚠️ | Faltan los «pendientes» (T11.1) |
| 10.2.3 | Crear borrador y pedir aprobación antes de enviar | ✅ | — |
| 10.2.4 | Tipos: confirmación, solicitud de datos, respuesta operativa, seguimiento, estatus, coordinación de llamada | ⚠️ | La propuesta es genérica; no se elige el tipo (T11.1) |

### 10.3 Priorización: urgente vs importante

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 10.3.1 | Distinguir urgente de importante | ✅ | — |
| 10.3.2 | Definición de importante (estratégico) | ✅ | — |
| 10.3.3 | Definición de urgente (riesgo, vencimiento, cliente molesto, claim, demora, escalación) | ✅ | — |
| 10.3.4 | Enseñar reglas por remitente, palabras clave, cuerpo y cliente | ✅ | /correo → Reglas de prioridad |
| 10.3.5 | Enseñar reglas por personas en copia, título, shipment y patrones históricos | ❌ | T11.1 |
| 10.3.6 | Evolucionar por retroalimentación del usuario | ⚠️ | Solo por remitente |
| 10.3.7 | (hallazgo) El chat aplica las reglas del usuario | ❌ | **Bug:** el chat las ignora (T11.1) |
| 10.3.T1 | Tabla, Urgente: alerta inmediata y resumen | ⚠️ | El resumen ✅; la alerta inmediata ❌ (T11.1) |
| 10.3.T2 | Tabla, Importante: resumen destacado y seguimiento | ⚠️ | El resumen ✅; el seguimiento ❌ |
| 10.3.T3 | Tabla, Informativo: incluir en el resumen | ✅ | — |
| 10.3.T4 | Tabla, Pendiente: reminder y follow-up | ❌ | T11.1 |

### 10.4 Resúmenes diarios de correo

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 10.4.1 | Resumir por urgentes, importantes y pendientes | ✅ | — |
| 10.4.2 | Resumir claims y conversaciones críticas | ❌ | T11.1 |
| 10.4.3 | Basado en reglas personalizadas del usuario | ⚠️ | En el portal y el programado sí; en el chat no |
| 10.4.4 | Bajo demanda o en horario configurado | ✅ | /correo → Hora del resumen |
| 10.4.5 | Canales WhatsApp, Teams, dashboard y canal principal | ✅ · 🔑 | — |
| 10.4.6 | Canal correo | ❌ | T11.3 |

### 10.5 Alertas inteligentes de comunicación

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 10.5.1 | «Avísame cuando este cliente responda» | ⚠️ | En el portal sí; por chat no (T11.3) |
| 10.5.2 | Monitorear respuestas a hilos específicos | 🔑 | — |
| 10.5.3 | Alertar cuando una empresa envíe una invitación | 🔑 | — |
| 10.5.4 | Alertar por palabra clave o cliente | ✅ · 🔑 | — |
| 10.5.5 | Alertar por shipment | ⚠️ | Por chat no (T11.3) |
| 10.5.6 | Canales WhatsApp, Teams y email | ⚠️ | Falta el email (T11.3) |
| 10.5.7 | Llamadas automatizadas | — | El FRD las deja para fases futuras |

### 10.6 Calendar e invitaciones

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 10.6.1 | Crear invitaciones en nombre del usuario autorizado | 🔑 | `Calendars.ReadWrite` |
| 10.6.2 | Aceptar o rechazar invitaciones, si hay autorización | 🔑 | — |
| 10.6.3 | No comprometer agendas de terceros | ✅ | — |
| 10.6.4 | Flujo propuesta → invitación → aceptación → confirmación | ⚠️ | No se avisa al organizador cuando responden |
| 10.6.5 | Generar links de Teams | 🔑 | — |

### 10.7 Reglas de correo por lenguaje natural

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 10.7.1 | Crear reglas por texto o voz | 🔑 | `MailboxSettings.ReadWrite` |
| 10.7.2 | Interpretar, crear la carpeta si no existe, preparar la regla y confirmar | 🔑 | — |
| 10.7.3 | Registrar creación y modificación | ✅ · ⚠️ | La creación sí; modificar y borrar no existen |

## 11. Recordatorios inteligentes y workflow engine

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 11.1 | Recordatorios por el canal principal que elija el usuario | ⚠️ | WhatsApp, Teams y portal sí; correo no (T11.3) |
| 11.2 | Proactivos: sin ir a una sección | ✅ | Push por WhatsApp/Teams y aviso en el portal |
| 11.3 | Reconocer husos horarios del lugar relevante | ⚠️ | Hoy el del usuario, no el del puerto (T11.3) |
| 11.4 | Reconocer idiomas y adaptar la comunicación | ⚠️ | El portal y el chat sí; los avisos proactivos siguen en español (T11.3) |
| 11.5 | Seguimiento sobre el seguimiento | ✅ · 🔑 | — |
| 11.6 | Consultar tareas en monday para saber su estado | ✅ · 🔑 | — |
| 11.7 | Reglas por eventos y por ausencia de eventos | ✅ | — |
| 11.T1 | Ejemplo «Avísame dos días antes del ETA» | ✅ · ⚠️ | Con «2» sí; con «dos» en palabras no (T11.3) |
| 11.T2 | Ejemplo «Si ocurre A y no B en 48 horas» | ✅ | — |
| 11.T3 | Ejemplo «Recuérdame si esta tarea sigue abierta mañana» | ❌ | **Bug:** crea un aviso de hora sin consultar el estado (T11.3) |
| 11.T4 | Ejemplo «Cuando el cliente responda este correo» | ⚠️ | En el portal sí; por chat no (T11.3) |

## 12. CargoWise Intelligence Layer

| # | Punto | Estado | Observación |
|---|---|---|---|
| 12.1 | Las acciones de usuarios no modifican CargoWise | ✅ | — |
| 12.2 | Externos sin acceso a datos sensibles | ✅ | — |
| 12.3 | Ciertos internos tampoco acceden a todo | ⚠️ ⚖️ | Los perfiles recortan módulos; las filas de embarques son iguales para L1, L2 y L3 |
| 12.4 | No chatear con la base productiva | ✅ | — |
| 12.5 | Base de consulta, data mart o capa segregada | ✅ | ADR 0006 |
| 12.6 | Bases o vistas para clientes, N1, N2 y N3 | ⚠️ ⚖️ | Cliente ✅; N1, N2 y N3 solo difieren en financieros |
| 12.7 | El equipo puede proponer alternativas que mitiguen riesgos | ✅ | Export cifrado → mart con RLS |
| 12.T1 | Capa Clientes: shipments, eventos, milestones y reportes de su empresa | ✅ | — |
| 12.T2 | Capa Interno N1: datos operativos | ✅ | — |
| 12.T3 | Capa Interno N2: más visibilidad, reportes y comparativos | ⚠️ | Mismas filas que N1 más financieros ⚖️ |
| 12.T4 | Capa Interno N3: gerencial, analytics, profit y tendencias | ⚠️ · ❌ | Falta el profit (sin datos) |

### 12.1 Usuarios externos sobre CargoWise

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 12.1.1 | Consultar por UI lo de su capa | ✅ | /embarques |
| 12.1.2 | Shipments, eventos, milestones, estatus y reportes | ✅ | — |
| 12.1.3 | Reportes, tablas, comparativos y tendencias simples | ✅ | «Exportar comparativo» (PDF/Excel) |
| 12.1.4 | Gráficos en pantalla | ⚠️ | Solo en el PDF (T11.5) |
| 12.1.5 | Solo consultar, analizar, exportar y visualizar dentro de su capa | ✅ | — |
| 12.1.6 | Nada fuera de la capa disponible | ✅ | Suite adversarial |

## 13. Otros módulos de consulta y datos

### 13.1 Estado de cuenta y deuda

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 13.1.1 | Entregar deuda, estado de cuenta y últimos pagos | ✅ (fake) | /estado-de-cuenta |
| 13.1.2 | Fuente Power BI y sistemas autorizados | ❌ ⚖️ | Dataset y mapeo |
| 13.1.3 | Externos solo la propia | ✅ | — |
| 13.1.4 | Internos según permisos | ✅ | — |
| 13.1.5 | Outputs: resumen, PDF, Excel, dashboard | ✅ | — |

### 13.2 Organigrama

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 13.2.1 | Acceso al organigrama de Wideline | ✅ · ⚖️ | Personal real (hoy vacío en DEV) |
| 13.2.2 | Consultar responsables, áreas y jerarquías | ✅ | /organigrama y chat |
| 13.2.3 | Externos solo lo permitido | ✅ | — |

### 13.3 Itinerarios e INTTRA

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 13.3.1 | Acceder a las plataformas de itinerarios | 🔑 (fake) | /itinerarios |
| 13.3.2 | INTTRA como plataforma relevante | ❌ ⚖️ | API o canal por correo |
| 13.3.3 | Entregar naviera, servicio, ETD, ETA, tránsito, transbordos, origen/destino y observaciones | ✅ | — |
| 13.3.4 | Sujeto a confirmación | ✅ | — |

### 13.4 Tarifas contractuales

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 13.4.1 | Leer tarifas desde Excel | ✅ | /tarifas → importar |
| 13.4.2 | Leer tarifas desde bases de datos y plataformas | ❌ ⚖️ | Definir la fuente |
| 13.4.3 | Solo internos con nivel suficiente | ✅ | L2+ |
| 13.4.4 | Vigencias, proveedor, origen, destino, moneda, condiciones, incluidos y excluidos | ✅ | — |
| 13.4.5 | Comparar y señalar información incompleta o vencida | ✅ | Marcas Vigente, Vencida e Incompleta, con filtro |

## 14. Claims externos

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 14.1 | Solo claims externos (clientes → Wideline) | ✅ | — |
| 14.2 | Claims internos fuera de Fase 1 | ✅ | — |
| 14.3 | Recibir por correo, WhatsApp y otros canales | ✅ · 🔑 | WhatsApp y portal ✅; correo 🔑 |
| 14.4 | Identificar que es un claim | ✅ | — |
| 14.5 | Clasificar, organizar y extraer datos | ✅ | — |
| 14.6 | Crear ticket en monday | 🔑 | Token |
| 14.7 | Alertar a los responsables | ✅ | — |
| 14.8 | Confirmar la recepción al cliente | ✅ | — |
| 14.9 | Seguimiento básico y trazabilidad | ✅ | — |
| 14.T1 | Cliente / contacto | ✅ | — |
| 14.T2 | Canal de recepción | ✅ | — |
| 14.T3 | Descripción | ✅ | — |
| 14.T4 | Shipment relacionado | ✅ | — |
| 14.T5 | Tipo (operativo, documental, costo, demora, daño…) | ⚠️ | Falta «costo» (T11.4) |
| 14.T6 | Prioridad preliminar | ✅ | — |
| 14.T7 | Responsable interno | ✅ | — |
| 14.T8 | Evidencia y adjuntos | ✅ | — |
| 14.T9 | Estatus (inicial, en revisión, pendiente, cerrado) | ⚠️ | Faltan «en revisión» y «pendiente» (T11.4) |

## 15. Solicitudes inteligentes de cotización

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 15.1 | Solicitar por texto, email, WhatsApp, formularios o voz | ✅ · 🔑 | — |
| 15.2 | Convertir audio a texto | 🔑 | STT real |
| 15.3 | Interpretar la intención de cotizar | ✅ | — |
| 15.4 | Identificar datos disponibles y faltantes | ✅ | — |
| 15.5 | Pedir solo lo que falta | ✅ | — |
| 15.6 | Estructurar el requerimiento para comercial/pricing | ✅ | — |
| 15.7 | Canalizar al equipo o a Wisor | ✅ · 🔑 ⚖️ | Al equipo ✅; Wisor espera la API |
| 15.8 | Generación automática completa de la cotización | — | Fase posterior, según el propio FRD |
| 15.T1 | Origen / destino | ✅ | — |
| 15.T2 | Tipo de servicio (marítimo, aéreo, terrestre, proyecto) | ✅ | — |
| 15.T3 | Carga (descripción, peligrosa, dimensiones, peso, volumen) | ✅ | — |
| 15.T4 | Contenedor (20, 40, HQ, flat rack, open top) | ✅ | — |
| 15.T5 | Incoterm | ✅ | — |
| 15.T6 | Fecha (carga, urgencia, ventana) | ✅ | — |
| 15.T7 | Documentos | ✅ | — |
| 15.T8 | Requerimientos especiales | ✅ | — |

## 16. Reporting & Analytics

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 16.1 | Reportes en PDF y Excel | ✅ | — |
| 16.2 | Reportes operativos | ✅ | — |
| 16.3 | Reportes financieros | ⚠️ | Solo el estado de cuenta |
| 16.4 | Reportes comerciales, de claims y de KPIs | ❌ | Export (T11.4) |
| 16.5 | Reportes personalizados por cliente | ⚠️ | Por filtros |
| 16.6 | Tablas, comparativos y resúmenes ejecutivos | ✅ | — |
| 16.7 | Gráficos y dashboards | ✅ · ⚠️ | En el PDF y en pantallas internas; faltan en pantalla para clientes (T11.5) |
| 16.8 | Tablas dinámicas | ❌ | — |
| 16.9 | Analizar el comportamiento histórico de clientes | ✅ | — |
| 16.10 | Clientes que dejaron de dar carga en 3 o 6 meses | ✅ | Aviso a comercial |
| 16.11 | Clientes que bajan o suben profit abruptamente | ❌ ⚖️ | Sin datos financieros |
| 16.12 | Alertas a management | ⚠️ | La de inactividad va a comercial (T11.4) |
| 16.T1 | Cliente inactivo | ✅ | — |
| 16.T2 | Caída de profit | ❌ ⚖️ | — |
| 16.T3 | Incremento abrupto | ❌ | T11.4 |
| 16.T4 | Comparativo de periodos (3, 6, 12 meses, acumulado) | ✅ | — |
| 16.T5 | Tendencias (promedio, mediana, moda, máx, mín, proyección) | ✅ | — |

## 17. Logs y auditoría

| # | Punto | Estado | Qué falta |
|---|---|---|---|
| 17.1 | Todo lo generado con AI tiene log | ✅ | — |
| 17.2 | Distinguir cuándo el usuario actuó directo y cuándo con AI | ❌ | T11.4 |
| 17.3 | Qué hizo, dictó, recibió y consultó | ✅ | — |
| 17.4 | Fecha y hora con huso horario | ✅ | — |
| 17.5 | Canal, usuario, instrucción, audio, transcripción, respuesta y resultado | ✅ | — |
| 17.6 | Archivos entregados | ⚠️ | Falta la referencia al archivo (T11.4) |
| 17.7 | Acciones sugeridas | ⚠️ | Registradas dentro de las herramientas |
| 17.8 | Objetivo: auditoría, revisión, trazabilidad, seguridad y mejora | ✅ | /admin/auditoria |

## 18. Estado actual y siguientes pasos

| # | Punto | Estado | Observación |
|---|---|---|---|
| 18.1 | Versión v5.1, borrador de trabajo, no especificación final | — | El documento dice v5.0 en la tabla y en el pie |
| 18.2 | Revisar, eliminar secciones innecesarias, priorizar módulos y preparar anexos | ⚖️ | A cargo de Wideline |

## Anexos

### Anexo A — Lista de usuarios por departamento

| # | Punto | Estado | Observación |
|---|---|---|---|
| A.1 | Estructura por los 9 departamentos | ✅ | /admin/equipo |
| A.2 | Usuarios nominales | ⚖️ | Pendiente en el propio FRD |

### Anexo B — Matriz de integraciones

| # | Plataforma | Lectura | Escritura | Estado |
|---|---|---|---|---|
| B.1 | Microsoft 365 | 🔑 | 🔑 borradores, reglas y calendario con aprobación | 🔑 |
| B.2 | WhatsApp Business | ✅ WaSender | ✅ WaSender (envío probado) | 🔑 falta el secreto del webhook |
| B.3 | CargoWise | ✅ | No en Fase 1 ✅ | ✅ |
| B.4 | monday.com | 🔑 | 🔑 limitada | 🔑 |
| B.5 | Power BI | ❌ ⚖️ | No ✅ | ❌ ⚖️ |
| B.6 | Wisor | 🔑 fake | 🔑 fake | ⚖️ |
| B.7 | INTTRA | 🔑 fake | — | ⚖️ |
| B.8 | SharePoint / OneDrive | ❌ | — | ❌ ⚖️ |

### Anexo C — Casos de uso representativos

| # | Caso | Estado | Qué falta |
|---|---|---|---|
| C.1 | Buscar correo por audio de WhatsApp | ❌ | Internos por WhatsApp (T11.2) y bug de búsqueda (T11.1) |
| C.2 | Resumen diario de urgentes e importantes | ✅ | — |
| C.3 | Crear regla de correo con aprobación | ✅ · 🔑 | — |
| C.4 | Claim externo por WhatsApp → monday → confirmación | ✅ · 🔑 | — |
| C.5 | Cotización por voz | ✅ · 🔑 | — |
| C.6 | «Avísame dos días antes del ETA» | ✅ | Con «2»; con «dos» en palabras falta (T11.3) |
| C.7 | Estado de cuenta | ✅ (fake) | Power BI ⚖️ |
| C.8 | Itinerario | ✅ (fake) | INTTRA ⚖️ |

### Anexo D — Pendientes para desarrollo

| # | Punto | Estado | Evidencia |
|---|---|---|---|
| D.1 | Validar APIs (M365, WhatsApp, CargoWise, Wisor, monday, Power BI, INTTRA) | ✅ · ⚖️ | Matriz de integraciones; WhatsApp validado con WaSender; Wisor, INTTRA y Power BI esperan a Wideline |
| D.2 | Arquitectura de consulta segura para CargoWise | ✅ | ADR 0006 |
| D.3 | Modelo de autenticación y permisos | ✅ | ADR 0001, 0004, matriz y perfiles |
| D.4 | Infraestructura de logs y auditoría | ✅ | Auditoría append-only |
| D.5 | Almacenamiento de transcripciones y conversaciones | ✅ | Auditoría y storage |
| D.6 | Costos, tiempos y complejidad por módulo | ⚠️ | Presupuesto (ADR 0002) y prueba de carga (~200 usuarios con 1 proceso, ~500 con 3 réplicas). Falta el costo real con modelos reales 🔑 |

---

## Resumen de cumplimiento

Se cuentan los puntos construibles. Los informativos (—) no entran.

| Estado | Puntos | % |
|---|---|---|
| ✅ Cumple (incluye «✅ · 🔑», que ya funciona y solo espera credencial) | 155 | 60 % |
| 🔑 Solo falta la credencial | 20 | 8 % |
| ⚠️ Parcial | 53 | 20 % |
| ❌ No hecho | 22 | 8 % |
| ⚖️ Depende solo de Wideline | 9 | 3 % |
| **Total construible** | **259** | 100 % |

Cada fila se cuenta una vez, por su estado más bajo (si tiene ❌ cuenta como ❌; si tiene ⚠️, como parcial).

- **De los ❌ y ⚠️, dependen de nosotros** (T11.1 a T11.5): los bugs de búsqueda de correo, reglas en el chat y «tarea sigue abierta»; WhatsApp para internos; alertas de urgentes y follow-up; correo como canal de avisos; auditoría «directo o con IA»; KPIs por área; reportes de claims y KPIs; incremento abrupto; tipo «costo» y estatus; gráficos para clientes.
- **De los ❌ y ⚠️, dependen de Wideline:** Power BI, INTTRA, Wisor, SharePoint, profit, fuente de tarifas, personal real, capas N1/N2/N3 y el acuse automático.
