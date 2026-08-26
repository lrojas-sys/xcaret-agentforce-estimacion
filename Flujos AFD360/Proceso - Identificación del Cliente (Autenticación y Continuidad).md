# PROCESO – IDENTIFICACIÓN DEL CLIENTE (AUTENTICACIÓN Y CONTINUIDAD)

**Objetivo general:** Identificar al cliente de forma segura según el canal, validar su identidad, consultar información relevante y garantizar continuidad con el contexto completo en caso de transferencia.

## Tabla de acciones del flujo

| # | HU | Acción | Descripción | Etapa del flujo | Objetivo |
|---|----|--------|--------------|------------------|----------|
| 1 | Rq3-HU18 | Determinar método de identificación según canal | Detecta automáticamente el canal de entrada (WhatsApp, Messenger, Web, Instagram, otros) y asigna el método de autenticación correspondiente (número, correo, o transferencia si no hay datos) | Detección y Canal | Enrutar al cliente hacia el método de identificación correcto desde el primer contacto |
| 2 | Rq3-HU19 | Identificación automática por número telefónico (WhatsApp) | Captura y normaliza el número (con código de país), busca coincidencia en CRM (Contact o CRM Profile); si hay match único, autentica preliminarmente | Identificación | Autenticar al cliente sin fricción usando el número ya disponible en WhatsApp |
| 3 | Rq3-HU20 | Identificación por correo electrónico en redes y web | Solicita el correo de forma natural, valida su formato y busca coincidencia en CRM o Perfil Unificado | Identificación | Identificar al cliente en canales donde no hay número disponible (redes/web) |
| 4 | Rq3-HU20.5 | Identificación con más de un correo asociado | Considera todos los correos asociados al cliente, usa el más reciente registrado, y si no coincide, deriva a manejo de intento fallido o registro en Cognito | Identificación | Resolver ambigüedad cuando el cliente tiene múltiples correos vinculados |
| 5 | Rq3-HU21 | Validar consentimiento (Opt-In) | Consulta el campo de consentimiento/notificaciones; si está activo el flujo continúa, si está inactivo o vacío no se muestran datos sensibles | Validación | Garantizar que solo se comparta información sensible con clientes que dieron consentimiento |
| 6 | Rq3-HU22 | Gestionar autenticación fallida por no coincidencia de datos | Informa que no fue posible validar la identidad, permite hasta 3 reintentos; si se exceden, transfiere a ejecutivo humano | Manejo de Excepciones | Evitar bloqueos definitivos dando oportunidades controladas de reintento antes de escalar |
| 7 | Rq3-HU23 | Gestionar fallas por respuestas incorrectas, negativas o evasión | Contabiliza intentos fallidos (máx. 3); un rechazo o abandono del agente cuenta como intento; al llegar al límite transfiere y registra el motivo | Manejo de Excepciones | Controlar el comportamiento evasivo o de rechazo del cliente durante la autenticación |
| 8 | Rq3-HU23.1 | Configuración de vigencia de conversación | Define timeout estándar de 24 horas (validado con negocio); al expirar reinicia el flujo o solicita nuevamente el contexto | Manejo de Excepciones | Evitar que sesiones abandonadas queden abiertas indefinidamente |
| 9 | Rq3-HU23.2 | Atención fuera de horario y generación de caso | Gestiona la interacción cuando ocurre fuera del horario de atención, generando un caso para seguimiento posterior | Manejo de Excepciones | Asegurar continuidad de servicio aun sin atención en tiempo real |
| 10 | Rq3-HU24 | Registrar estado de autenticación del cliente | Registra el estado final (Autenticado exitoso, No identificado, Intentos excedidos, Cliente solicita humano, Opt-in no válido); solo "Autenticado exitoso" permite ver datos sensibles | Validación | Dejar constancia formal y auditable del resultado de la autenticación |
| 11 | Rq3-HU25 | Consultar productos o compras activas del cliente autenticado | Solo si está autenticado y tiene reserva/upgrade pagado; consulta reservas/productos activos en Salesforce y muestra resumen (fecha, hotel, parque, transporte, cortesías, adultos, menores) | Consulta y Servicio | Ofrecer al cliente información relevante y personalizada de sus compras/reservas |
| 12 | Rq3-HU26 | Ofrecer envío de detalle por correo electrónico | Tras mostrar la información, pregunta si el cliente desea recibir el detalle por correo; valida que exista correo asociado y registra aceptación/rechazo | Consulta y Servicio | Facilitar al cliente el acceso posterior a su información sin depender del chat |
| 13 | Rq3-HU27 | Transferir a ejecutivo humano por excepción de identificación | Transfiere automáticamente cuando no se localiza número/correo, hay respuesta incorrecta reiterada o rechazo de interacción; crea un Case con los datos capturados | Transferencia | Escalar a un agente humano los casos que el flujo automatizado no puede resolver |
| 14 | Rq3-HU28 / HU28.1 | Consultar Perfil Unificado / Xapi Profile | Consulta fuentes de identidad (Perfil Unificado y, como alterna, Xapi Profile) para homologar la respuesta al agente y documentar la fuente utilizada | Consulta y Servicio | Enriquecer y validar la identidad del cliente cruzando distintas fuentes de datos |
| 15 | Rq3-HU29 | Registrar trazabilidad completa del proceso de identificación | Registra canal, dato usado, número de intentos, estado final, motivo de transferencia, productos consultados y timestamps; deja todo disponible en Salesforce | Trazabilidad | Permitir reportería, auditoría de fallos y análisis de abandono del proceso |

