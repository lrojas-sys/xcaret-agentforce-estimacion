# Proceso de Identificación Cognito → XapiProfile

**Objetivo general:** Identificar al cliente combinando la autenticación externa (Cognito), la consulta al CRM y el perfil unificado (Xapi Profile), controlando el consentimiento, el canal de origen y los reintentos, y escalando a un agente humano cuando no es posible identificar al cliente.

## Tabla de acciones del flujo

| # | Acción | Descripción | Etapa del flujo | Objetivo |
|---|--------|--------------|------------------|----------|
| 1 | START | Punto de entrada del flujo de identificación | Inicio | Iniciar el proceso de identificación del cliente |
| 2 | Identificar canal (`get_channel_type`) | Detecta el canal de comunicación del cliente mediante la función `get_channel_type` | Enrutamiento por canal | Determinar el canal de origen para adaptar el proceso de identificación |
| 3 | ¿Aceptas términos y condiciones? | Verifica si el cliente aceptó los términos y condiciones | Consentimiento | Confirmar que el cliente dio su consentimiento antes de continuar |
| 4 | Búsqueda a Contacto – Campo noticias | Si aceptó, busca en el registro de Contacto el campo de noticias/consentimiento | Consentimiento | Validar el estado de consentimiento registrado en el CRM |
| 5 | ¿Campo verificado a verdadero? | Verifica si el campo de consentimiento ya está marcado como verdadero | Consentimiento | Confirmar si el consentimiento ya se encuentra validado |
| 6 | Marcar como verdadero el campo consultado | Si el campo no estaba en verdadero, lo actualiza | Consentimiento | Sincronizar el estado de consentimiento del cliente en el sistema |
| 7 | Action Opt-in False en messaging session | Si el cliente no aceptó términos, marca el Opt-in como falso en la sesión de mensajería | Consentimiento | Registrar el rechazo del consentimiento para no enviar información sensible |
| 8 | ¿Canal de WhatsApp? | Verifica si el canal de origen de la conversación es WhatsApp | Enrutamiento por canal | Definir la ruta de identificación según si el canal es WhatsApp u otro |
| 9 | Búsqueda directa por CRM – Teléfono | Si el canal es WhatsApp, busca directamente al cliente en el CRM usando su número telefónico | Identificación por CRM | Identificar al cliente aprovechando el número ya disponible en WhatsApp |
| 10 | Ofrecer Link Cognito | Si el canal no es WhatsApp, ofrece al cliente un link de autenticación Cognito | Identificación vía Cognito | Iniciar la autenticación externa en canales donde no hay número disponible |
| 11 | Campo nuevo `Bandera_auth_CRM_Xapi` (Picklist) | Registra la bandera de autenticación que define la fuente a usar (CRM o Xapi) | Identificación vía Cognito/Xapi | Controlar qué fuente de identidad se utilizará en el intento actual |
| 12 | Buscar en CRM por email | Busca al cliente en el CRM utilizando el correo electrónico | Identificación por CRM | Localizar al cliente mediante su correo registrado en el CRM |
| 13 | ¿Respuesta de Flow CRM exitosa? (ID contact) | Evalúa si la búsqueda en CRM devolvió un ID de contacto válido | Identificación por CRM | Confirmar si el cliente fue encontrado en el CRM |
| 14 | ¿Respuesta de Cognito exitosa? (CognitoEmail) | Evalúa si Cognito devolvió el correo del cliente autenticado | Identificación vía Cognito | Confirmar si la autenticación en Cognito fue exitosa |
| 15 | Mensaje amigable de verificar/cambiar datos ingresados en el Link | Si Cognito falla, envía un mensaje solicitando verificar o corregir los datos ingresados | Identificación vía Cognito | Dar al cliente la oportunidad de corregir sus datos antes de reintentar |
| 16 | CognitoEmail → Input_Xapi | Envía el correo obtenido de Cognito como dato de entrada hacia Xapi Profile | Identificación vía Xapi | Transferir el dato de identidad para su validación en Xapi Profile |
| 17 | ¿Cliente encontrado? (Salida de Xapi) | Evalúa si Xapi Profile encontró coincidencia con el correo recibido | Identificación vía Xapi | Confirmar si el cliente existe en el perfil unificado (Xapi) |
| 18 | Cliente Identificado | Marca al cliente como identificado exitosamente (por CRM o por Xapi) | Resultado de identificación | Confirmar la identidad del cliente para continuar con el flujo de servicio |
| 19 | Intento n+1 – Proceso Identificación Cognito → Xapi | Registra un nuevo intento del ciclo de identificación cuando no se encontró al cliente | Manejo de reintentos | Permitir reintentos controlados del proceso de identificación |
| 20 | ¿Núm. de intentos => 3? | Verifica si se alcanzó el límite máximo de intentos (3) | Manejo de reintentos | Controlar cuántas veces se reintenta el proceso antes de escalar a un humano |
| 21 | Preparación de transferencia | Prepara la información del caso para transferirlo a un agente humano | Transferencia a agente humano | Consolidar el contexto de la conversación para un traspaso ordenado |
| 22 | Derivar automáticamente agente humano | Deriva la conversación automáticamente hacia un agente humano | Transferencia a agente humano | Escalar el caso cuando no fue posible identificar al cliente tras los reintentos |
| 23 | END | Punto de cierre del flujo, ya sea por identificación exitosa o por transferencia a un humano | Fin | Finalizar el proceso de identificación |

## Notas del flujo

- El proceso combina tres fuentes de identidad: CRM (por teléfono o correo), Cognito (autenticación externa) y Xapi Profile (perfil unificado).
- El consentimiento (Opt-in) se valida y sincroniza al inicio del flujo, antes de cualquier búsqueda de identidad.
- El canal de origen (WhatsApp vs. otros) determina si se busca directamente por teléfono en el CRM o si se ofrece el link de Cognito.
- Cuando Cognito entrega el correo del cliente, este se valida contra Xapi Profile; si no hay coincidencia, se reintenta el ciclo completo.
- El proceso permite un máximo de 3 intentos antes de preparar y ejecutar la transferencia automática a un agente humano.
- El flujo cierra únicamente en dos escenarios: cliente identificado o transferencia a agente humano.

## Leyenda de etapas

| Etapa | Descripción |
|-------|-------------|
| Inicio / Fin | Puntos de entrada y salida del flujo. |
| Enrutamiento por canal | Detección del canal de origen y su impacto en la ruta de identificación. |
| Consentimiento | Validación y sincronización del Opt-in del cliente. |
| Identificación por CRM | Búsqueda del cliente por teléfono o correo directamente en el CRM. |
| Identificación vía Cognito | Autenticación externa del cliente mediante link de Cognito. |
| Identificación vía Xapi | Validación del cliente contra el perfil unificado (Xapi Profile). |
| Manejo de reintentos | Control del número de intentos del ciclo de identificación. |
| Transferencia a agente humano | Escalamiento del caso cuando la identificación automática no tiene éxito. |
| Resultado de identificación | Confirmación de que el cliente fue identificado exitosamente. |
