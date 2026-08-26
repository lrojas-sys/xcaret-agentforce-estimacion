# PROCESO – SEGUIMIENTO DE CASOS O TRÁMITES (Documentos para presentar / Seguimiento de casos)

**Objetivo general:** Permitir al cliente consultar el avance de su caso o solicitud y conocer el estatus actual sin necesidad de llamar al Contact Center.

## Tabla de acciones del flujo

| # | HU | Acción | Descripción | Etapa del flujo | Objetivo |
|---|----|--------|--------------|------------------|----------|
| 1 | Req4-HU38 | Detectar intención de seguimiento de caso o trámite | El agente identifica que el cliente desea consultar el avance de un caso o solicitud (seguimiento de caso, consulta de estatus, revisión de trámite, avance de solicitud, documentos pendientes) | Detección / Identificación | Reconocer automáticamente la intención del cliente para iniciar el flujo de seguimiento correcto |
| 2 | Req4-HU39 | Solicitar número oficial de caso para identificación | El agente solicita el número oficial de caso como identificador; valida el formato y lo almacena en sesión; si no se proporciona o no es válido, activa transferencia a un humano | Consulta / Búsqueda | Obtener un identificador confiable para localizar el caso correcto |
| 3 | Req4-HU40 | Consultar expediente en objeto Case | El sistema consulta el expediente en Salesforce (objeto Case) usando el número de caso o el correo del cliente; incluye búsqueda exacta por número, por correo y recuperación de casos abiertos o con actividad pendiente | Consulta / Búsqueda | Recuperar la información del caso desde el sistema de registro oficial |
| 4 | Req4-HU41 | Informar estatus actual del caso al cliente | El agente informa el estatus vigente y los detalles relevantes: estatus actual, fecha de última actualización, próximos pasos/acciones pendientes y documentos pendientes (si aplica) | Información al cliente | Mantener al cliente informado del avance de su caso sin necesidad de contactar al Contact Center |
| 5 | Req4-HU68 | Mantener contexto conversacional sobre la reserva consultada | Se mantiene el contexto de la conversación y la información del caso en sesión para continuidad y consultas posteriores | Información al cliente | Evitar que el cliente repita información en consultas subsecuentes dentro de la misma sesión |
| 6 | Req4-HU69 | Manejo de reserva no localizada o sin pickup asociado | No se encontró el caso con el número o correo proporcionado; se informa al cliente y se ofrecen opciones (verificar nuevamente datos, buscar con otros datos, orientación general, transferencia a un ejecutivo); se mantiene el contexto y la consulta queda registrada | Manejo de excepciones | Dar alternativas al cliente cuando no se puede localizar el caso, evitando un callejón sin salida |
| 7 | Req4-HU72 | Transferencia contextual a ejecutivo humano cuando el cliente requiere apoyo adicional | El cliente solicita hablar con el ejecutivo o requiere apoyo adicional | Transferencia a humano | Detectar la necesidad de escalamiento a un agente humano |
| 8 | Req4-HU70 | Crear caso únicamente cuando exista transferencia o gestión humana solicitada | Se transfiere el caso con el contexto completo de la conversación | Transferencia a humano | Asegurar que el ejecutivo humano reciba toda la información necesaria sin que el cliente repita datos, creando el caso solo cuando realmente se requiere |
| 9 | Req4-HU71 | Registrar historial de consulta de caso | Se registra el historial de consulta de caso, se almacena información relevante (número de caso, estatus, resultados, interacciones y canal) y se asegura la trazabilidad completa de la consulta y acciones realizadas | Trazabilidad / Registro | Permitir auditoría, mejora continua y reporting del proceso de seguimiento de casos |

## Notas del flujo

- La información mostrada corresponde únicamente al caso consultado.
- No se envían comprobantes ni documentos por este canal.
- El agente nunca toma acciones de cambio o cancelación; solo informa.
- Se respeta la privacidad y seguridad de la información del cliente.

## Leyenda de etapas

| Etapa | Descripción |
|-------|-------------|
| Detección / Identificación | Reconocimiento de la intención del cliente. |
| Consulta / Búsqueda | Obtención del identificador y búsqueda del expediente en Salesforce. |
| Información al cliente | Comunicación del estatus y mantenimiento del contexto conversacional. |
| Manejo de excepciones | Gestión de casos no localizados u otras desviaciones del flujo. |
| Transferencia a humano | Escalamiento a un ejecutivo cuando el cliente lo solicita o el caso lo requiere. |
| Trazabilidad / Registro | Registro histórico de la consulta para auditoría y mejora continua. |

## Alineación con el diagrama de Identificación Cognito → XapiProfile

Este flujo identifica el **caso** (por número o correo), no a la persona. No ejecuta su propio ciclo de identificación de cliente (Cognito/CRM/Xapi) y no debería duplicarlo.

- **Req4-HU39 / Req4-HU40:** el uso de "el correo del cliente" como criterio de búsqueda alterno asume que el cliente **ya fue identificado previamente** por el flujo `Proceso - Identificación del Cliente (Autenticación y Continuidad)` (que a su vez sigue el diagrama Cognito → XapiProfile), y que ese correo/teléfono ya está disponible en el contexto de sesión. Este documento no declaraba esa dependencia explícitamente — se deja documentada aquí para que quede trazable entre ambos flujos.
- No se identificó ninguna otra acción de este flujo que debiera seguir el diagrama de identificación de cliente.

## Preguntas abiertas / a validar

