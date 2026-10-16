# Diccionario de Datos: Campos de la Incidencia (Post-Venta Vitain)

Este documento detalla exhaustivamente todos los campos utilizados en el ciclo de vida de una **Incidencia** en la plataforma, abarcando su representación en la Base de Datos (`form_respuesta`, `form_respuesta_detalle`, `form_evidencia`), los DTOs de comunicación del Backend (Spring Boot), la sincronización con el CRM (Bitrix24) y su consumo/visualización en el Frontend (Angular).

---

## 1. Cabecera Principal de la Incidencia (`FormularioRespuesta` / `form_respuesta`)

Representa la entidad central de cada solicitud de post-venta registrada en el sistema.

| Campo (Frontend / DTO) | Columna BD (`form_respuesta`) | Tipo de Dato | Origen / Asignado Por | Descripción y Uso en Frontend / Flujo |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `BIGINT` (PK) | Sistema (Auto) | Identificador numérico único de la incidencia. Usado en rutas (`/post-venta/incidencias/detalle/:id`) y canales WebSocket. |
| `numeroRegistro` | `numero_registro` | `VARCHAR(800)` | Backend (`ATN-{id}`) | Código correlativo amigable visible para el cliente (ej. `ATN-104`). Usado en filtros de búsqueda, tarjetas y WhatsApp. |
| `uuidDispositivo` | `uuid_dispositivo` | `VARCHAR(64)` | Frontend Móvil | Identificador único del dispositivo emisor para evitar envíos duplicados por problemas de red. |
| `fechaReal` | `fecha_real` | `TIMESTAMP` | Dispositivo | Fecha y hora exacta de captura en el celular/navegador del usuario. |
| `fechaRegistro` | `fecha_registro` | `TIMESTAMP` | Backend | Fecha y hora oficial en que la incidencia fue persistida en la base de datos. Se muestra en las tarjetas y encabezado del detalle. |
| `estadoAtencion` | `estado_atencion` | `VARCHAR(30)` | Flujo (SAC / CRM) | Estado actual del ciclo de vida (`REGISTRADO`, `OBSERVADO`, `EN_PROGRAMACION`, `PROGRAMADO`, etc.). Controla qué botones y avisos se renderizan. |
| `formularioId` | `formulario_id` | `BIGINT` (FK) | Frontend / Form | ID del formulario dinámico utilizado para capturar las preguntas y respuestas. Permite recuperar el esquema de campos. |
| `formularioNombre` | *(relación `formulario`)* | `VARCHAR` | Backend | Nombre legible del formulario (ej. "Formulario Post-Venta Departamentos"). |
| `registradoPor` | `registrado_por_id` | `BIGINT` (FK) | Auth (JWT) | Usuario propietario que creó la incidencia (`id`, `nombres`, `apellidos`, `email`). |
| `comentarioFlujo` | `comentario_flujo` | `TEXT` | SAC / CRM | Mensaje informativo o resumen del estado que se le muestra directamente al cliente (ej. detalle del cierre o motivo de programación). |
| `detalle` | `detalle` | `TEXT` | Backend / SAC | Notas o detalles técnicos internos adicionales sobre la incidencia. |

---

## 2. Validación de SAC y Subsanación de Observaciones

Campos utilizados cuando el equipo de Atención al Cliente (SAC) evalúa la solicitud inicial y solicita correcciones al propietario.

| Campo (Frontend / DTO) | Columna BD (`form_respuesta`) | Tipo de Dato | Origen / Asignado Por | Descripción y Uso en Frontend / Flujo |
| :--- | :--- | :--- | :--- | :--- |
| `fechaValidacion` | `fecha_validacion` | `TIMESTAMP` | SAC / Bitrix24 | Fecha/hora en la que SAC evaluó y validó la admisibilidad de la incidencia. |
| `observacionesValidacion` | `observaciones_validacion` | `TEXT` | SAC / Bitrix24 | Texto del motivo por el cual la incidencia fue observada (ej. *"Fotos borrosas, no se aprecia la fuga"*). Se muestra destacado en el banner amarillo de subsanación. |
| `usuarioValidadorId` | `usuario_validador_id` | `BIGINT` (FK) | SAC / Bitrix24 | ID del colaborador de SAC que realizó la validación. |
| `usuarioValidadorEmail` | *(resolución usuario)* | `VARCHAR` | DTO / Bitrix24 | Correo del validador de SAC remitido desde Bitrix24. |
| `nuevosArchivos` *(DTO subsanar)* | *(hacia `form_evidencia`)* | `List<String>` | Propietario (UI) | URLs de las nuevas fotos/videos que el propietario sube al subsanar en estado `OBSERVADO`. |
| `comentario` / `comentarioSubsanacion` | *(hacia `comentario_flujo`)* | `TEXT` | Propietario (UI) | Texto explicativo opcional ingresado por el propietario al momento de enviar las nuevas evidencias. |

