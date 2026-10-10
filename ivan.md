# Horas de Ivan — proyecto WAI

Del 14-09-2026 al 10-10-2026 · fuentes: horas-consolidado.csv, git log y tasks.md

## Total

| Concepto | Horas |
|---|---|
| Trabajo medido (sesiones de Claude y commits) | 78.1 |
| Reuniones | 46.0 |
| **Total** | **124.1** |

- Días con trabajo: 22 · commits: 79 · entregas marcadas como hechas: 154
- Las reuniones son 2 h por cada día del periodo (`reunion-diaria` en el CSV): son un acuerdo, no una medición.

## Por semana

| Semana (lunes) | Trabajo | Reuniones | Total |
|---|---|---|---|
| 14-09 | 13.8 | 10.0 | 23.8 |
| 21-09 | 18.2 | 12.0 | 30.2 |
| 28-09 | 12.2 | 12.0 | 24.2 |
| 05-10 | 33.9 | 12.0 | 45.9 |

## Por día

| Fecha | Trabajo | Reunión | Commits | Lo principal que quedó hecho |
|---|---|---|---|---|
| 14-09 | 0.0 | 2.0 |  | ver pedidos del día en horas-detalle |
| 15-09 | 1.7 | 2.0 |  | ver pedidos del día en horas-detalle |
| 17-09 | 0.4 | 2.0 |  | ver pedidos del día en horas-detalle |
| 18-09 | 6.5 | 2.0 | 33 | T1.16 Docker: backend/Dockerfile multi-stage (uv) + compose.yaml (front/nginx, api, worker, scheduler, redis,… · T1.22 Infra: front/Dockerfile (node build → nginx con la SPA; Caddy hasta el 18-sep), backend/Dockerfile… · T2.5b (S2.5 stack v1, 18-sep) Enlace mágico por correo para externos: start_magic/verify_magic, un solo uso,… (+1 más) |
| 19-09 | 5.2 | 2.0 | 38 | voice: S3.2 audio y transcripcion al StorageProvider, audio_ref en auditoria; InMemoryStorage; media_key por… · mail: S4.3 tier chico de LLM tras las reglas (solo lo ambiguo, tope por bandeja, presupuesto, fail-soft);… · ops: S6.1 OpenTelemetry por configuracion (api/worker/scheduler), spans sin PII, colector OTLP en el VPS (+35 commits más) |
| 21-09 | 4.7 | 2.0 | 2 | T1.19 Monorepo wai_mas/: backend/ (proyecto Python; swarm done corre aquí), front/ (Angular 22), infra/,… |
| 22-09 | 5.1 | 2.0 |  | Portal: tablero de módulos, panel de administración y de usuario, menú lateral nuevo (22-sep) · Asistente: conversación que sobrevive a navegar/recargar (por usuario y empresa, se borra al cerrar la… · Seed de DEV ya no borra los embarques reales en cada despliegue (22-sep, test de regresión) (+9 más) |
| 23-09 | 0.0 | 0.0 |  | Suscripciones de Graph por usuario (conv.graph_subscription, migración 0019): el scheduler las crea/renueva… · Alta de contactos de clientes reales desde Administración → Clientes y contactos: nombre + correo y/o… · Hueco de seguridad cerrado: el primer login habilitaba al contacto en TODAS sus empresas sin intervención de… (+9 más) |
| 24-09 | 0.8 | 2.0 |  | Módulo de usuarios internos (Administración → Equipo interno, FRD §6.1-6.2, Anexo A, §13.2) — T6.20 · (hecho en T8.9) 🛠 Invitaciones de Equipo interno: expiración (30-60 días sin login → aviso a Dirección) y… |
| 25-09 | 3.8 | 2.0 | 1 | T7.1 Búsqueda de correo estructurada: remitente, destinatario, rango de fechas y referencia como propiedades… · T7.2 Reglas de prioridad por usuario (remitente, palabra clave, cliente) + retroalimentación «esto no era… · T7.3 Resumen diario de correo programado a la hora que elige el usuario, por su canal preferido — FRD §10.4 (+7 más) |
| 26-09 | 3.2 | 2.0 | 4 | T9.1 Cierre FRD §9 — firma «on behalf of» también en respuestas de ayuda (KB), analítica y confirmación de… · T9.2 Cierre FRD §9 — búsqueda de contactos por chat y portal (reemplaza T6.12): corporativos @wideline.biz… · T7.11 Asistente personalizado: nombre y avatar por usuario, firma «Artificial Intelligence Assistant on… (+17 más) |
| 27-09 | 0.7 | 2.0 | 1 | Código muerto en _PENDING_FEATURE (cerrado 27-sep, T10.5, ver issues.md) — memory/issues.md · T10.1 Perfiles editables por encima del nivel (reunión 26-ago: «roles editables, crear nuevos perfiles y… · T10.2 Nombres de puertos: completar mart/locodes.py para los ~1.600 embarques que muestran el código del… (+3 más) |
| 28-09 | 0.6 | 2.0 |  | ver pedidos del día en horas-detalle |
| 29-09 | 1.5 | 2.0 |  | ver pedidos del día en horas-detalle |
| 30-09 | 6.0 | 2.0 |  | alerta inmediata de urgentes (mail/urgent_alerts.py, scheduler; reglas del usuario, una vez por correo,… · T11.2 WhatsApp para usuarios internos (§6.3 «canal principal para clientes, usuarios internos», Anexo C caso… · «recuérdame si esta tarea sigue abierta mañana» → seguimiento de ticket (último ticket propio si no hay… (+3 más) |
| 01-10 | 1.8 | 2.0 |  | Mail.Send de aplicación CONCEDIDO y validado (202, correo real enviado); MAGIC_LINK_SENDER configurado al… · permisos de Graph validados en vivo — Files.Read.All y User.ReadBasic.All concedidos como APLICACIÓN y… · §12.1.4/§16.7 gráficos en pantalla para el cliente externo — módulo analytics abierto a L0 (FRD §12.1.3-4;… (+5 más) |
| 02-10 | 2.1 | 2.0 |  | ver pedidos del día en horas-detalle |
| 03-10 | 0.2 | 2.0 |  | base — @angular/material+cdk+animations 22, Material Symbols autoalojados, tema M3 mapeado a la paleta… · esqueleto — lateral con iconos Material Symbols (rellenos en el activo, estilo píldora M3), menú de usuario… · /correo con Material — búsqueda con campo outline e icono, periodos y vista original/texto con button-toggle,… (+5 más) |
| 04-10 | 0.0 | 0.0 |  | paginación y filtrado en TODAS las tablas — shared/paging.ts (paginador local reutilizable + matchesText sin… |
| 05-10 | 4.0 | 2.0 |  | auditoría con 12 agentes (7 UX/UI + 5 estructura; docs/auditoria-12-agentes-2026-10-05.md) y corrección del… · sofisticación visual de cara al cliente — tablero del cliente abre con 3 KPI de SU negocio (activos / con… · QA dimensional con 8 agentes de píxeles (pedido del usuario: «cuadros desproporcionados sin estándar de… (+1 más) |
| 06-10 | 6.4 | 2.0 |  | paleta naviera (adenda ADR 0009; pedido: «colores que no parezcan hechos por IA»): fuera el turquesa en todo… · identidad Wideline oficial (el usuario entregó el logo; adenda 2 del ADR 0009): colores extraídos del PNG… · solo colores de marca + blanco (pedido: «veo un color medio azul que no cuadra»): retirados TODOS los azules… (+8 más) |
| 07-10 | 4.8 | 2.0 |  | el chat del portal llama al asistente por su nombre (T7.11 / FRD §9; pedido: «el asistente deberá tener… · renombrar al asistente desde el propio chat (FRD §9 «nombre configurable por el propio usuario»; el usuario… · FRD §9 verificado punto por punto y cerrado (pedido: «revisa todos estos puntos, verifica si está realizado… (+4 más) |
| 08-10 | 0.2 | 2.0 |  | ver pedidos del día en horas-detalle |
| 09-10 | 12.4 | 2.0 |  | §9.8 — OBSOLETO (9-oct): User.ReadBasic.All ya estaba consentido desde el 1-oct; la búsqueda corporativa… · auditoría FRD §9 con 4 agentes (backend senior, frontend/UX, QA en vivo, seguridad; pedido: «analízalo con 4… · S-3 HIGH (9-oct) — CERRADO 9-oct (csrf_protect + test adversarial): POST /admin/whatsapp/send y /send-media… (+7 más) |
| 10-10 | 6.1 | 2.0 |  | Auditoría del Mapa de Módulos FRD §8 (4 agentes, solo lectura) → docs/auditoria-mapa-modulos-2026-10-10.md (+… · Ola A implementada 10-oct (A1 monday no bloqueante + reintento por reentrada; en monday NO existe tablero de… · Ola B implementada 10-oct: B1 lateral agrupado por §8 (front, module-presentation.ts), Centro de Reportes… (+29 más) |
| **Total** | **78.1** | **46.0** | **79** | **124.1 h** |