## Notas del flujo

- El proceso es secuencial en 3 filas: pasos 1–10 (detección → validación), 11–13 (consulta/servicio → transferencia), 14–15 (fuentes de identidad → trazabilidad final).
- Solo con estado "Autenticado exitoso" se habilita el acceso a información sensible del cliente.
- La autenticación protege datos sensibles y cumple lineamientos de privacidad.
- El flujo es conversacional, adaptativo y centrado en el cliente.
- La trazabilidad permite medir fricción, calidad y eficiencia del proceso.

## Estados de autenticación

| Estado | Descripción |
|--------|-------------|
| Autenticado exitoso | Identidad validada correctamente. Acceso a información sensible. |
| No identificado | No se encontró coincidencia. |
| Intentos excedidos | Se superó el límite de intentos (3). |
| Cliente solicita humano | El cliente pidió hablar con un humano. |
| Opt-in no válido | Consentimiento no activo / inválido. |

## Alineación con el diagrama de Identificación Cognito → XapiProfile

Este flujo describe a nivel de negocio (HUs) el mismo proceso que el diagrama técnico `Proceso de Identificación Cognito → XapiProfile.md`. Cruce de ambos documentos:

| HU | Nodo(s) equivalente(s) en el diagrama | Estado |
|----|----------------------------------------|--------|
| Rq3-HU18 (método según canal) | Paso 2 (`get_channel_type`) y paso 8 (¿Canal WhatsApp?) | ✅ Alineado |
| Rq3-HU19 (identificación por teléfono) | Paso 9 (Búsqueda directa por CRM – Teléfono) | ✅ Alineado |
| Rq3-HU20 (identificación por correo) | Pasos 10, 12, 13, 14, 16-17 (Ofrecer Link Cognito → CRM por email → evaluar Cognito → Xapi) | ⚠️ Gap: la descripción de esta HU no menciona el paso de Cognito (ofrecer el link, evaluar `CognitoEmail`), que es el mecanismo real del diagrama para canales sin número disponible. Se sugiere actualizar la descripción de la HU para reflejarlo explícitamente. |
| Rq3-HU20.5 (múltiples correos) | Sin nodo equivalente | ⚠️ Gap: el diagrama no tiene una rama explícita para "más de un correo asociado" — solo maneja resultados booleanos (encontrado/no encontrado) en cada fuente. Falta definir en qué punto del diagrama se resolvería esta ambigüedad. |
| Rq3-HU21 (Opt-in) | Pasos 3-7 (términos y condiciones, campo noticias, marcar verdadero, Opt-in False) | ⚠️ Inconsistencia de orden: el diagrama valida el consentimiento **antes** de cualquier intento de identificación (pasos 3-7 preceden a los pasos 8-9); la tabla de HUs lista la identificación (HU19/HU20) **antes** del Opt-in (HU21). Confirmar con el equipo de diseño cuál orden es el correcto para evitar mostrar datos sensibles antes de validar consentimiento. |
| Rq3-HU22 / HU23 (fallos y reintentos) | Pasos 15, 19, 20 | ✅ Alineado |
| Rq3-HU27 (transferencia) | Pasos 20, 21, 22 | ✅ Alineado |
| Rq3-HU28 / HU28.1 (Perfil Unificado / Xapi Profile) | Pasos 16-17 (`CognitoEmail → Input_Xapi`, ¿Cliente encontrado?) | ✅ Resuelto — **Perfil Unificado y Xapi Profile son la misma fuente.** El cliente confirma que no utiliza Data 360 (Data Cloud) para unificación de perfiles: la consulta de Xapi Profile se hace directo contra Core CRM. Esto se refleja en la estimación de créditos (ver tabla abajo): la acción no genera cargo de Data Cloud. |
| — | Paso 11 (`Bandera_auth_CRM_Xapi`) | Ninguna HU referencia explícitamente este flag, que es el que decide técnicamente si la fuente usada es CRM o Xapi. Se sugiere trazarlo en HU19/HU20 para que la documentación de negocio y el diagrama técnico queden ligados. |

## Estimación de consumo de créditos

