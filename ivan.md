# Horas de Ivan — proyecto WAI

Del 14-09-2026 al 10-10-2026 · generado el 11-10-2026

Horas de desarrollo medidas en las sesiones de Claude Code y en los commits, con la obra de cada día al lado: archivos distintos editados, veces que se lanzó cada suite de pruebas y despliegues. Las reuniones no se miden, son un acuerdo.

| Día | Horas | Archivos | Pruebas | Despliegues |
|---|---|---|---|---|
| 15-09 | 0.8 | 4 | 0 | 0 |
| 17-09 | 0.4 | 1 | 0 | 0 |
| 18-09 | 6.5 | — | — | — |
| 19-09 | 5.2 | — | — | — |
| 21-09 | 4.1 | 98 | 137 | 27 |
| 22-09 | 1.4 | 24 | 27 | 19 |
| 24-09 | 0.8 | 9 | 22 | 4 |
| 25-09 | 3.8 | 276 | 345 | 3 |
| 26-09 | 3.1 | 124 | 81 | 2 |
| 27-09 | 0.7 | 3 | 1 | 1 |
| 28-09 | 0.6 | 1 | 0 | 0 |
| 29-09 | 1.5 | 4 | 7 | 6 |
| 30-09 | 6.1 | 11 | 73 | 19 |
| 01-10 | 1.7 | 7 | 31 | 6 |
| 02-10 | 2.1 | 3 | 25 | 7 |
| 03-10 | 0.2 | 1 | 1 | 1 |
| 05-10 | 4.0 | 67 | 54 | 11 |
| 06-10 | 6.4 | 34 | 95 | 23 |
| 07-10 | 4.8 | 12 | 210 | 6 |
| 08-10 | 0.2 | 0 | 0 | 0 |
| 10-10 | 1.3 | 0 | 9 | 4 |
| **Total** | **55.7** | | | |

Los días con guiones no tienen sesión: sus horas salen de los commits.

Con **21.0 h de reuniones**, el total de Ivan es **76.7 h**.

**15-09-2026 · 0.8 h**
- Cambios: 4 archivos
  - otros (4): ARQUITECTURA.md, PLAN_DE_TRABAJO.md, diagram.py, diagram2.py

**17-09-2026 · 0.4 h**
- Cambios: 1 archivos
  - otros (1): test-ms-devicecode.sh

**21-09-2026 · 4.1 h**
- Cambios: 98 archivos
  - front/pantallas (31): admin-shell.html, admin-shell.scss, admin-shell.spec.ts, admin-shell.ts, audit.spec.ts, channel.html, channel.scss, channel.spec.ts, channel.ts, claims.html …
  - front/común (14): api.service.ts, app.routes.ts, demo-session.ts, guards.ts, module-presentation.spec.ts, module-presentation.ts, modules.store.spec.ts, modules.store.ts, schema.d.ts, shell.html …
  - otros (13): .env.example, MEMORY.md, e2e-label-vs-code-selector.md, fake_live_port_pattern.md, frontend-guards-vs-editable-rbac.md, opt-wai-docs-cargowise-age-operational-snug-rain.md, pii-in-storage-keys.md, probe.py, probe2.py, probe3.py …
  - front/otros (8): a11y.spec.ts, dev-ip.conf, portal.spec.ts, styles.scss, zz-clic.spec.ts, zz-diag.spec.ts, zz-publico.spec.ts, zz-safari.spec.ts
  - tests backend (8): helpers.py, test_api_modules.py, test_check.py, test_claims.py, test_kpis.py, test_mail_followup.py, test_monday_client.py, test_pipeline.py
  - backend/api (5): claims.py, kpis.py, modules.py, schemas.py, webhooks_monday.py
  - backend/claims (5): models.py, priority.py, service.py, store.py, ticketing.py
  - backend/audit (2): decisions.py, writer.py
  - backend/raíz (2): config.py, main.py
  - documentos (2): acceso-export-cargowise.md, verificacion-frd-v5.1.md
  - backend/analytics (1): kpis.py
  - backend/common (1): timeutil.py
  - backend/domain (1): tickets.py
  - backend/orchestration (1): pipeline.py
  - backend/security (1): permissions.py
  - backend/storage (1): provider.py
  - migraciones (1): 0014_claim_priority_evidence.py
  - tasks.md (1): tasks.md
- Pruebas: pytest ×38 · tests del front ×35 · arch-review ×34 · ruff format ×16 · despliegue con gate completo ×8 · ruff ×6
- Despliegues: 27 · agentes lanzados: 15