---

## 3. Coordinación y Programación de la Visita Técnica

Campos donde SAC propone fechas tentativas y el propietario selecciona/confirma el día de la inspección.

| Campo (Frontend / DTO) | Columna BD (`form_respuesta`) | Tipo de Dato | Origen / Asignado Por | Descripción y Uso en Frontend / Flujo |
| :--- | :--- | :--- | :--- | :--- |
| `fechaTentativa1` | `fecha_tentativa_1` | `TIMESTAMP` | SAC / Bitrix24 | Opción de fecha y hora 1 propuesta por SAC para la visita técnica. |
| `fechaTentativa2` | `fecha_tentativa_2` | `TIMESTAMP` | SAC / Bitrix24 | Opción de fecha y hora 2 propuesta por SAC para la visita técnica. |
| `fechaTentativa3` | `fecha_tentativa_3` | `TIMESTAMP` | SAC / Bitrix24 | Opción de fecha y hora 3 propuesta por SAC para la visita técnica. |
| `fechaConfirmadaCoordinacion` | `fecha_confirmada_coordinacion`| `TIMESTAMP` | Propietario / SAC | Fecha y hora definitiva seleccionada por el cliente. Al elegirla pasa el estado a `PROGRAMADO`. |
| `fechaEjecucionVisita` | `fecha_ejecucion_visita` | `TIMESTAMP` | Arquitecto Campo | Fecha real en la que el técnico o arquitecto asistió al inmueble a inspeccionar la falla. |
| `confirmacionProgramacionFechaVisita` | `confirmacion_programacion_fecha_visita` | `TIMESTAMP` | SAC / CRM | Registro oficial de la confirmación de la cita técnica. |
| `labelProgramacionVisita` | `label_programacion_visita` | `VARCHAR(255)` | SAC / CRM | Etiqueta o glosa personalizada descriptiva de la cita técnica. |

---

## 4. Garantía y Plan de Trabajo (PV03 - Dictamen Técnico y Reparaciones)

Campos alimentados tras la visita técnica cuando se aprueba la procedencia por garantía (Vitain o Proveedor).

| Campo (Frontend / DTO) | Columna BD (`form_respuesta`) | Tipo de Dato | Origen / Asignado Por | Descripción y Uso en Frontend / Flujo |
| :--- | :--- | :--- | :--- | :--- |
| `planTrabajo` | `plan_trabajo` | `TEXT` | Arquitecto / SAC | Detalle explicativo de los trabajos de refacción o reparación que se ejecutarán. Mostrado en tarjeta "Trabajo a realizar". |
| `fechaPropuesta1` | `fecha_propuesta_1` | `TIMESTAMP` | SAC / CRM | Opción 1 de fecha propuesta al cliente para ingresar a realizar las reparaciones. |
| `fechaPropuesta2` | `fecha_propuesta_2` | `TIMESTAMP` | SAC / CRM | Opción 2 de fecha propuesta al cliente para ingresar a realizar las reparaciones. |
| `duracionEstimada` | `duracion_estimada` | `VARCHAR` | Arquitecto / SAC | Tiempo estimado para culminar los trabajos (ej. *"2 días hábiles"*, *"4 horas"*). |
| `responsableEjecucion` | `responsable_ejecucion` | `VARCHAR` | SAC / Contratista | Nombre del maestro de obra, especialista o cuadrilla asignada para los trabajos. |
| `observaciones` | `observaciones` | `TEXT` | Técnico / SAC | Especificaciones técnicas o consideraciones preventivas adicionales. |
| `proveedorResponsable` | `proveedor_responsable` | `VARCHAR` | Logística / SAC | Nombre de la empresa o contratista externo responsable (si aplica). |
| `formatoVitainTipo` | `formato_vitain_tipo` | `VARCHAR` | CRM / Vitain | Clasificación macro de garantía Vitain. |
| `formatoVitainSubTipo` | `formato_vitain_subtipo` | `VARCHAR` | CRM / Vitain | Subclasificación específica del tipo de falla. |
| `tipoGarantiaVitain` | `tipo_garantia_vitain` | `VARCHAR` | CRM / Vitain | Modalidad de cobertura de la garantía. |
| `clasificacionVitain` | `clasificacion_vitain` | `VARCHAR` | Peritaje Técnico | Calificación técnica de la procedencia. |
| `datosContacto` | `datos_contacto` | `TEXT` | Propietario / SAC | Números y canales de contacto del propietario para coordinar la cuadrilla en obra. |