| # | HU | Acción | Etapa | ¿Consume créditos? | Créditos Agentforce (Flex Credits) | Créditos Data Cloud | Justificación |
|---|----|--------|-------|---------------------|--------------------------------------|----------------------|----------------|
| 1 | Rq3-HU18 | Determinar método de identificación según canal | Detección y Canal | No | 0 | 0 | Es lógica condicional de enrutamiento; no invoca ninguna acción de agente ni consulta a CRM/Data Cloud. |
| 2 | Rq3-HU19 | Identificación automática por número telefónico (WhatsApp) | Identificación | Sí | 20 | 0 | Ejecuta una acción estándar de Agentforce (búsqueda de Contact o CRM Profile); al ser consulta directa a Core CRM no genera cargo de Data Cloud. |
| 3 | Rq3-HU20 | Identificación por correo electrónico | Identificación | Sí | 20 | 0 | Acción estándar que consulta CRM/Perfil Unificado por correo; se asume consulta a Core CRM, sin cargo adicional de Data Cloud. |
| 4 | Rq3-HU20.5 | Identificación con más de un correo asociado | Identificación | No | 0 | 0 | Reutiliza los datos ya devueltos por la consulta de identificación por correo; no dispara una nueva acción salvo que derive a registro en Cognito (cubierto en la transferencia a ejecutivo). |
| 5 | Rq3-HU21 | Validar consentimiento (Opt-In) | Validación | No | 0 | 0 | Lee un campo del mismo registro ya consultado en la identificación previa; no es una acción ni consulta adicional. |
| 6 | Rq3-HU22 | Gestionar autenticación fallida por no coincidencia | Manejo de Excepciones | No | 0 | 0 | Es respuesta conversacional generada por el agente dentro del mismo turno; el registro formal solo ocurre en el paso de registro de estado. |
| 7 | Rq3-HU23 | Gestionar fallas por respuestas incorrectas/evasión | Manejo de Excepciones | No | 0 | 0 | Conteo de intentos en memoria de sesión, sin escritura a Salesforce hasta el registro final. |
| 8 | Rq3-HU23.1 | Configuración de vigencia de conversación | Manejo de Excepciones | No | 0 | 0 | Es un parámetro de configuración de sesión/timeout; no invoca ninguna acción ni consulta. |
| 9 | Rq3-HU23.2 | Atención fuera de horario y generación de caso | Manejo de Excepciones | Sí | 20 | 0 | Crea un registro de Case en Salesforce mediante Flow/Apex invocado por el agente; es una operación de Core CRM. |
| 10 | Rq3-HU24 | Registrar estado de autenticación del cliente | Validación | Sí | 20 | 0 | Escribe/actualiza el estado final en el registro del cliente en Salesforce mediante acción estándar de agente. |
| 11 | Rq3-HU25 | Consultar productos o compras activas | Consulta y Servicio | Sí | 20 | 0 | Consulta reservas/productos activos en Salesforce (Core CRM) mediante acción estándar de agente. |
| 12 | Rq3-HU26 | Ofrecer envío de detalle por correo electrónico | Consulta y Servicio | Sí | 20 | 0 | Requiere una acción (Flow/Apex) para validar el correo y enviar/registrar el detalle; es una operación de Core CRM. |
| 13 | Rq3-HU27 | Transferir a ejecutivo humano por excepción | Transferencia | Sí | 20 | 0 | Crea un Case con los datos capturados mediante acción estándar de agente. |
| 14 | Rq3-HU28 / HU28.1 | Consultar Perfil Unificado / Xapi Profile | Consulta y Servicio | Sí | 20 | 0 | Confirmado con el cliente: Perfil Unificado y Xapi Profile son la misma fuente, y el cliente no utiliza Data 360 (Data Cloud) para unificación de perfiles — la consulta de Xapi Profile es directa a Core CRM. Solo genera el costo de la acción estándar de Agentforce. |
| 15 | Rq3-HU29 | Registrar trazabilidad completa del proceso | Trazabilidad | No | 0 | 0 | Es logging/auditoría de fondo (background automation), que según el baseline no consume créditos; probablemente reutiliza la misma escritura del registro de estado. |

**Totales estimados por escenario:**
- Conversación exitosa con consulta e intención de recibir correo: 4 acciones que consumen créditos → **80 Flex Credits de Agentforce**, 0 de Data Cloud.
- Conversación que termina en transferencia a ejecutivo: se suman la creación de caso y, si se llegó a consultar el Perfil Unificado/Xapi Profile, esa acción → hasta **100 Flex Credits de Agentforce**, 0 de Data Cloud.

**Nota:** todo el consumo de este flujo es en Flex Credits de Agentforce (acciones estándar). No hay cargo de Data Cloud porque el cliente confirma que no utiliza Data 360 para unificación de perfiles — tanto la identificación por CRM como la consulta a Xapi Profile se resuelven directo contra Core CRM.