**22-09-2026 · 1.4 h**
- Cambios: 24 archivos
  - otros (8): .env.example, MEMORY.md, entra_oidc_and_magic_link_good_patterns.md, fake_mode_auth_backdoors.md, independent-auth-mode-flags.md, narrow-except-http-client-wrapper.md, password_hashing_good_patterns.md, state.md
  - infra (3): compose.vps-dev.yaml, deploy.sh, setup-login.sh
  - tests backend (3): test_api_auth.py, test_api_auth_password.py, test_magic_mail.py
  - backend/raíz (2): __main__.py, config.py
  - backend/security (2): magic_mail.py, passwords.py
  - front/otros (2): login-helpers.ts, portal.spec.ts
  - backend/api (1): auth.py
  - documentos (1): login.md
  - front/común (1): api.service.ts
- Pruebas: ruff format ×13 · pytest ×7 · arch-review ×3 · despliegue con gate completo ×2 · tests del front ×1 · ruff ×1
- Despliegues: 19 · agentes lanzados: 4

**24-09-2026 · 0.8 h**
- Cambios: 9 archivos
  - otros (4): MEMORY.md, audit_summary_pii_via_fstring.md, jit_invite_by_email_sso_binding.md, lifecycle-status-added-to-identity.md
  - front/pantallas (2): team.html, team.ts
  - backend/api (1): team.py
  - tests backend (1): test_team.py
- Pruebas: ruff format ×11 · arch-review ×5 · pytest ×3 · ruff ×1 · tests del front ×1 · evals LLM ×1
- Despliegues: 4 · agentes lanzados: 3

**25-09-2026 · 3.8 h**
- Cambios: 276 archivos
  - tests backend (48): conftest.py, test_account.py, test_api_account.py, test_api_auth_entra.py, test_api_mail_rules_calendar.py, test_api_modules.py, test_api_rates.py, test_api_schedules.py, test_assistant_signature.py, test_budget_degradation.py …
  - front/pantallas (44): account.html, account.scss, account.spec.ts, account.ts, calendar.html, calendar.scss, calendar.spec.ts, calendar.ts, claims.html, claims.spec.ts …
  - backend/orchestration (26): __init__.py, account.py, azure_openai.py, budget.py, calendar.py, claim.py, fake.py, foundry_anthropic.py, identity_replies.py, intent.py …
  - otros (16): .env.example, MEMORY.md, audit_summary_pii_via_fstring.md, auto-signature-implies-unearned-human-review.md, broad_internal_audit_read_default.md, button_id_before_llm_confirm_pattern.md, explore_t7.py, explore_t8.py, issues.md, llm_prices.json …
  - backend/api (16): account.py, admin.py, auth.py, calendar.py, claims.py, mail.py, me.py, modules.py, orgchart.py, quotes.py …
  - backend/mail (13): classify.py, digest.py, digest_schedule.py, fake_graph.py, gateway.py, graph_client.py, graph_tokens.py, intake.py, mail_rules.py, mail_rules_flow.py …
  - front/común (13): api.service.spec.ts, api.service.ts, app.routes.ts, fecha.pipe.ts, guards.spec.ts, guards.ts, i18n.service.spec.ts, i18n.service.ts, module-presentation.spec.ts, module-presentation.ts …
  - evals (10): analytics_plan.jsonl, claims_extract.jsonl, graders.py, mail_classify.jsonl, mail_reply.jsonl, quotes_extract.jsonl, report_narrative.jsonl, run_evals.py, thresholds.json, translate.jsonl
  - backend/rates (8): __init__.py, domain.py, format.py, importer.py, models.py, query_extract.py, store.py, template.py
  - migraciones (8): 0021_mail_priority_and_alerts.py, 0022_claim_org_quote_dimensions.py, 0023_shipment_invalid_containers.py, 0024_assistant_identity.py, 0025_mail_rules_and_calendar.py, 0026_rates_rate.py, 0027_owner_rls.py, env.py
  - backend/account (7): __init__.py, export.py, fake_source.py, format.py, models.py, powerbi_source.py, source.py
  - backend/calendar (7): __init__.py, actions.py, fake_calendar.py, gateway.py, graph_calendar.py, parse.py, proposals.py
  - backend/claims (7): __init__.py, extract.py, models.py, priority.py, service.py, store.py, ticketing.py
  - backend/mart (5): curate.py, models.py, query.py, repo.py, scope.py
  - backend/quotes (5): desk.py, extract.py, models.py, service.py, store.py
  - backend/rules (5): engine.py, models.py, parse.py, runtime.py, store.py
  - backend/schedules (5): __init__.py, extract.py, ports.py, render.py, source.py
  - backend/identity (4): models.py, orgchart.py, service.py, sql_store.py
  - backend/reports (4): charts.py, period.py, shipments.py, xlsx_utils.py
  - backend/channels (3): client.py, messages.py, window.py
  - backend/raíz (3): __main__.py, config.py, main.py
  - backend/analytics (2): inactivity.py, service.py
  - backend/common (2): errors.py, signature.py
  - backend/db (2): base.py, session.py
  - backend/security (2): permissions.py, session.py
  - documentos (2): 0007-escalada-fallback-y-sensibilidad-llm.md, data-mart-v1.md
  - infra (2): README.md, k6-llm.js
  - backend/audit (1): decisions.py
  - backend/domain (1): tickets.py
  - backend/integrations (1): status.py
  - backend/notify (1): tickets.py
  - front/otros (1): styles.scss