---

## 5. Subprocesos Operativos y CRM (Servicios PV03a, Materiales PV03b y Aprobadores)

Campos operativos manejados principalmente entre el Backend y Bitrix24 para compras, costos y aprobaciones.

| Campo (Frontend / DTO) | Columna BD (`form_respuesta`) | Tipo de Dato | Origen / Asignado Por | Descripción y Uso en Frontend / Flujo |
| :--- | :--- | :--- | :--- | :--- |
| `numeroOS` | `numero_os` | `VARCHAR` | Costos / SAC | Número de Orden de Servicio emitida para la reparación. |
| `costoServicio` | `costo_servicio` | `VARCHAR` | Costos | Monto presupuestado/aprobado por el servicio de mano de obra. |
| `fechaInstalacion` | `fecha_instalacion` | `TIMESTAMP` | Cuadrilla | Fecha en la que se instalaron los repuestos o materiales. |
| `duracionInstalacion` | `duracion_instalacion` | `VARCHAR` | Cuadrilla | Horas o jornadas empleadas en la instalación. |
| `estadoStock` | `estado_stock` | `VARCHAR` | Almacén | Estado de disponibilidad de insumos (`EN_STOCK`, `POR_COMPRAR`, etc.). |
| `numeroOC` | `numero_oc` | `VARCHAR` | Logística | Número de Orden de Compra para adquisición de insumos/repuestos. |
| `costoMaterial` | `costo_material` | `VARCHAR` | Logística | Importe del costo de materiales comprados. |
| `fechaRecepcionMaterial` | `fecha_recepcion_material` | `TIMESTAMP` | Almacén / Obra | Fecha en que el material llegó físicamente a obra para iniciar trabajos. |
| `fechaRecepcionMateriales`| `fecha_recepcion_materiales`| `TIMESTAMP` | Almacén / Obra | Variante de marca de tiempo para control logístico de recepción de materiales. |
| `fechaServicioInstalacion`| `fecha_servicio_instalacion`| `TIMESTAMP` | Operaciones | Fecha programada del servicio de instalación técnica. |
| `estadoServicio` | `estado_servicio` | `VARCHAR` | Operaciones | Estado operativo del servicio (`PENDIENTE`, `EN_CURSO`, `COMPLETADO`). |
| `aprobadorId` | `aprobador_id` | `VARCHAR` | CRM | ID del usuario que autoriza los costos/presupuesto en Bitrix24. |
| `aprobadorNombre` | `aprobador_nombre` | `VARCHAR` | CRM | Nombre completo del aprobador financiero/técnico. |
| `aprobadorCorreo` | `aprobador_correo` | `VARCHAR` | CRM | Correo electrónico del aprobador. |
| `aprobadorUserId` | `aprobador_user_id` | `VARCHAR` | CRM | Identificador de usuario CRM del aprobador. |
| `dealId` | *(DTO Integración)* | `BIGINT` | Bitrix24 CRM | ID del Deal / Negociación vinculado en Bitrix24. |
| `fechaEntrega` | *(DTO Integración)* | `DATE` | Bitrix24 CRM | Fecha de entrega del inmueble obtenida de la negociación en Bitrix. |

---

## 6. Respuestas Dinámicas a Preguntas (`FormularioRespuestaDetalle` / `form_respuesta_detalle`)

Cada incidencia almacena en esta tabla hija las respuestas a las preguntas individuales del formulario dinámico de post-venta.