- **Req4-HU39 (solicitar número de caso):** ¿la validación del número de caso en este paso se limita al formato (estructura/longitud) en memoria de sesión, o también valida su existencia contra Salesforce antes de continuar? Esto determina si la acción debe contabilizarse aquí o si el costo real ocurre únicamente en la consulta del expediente (Req4-HU40). *Supuesto usado en la tabla: solo pide el dato y valida formato, sin costo.*
- **Req4-HU69 (caso no localizado):** el flujo indica que "la consulta queda registrada" cuando no se encuentra el caso — ¿esto implica una escritura inmediata a Salesforce (acción con costo) independiente del registro de trazabilidad final (Req4-HU71), o es el mismo evento reportado dos veces en la documentación? *No se resolvió; queda marcado como "Por confirmar" en la tabla con 0 créditos como supuesto conservador.*
- **Req4-HU69 vs Req4-HU71 (posible duplicidad de registro):** si el registro de "consulta no localizada" y el "historial de consulta de caso" son eventos distintos que ambos escriben a Salesforce, el estimado actual subestimaría el consumo real (faltaría una acción adicional de ~20 Flex Credits por cada conversación con caso no localizado). Si son el mismo evento, el estimado actual (sin costo, log de fondo) es correcto tal cual está. Vale la pena confirmar con el equipo de diseño del flujo antes de usar esta cifra en una propuesta formal.

## Sugerencias de mejora del flujo

- **Req4-HU72 vs Req4-HU70:** ambas HUs describen la misma transferencia a ejecutivo desde ángulos distintos (HU72 = el cliente solicita/necesita apoyo; HU70 = se crea el caso con el contexto). Se sugiere separar explícitamente en la documentación cuál acción es solo detección de intención (sin costo) y cuál es la que efectivamente crea el registro en Salesforce (con costo), para evitar ambigüedad sobre dónde se contabiliza el consumo de créditos.

## Estimación de consumo de créditos

| # | HU | Acción | Etapa | ¿Consume créditos? | Créditos Agentforce (Flex Credits) | Créditos Data Cloud | Justificación |
|---|----|--------|-------|---------------------|--------------------------------------|----------------------|----------------|
| 1 | Req4-HU38 | Detectar intención de seguimiento de caso o trámite | Detección / Identificación | No | 0 | 0 | Es reconocimiento de intención del cliente; lógica conversacional sin invocar acción ni consulta. |
| 2 | Req4-HU39 | Solicitar número oficial de caso para identificación | Consulta / Búsqueda | No | 0 | 0 | Se asume que solo solicita el dato y valida el formato en memoria de sesión, sin consultar Salesforce todavía (ver pregunta abierta). |
| 3 | Req4-HU40 | Consultar expediente en objeto Case | Consulta / Búsqueda | Sí | 20 | 0 | Ejecuta una acción estándar de Agentforce que consulta directo el objeto Case en Core CRM (confirmado); al no pasar por Data Cloud no genera cargo adicional. |
| 4 | Req4-HU41 | Informar estatus actual del caso al cliente | Información al cliente | No | 0 | 0 | Reutiliza los datos ya obtenidos en la consulta del expediente (confirmado); es una respuesta conversacional, no una acción nueva. |
| 5 | Req4-HU68 | Mantener contexto conversacional sobre el caso consultado | Información al cliente | No | 0 | 0 | Es memoria de sesión/contexto conversacional; no invoca ninguna acción ni consulta adicional. |
| 6 | Req4-HU69 | Manejo de caso no localizado | Manejo de excepciones | Por confirmar | 0 (supuesto) | 0 | Depende de si "la consulta queda registrada" es una escritura separada a Salesforce o el mismo evento del historial final (ver pregunta abierta); se deja 0 como supuesto conservador. |
| 7 | Req4-HU72 | Transferencia contextual a ejecutivo humano cuando el cliente requiere apoyo | Transferencia a humano | No | 0 | 0 | Es la detección de la intención/necesidad de escalar; la acción con costo real ocurre al crear el caso (ver siguiente fila y sugerencia de mejora). |
| 8 | Req4-HU70 | Crear caso únicamente cuando exista transferencia o gestión humana solicitada | Transferencia a humano | Sí | 20 | 0 | Crea un registro de Case en Salesforce con el contexto de la conversación mediante acción estándar de agente. |
| 9 | Req4-HU71 | Registrar historial de consulta de caso | Trazabilidad / Registro | No | 0 | 0 | Es logging/auditoría de fondo (background automation), sin costo según el baseline; posible solape con el registro mencionado en el manejo de caso no localizado (ver pregunta abierta). |

**Totales estimados por escenario:**
- Consulta de estatus exitosa (sin escalamiento): 1 acción con costo (consultar expediente) → **20 Flex Credits de Agentforce**, 0 de Data Cloud.
- Consulta que termina en transferencia a ejecutivo: se suma la creación del caso → **40 Flex Credits de Agentforce**, 0 de Data Cloud.
- Si se confirma que el manejo de caso no localizado (fila 6) implica un registro separado, sumar **+20 Flex Credits** adicionales a cualquier conversación donde el caso no se encuentre.

**Nota sobre consultas repetidas dentro de la misma sesión:** el flujo no especifica un límite a cuántos casos distintos puede consultar un cliente en una misma conversación. No se tiene un dato real de frecuencia, así que se sugiere modelar el consumo **por consulta** (20 Flex Credits cada vez que se ejecuta la consulta del expediente, fila 3) en lugar de un total fijo por conversación, usando 1 consulta como supuesto base para esta estimación y aplicando un multiplicador cuando se tenga visibilidad del promedio real de consultas por sesión.