- Pruebas: pytest ×161 · ruff ×72 · ruff format ×57 · evals LLM ×34 · arch-review ×12 · tests del front ×9
- Despliegues: 3 · agentes lanzados: 32

**26-09-2026 · 3.1 h**
- Cambios: 124 archivos
  - front/pantallas (49): admin-shell.html, admin-shell.spec.ts, admin-shell.ts, audit.html, audit.ts, calendar.html, calendar.ts, chat.html, chat.spec.ts, chat.ts …
  - tests backend (14): test_access_profiles.py, test_adversarial.py, test_budget_profiles.py, test_claims.py, test_contacts.py, test_curate.py, test_db_postgres.py, test_e2e_whatsapp.py, test_graph_mail.py, test_locodes.py …
  - backend/orchestration (11): budget.py, contacts.py, fake.py, intent.py, intent_v1.py, mail.py, pipeline.py, registry.py, report.py, services.py …
  - backend/api (8): admin.py, auth.py, contacts.py, deps.py, mail.py, modules.py, schemas.py, team.py
  - backend/mail (8): auth_results.py, gateway.py, graph_client.py, graph_tokens.py, intake.py, llm_classify.py, llm_draft.py, llm_search.py
  - backend/identity (5): corporate_directory.py, models.py, service.py, sql_store.py, whatsapp_directory.py
  - front/común (5): api.service.ts, app.routes.ts, i18n.service.spec.ts, i18n.service.ts, types.ts
  - backend/security (4): permissions.py, profile_sql.py, profiles.py, session.py
  - infra (3): onedrive-sync-cron.sh, onedrive_sync.py, set_onedrive_password.py
  - otros (3): MEMORY.md, issues.md, profile_scoped_below_level_good_pattern.md
  - backend/audit (2): models.py, writer.py
  - backend/channels (2): messages.py, wasender.py
  - backend/raíz (2): config.py, main.py
  - migraciones (2): 0028_access_profiles.py, 0029_audit_media_ref.py
  - backend/claims (1): service.py
  - backend/common (1): ids.py
  - backend/mart (1): locodes.py
  - documentos (1): data-mart-v1.md
  - evals (1): dataset.jsonl
  - tasks.md (1): tasks.md
- Pruebas: pytest ×29 · ruff format ×19 · ruff ×15 · tests del front ×12 · evals LLM ×6
- Despliegues: 2 · agentes lanzados: 14

**27-09-2026 · 0.7 h**
- Cambios: 3 archivos
  - backend/channels (1): allowlist.py
  - documentos (1): estado-frd-v5.1-2026-09-28.md
  - infra (1): k6-arch-100.js
- Pruebas: ruff format ×1
- Despliegues: 1

**28-09-2026 · 0.6 h**
- Cambios: 1 archivos
  - infra (1): stress-steps.sh

**29-09-2026 · 1.5 h**
- Cambios: 4 archivos
  - documentos (2): cumplimiento-frd-v5.1-desde-punto-4.md, cumplimiento-frd-v5.1-punto-por-punto.md
  - backend/channels (1): message_log.py
  - tests backend (1): test_wa_message_log.py
- Pruebas: ruff format ×4 · pytest ×2 · ruff ×1
- Despliegues: 6

**30-09-2026 · 6.1 h**
- Cambios: 11 archivos
  - backend/mail (4): html_view.py, pending_followups.py, pending_snapshot.py, urgent_alerts.py
  - documentos (3): casos-de-uso-y-flujos.md, gobernanza-ia.md, matriz-integraciones.md
  - tests backend (2): test_mail_urgent_alerts.py, test_whatsapp_internal.py
  - backend/orchestration (1): openai_api.py
  - otros (1): hola-con-respecot-al-foamy-fern.md
- Pruebas: ruff format ×45 · pytest ×14 · ruff ×7 · despliegue con gate completo ×4 · tests del front ×2 · evals LLM ×1
- Despliegues: 19 · agentes lanzados: 5

**01-10-2026 · 1.7 h**
- Cambios: 7 archivos
  - backend/reports (3): claims_report.py, kpi_report.py, tabular.py
  - backend/analytics (1): volume_alerts.py
  - backend/calendar (1): confirmations.py
  - backend/common (1): notices_i18n.py
  - documentos (1): encendido-sistemas-fuente.md