| Campo | Columna BD (`form_respuesta_detalle`) | Tipo de Dato | Descripción y Uso en Frontend |
| :--- | :--- | :--- | :--- |
| `id` | `id` | `BIGINT` (PK) | Identificador único del registro de respuesta al campo. |
| `respuesta` | `respuesta_id` | `BIGINT` (FK) | Relación hacia la cabecera `form_respuesta`. |
| `fieldKey` | `field_key` | `VARCHAR(80)` | Clave técnica única de la pregunta (ej. `departamento`, `ambiente`, `tipo_incidencia`, `descripcion_falla`). |
| `valor` | `valor` | `VARCHAR(500)` | Contenido guardado: texto libre, ID de opción seleccionada, array serializado de IDs o URL de archivo. |
| `valorLabel` | `valor_label` | `VARCHAR(300)` | Texto legible mostrado en la interfaz (ej. *"Cocina / Lavandería"*, *"Filtración en grifería"*). En `incidencia-detalle` se mapea con la pregunta correspondiente. |

---

## 7. Evidencias y Multimedia (`EvidenciasFormulario` / `form_evidencia`)

Guarda los archivos (imágenes, videos o PDFs) adjuntados originalmente en el formulario y los adjuntados durante la subsanación.

| Campo | Columna BD (`form_evidencia`) | Tipo de Dato | Descripción y Uso en Frontend |
| :--- | :--- | :--- | :--- |
| `id` | `id` | `BIGINT` (PK) | Identificador único de la evidencia adjunta. |
| `formularioRespuesta` | `formulario_respuesta_id` | `BIGINT` (FK) | Relación hacia la cabecera de la incidencia `form_respuesta`. |
| `nombreArchivo` | `nombre_archivo` | `VARCHAR` | Nombre del archivo o etiqueta descriptiva (ej. *"Archivo: Foto de la falla"*). |
| `pathArchivo` | `path_archivo` | `VARCHAR` | URL pública / S3 donde está alojado el archivo. Se utiliza para mostrar miniaturas, abrir en lightbox modal con zoom/rotación o descargar. |
| `usuarioSubida` | `creado_por` | `BIGINT` (FK) | Usuario que adjuntó el archivo (cliente o técnico). |

---

## 8. Catálogo de Estados (`EstadoAtencion`) y Representación en la UI

Estados que toma el campo `estadoAtencion` a lo largo del flujo y su visualización en el frontend:

| Clave Backend (`EstadoAtencion`) | Etiqueta Propietario (`estadoLabel`) | Nombre Formal (`estadoNombre`) | Paso (1-5) | Indicador Visual / Botón Principal |
| :--- | :--- | :--- | :---: | :--- |
| `EN_REGISTRO` / `PENDIENTE` / `REGISTRADO` | Recibida | Registrado | 1 | Gris / Azul claro · *"Ver detalle"* |
| `EN_EVALUACION` | En revisión | En evaluación | 2 | Azul cielo · *"Ver detalle"* |
| `OBSERVADO` | Necesitamos nuevas fotos | Observado | 2 (Alerta) | Ámbar / Pulsante · **Acción requerida: "Subsanar Observación - Subir Fotos"** |
| `REQUERIMIENTO_INFORMACION` | Falta información | Requerimiento de información | 2 (Alerta) | Naranja · *"Ver detalle"* |
| `ATENDIBLE` | Aprobada | Atendible | 2 | Celeste · *"Ver detalle"* |
| `EN_PROGRAMACION` | Elige día de visita | En programación | 3 (Acción) | Índigo / Pulsante · **Acción requerida: "Confirmar Fecha de Visita"** |
| `PROGRAMADO` | Visita agendada | Programado | 3 | Verde esmeralda · *"Ver detalle"* / Tarjeta de fecha confirmada |
| `VISITADO` | Visita realizada | Visitado | 3 | Verde azulado (Teal) · *"Ver detalle"* |
| `GARANTIA_POR_VITAIN` | Reparación aprobada | Garantía por Vitain | 4 | Púrpura · Botones interactivos para aceptar/rechazar plan de trabajo |
| `GARANTIA_POR_PROVEEDOR` | Reparación aprobada | Garantía por proveedor | 4 | Rosa · *"Ver detalle"* |
| `GARANTIA_VENCIDA` | Garantía vencida | Garantía vencida | 4 (Alerta) | Rojo oscuro · *"Ver detalle"* |
| `EN_REPARACION` / `TRABAJOS_INICIADOS` | En reparación | En reparación | 4 | Ámbar · *"Ver detalle"* |
| `CONCLUIDO` / `CERRADO` / `ATENDIDO` | Terminada | Concluido | 5 | Verde esmeralda · *"Ver detalle"* / Banner de conformidad |
| `RECHAZADO` | No procede | Rechazado | 2 | Rosa rojizo · *"Ver detalle"* |