- Pruebas: ruff format ×13 · pytest ×8 · tests del front ×5 · ruff ×3 · despliegue con gate completo ×2
- Despliegues: 6

**02-10-2026 · 2.1 h**
- Cambios: 3 archivos
  - documentos (2): 0008-angular-material.md, auditoria-ux-2026-10-03.md
  - front/común (1): echarts-core.ts
- Pruebas: tests del front ×19 · ruff format ×3 · pytest ×2 · despliegue con gate completo ×1
- Despliegues: 7

**03-10-2026 · 0.2 h**
- Cambios: 1 archivos
  - front/común (1): paging.ts
- Pruebas: tests del front ×1
- Despliegues: 1

**05-10-2026 · 4.0 h**
- Cambios: 67 archivos
  - front/pantallas (48): account.scss, admin-shell.html, admin-shell.scss, analitica.html, analitica.scss, analitica.ts, audit.html, audit.ts, ayuda.scss, calendar.html …
  - front/común (8): app.routes.ts, design-tokens.spec.ts, fecha.pipe.ts, pager.ts, route-map.ts, shell.html, shell.scss, shell.ts
  - otros (4): Embarque.dc.html, Main.dc.html, Tablero.dc.html, canvas.json
  - documentos (3): 0009-plantilla-v2.md, auditoria-12-agentes-2026-10-05.md, auditoria-estructura-6-agentes-2026-10-06.md
  - front/otros (2): _tokens.scss, styles.scss
  - tasks.md (1): tasks.md
  - tests backend (1): test_shipments_order.py
- Pruebas: tests del front ×41 · pytest ×6 · ruff format ×4 · ruff ×2 · despliegue con gate completo ×1
- Despliegues: 11 · agentes lanzados: 26

**06-10-2026 · 6.4 h**
- Cambios: 34 archivos
  - front/pantallas (10): chat.html, chat.ts, preferencias.ts, wa-chat.html, wa-chat.scss, wa-chat.spec.ts, wa-chat.ts, whatsapp-chats.html, whatsapp-chats.scss, whatsapp-chats.ts
  - backend/api (4): auth.py, media.py, notifications.py, schemas.py
  - documentos (4): 0010-centro-de-avisos.md, estado-desarrollo-2026-10-06.md, verificacion-frd-9-2026-10-07.md, verificacion-frd-v5.1.md
  - backend/identity (2): corporate_directory.py, service.py
  - backend/notify (2): models.py, notices.py
  - backend/security (2): media_links.py, session.py
  - front/común (2): assistant-avatar.ts, session.store.ts
  - tests backend (2): test_notices.py, test_whatsapp_reply.py
  - backend/channels (1): messages.py
  - backend/common (1): signature.py
  - backend/reports (1): shipments.py
  - front/otros (1): wai-locations.conf
  - migraciones (1): 0034_notify_notice.py
  - otros (1): llenar_bitacora.py
- Pruebas: ruff format ×27 · pytest ×26 · ruff ×19 · tests del front ×19 · despliegue con gate completo ×3 · arch-review ×1
- Despliegues: 23 · agentes lanzados: 17

**07-10-2026 · 4.8 h**
- Cambios: 12 archivos
  - backend/mail (3): followup_drafts.py, priority_teach.py, summarize.py
  - otros (3): MEMORY.md, cableado-pendiente.md, fake-masks-real-adapter-field.md
  - tests backend (2): test_mail_summarize.py, test_signature_config.py
  - backend/channels (1): outreach.py
  - backend/orchestration (1): mail_summary_v1.py
  - documentos (1): preguntas-wideline-frd-9.md
  - front/otros (1): wai-4.svg
- Pruebas: ruff format ×94 · pytest ×69 · ruff ×34 · arch-review ×8 · tests del front ×4 · despliegue con gate completo ×1
- Despliegues: 6 · agentes lanzados: 18

**10-10-2026 · 1.3 h**
- Pruebas: ruff ×3 · arch-review ×2 · despliegue con gate completo ×2 · tests del front ×1 · pytest ×1
- Despliegues: 4 · agentes lanzados: 2

## Cómo se midió

- **Bloque:** actividad continua. Una pausa de más de 30 minutos lo corta y no se cuenta. Los agentes en paralelo cuentan una sola vez.
- **Días sin transcript:** se cubren con los commits, 30 minutos de arranque por bloque.
- **Pruebas:** veces que se lanzó cada suite, no pruebas individuales (una corrida de pytest son ~1.900).
- **Es un piso, no un techo:** no se cuenta nada que no deje transcript ni commit.

Se rehace con `bash actualizar.sh`; sale del mismo cálculo que `informe-horas-trabajadas.md`, así que los dos no pueden contradecirse.
