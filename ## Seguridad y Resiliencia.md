# Seguridad y Resiliencia del Sistema[cite: 7]
Asistente de Pedidos IA Distribuidora de Alimentos[cite: 7]

## 1. Minimización de datos[cite: 7]
El principio general aplicado: el LLM nunca ve más información de la que necesita para resolver el turno actual[cite: 7].

| Componente | Lo que se envía | Lo que NO se envía |
| :--- | :--- | :--- |
| **Tool "Consultar Stock y Precio"**[cite: 7] | Solo los registros de Productos que matchean la búsqueda puntual[cite: 7] | El resto del catálogo completo[cite: 7] |
| **Tool "Buscar Cliente"**[cite: 7] | Solo el registro del cliente que coincide por sessionId o email[cite: 7] | La tabla completa de Clientes (con datos de otros clientes)[cite: 7] |
| **Memoria de conversación**[cite: 7] | Los últimos 12 mensajes de la sesión actual[cite: 7] | Historial completo indefinido, ni conversaciones de otras sesiones[cite: 7] |
| **Nodo "Crear pedido en Airtable"**[cite: 7] | Solo los campos del pedido puntual[cite: 7] | Ningún dato adicional del cliente más allá de lo que el pedido necesita[cite: 7] |
| **Credenciales**[cite: 7] | -[cite: 7] | Nunca viajan en el prompt ni en los mensajes al modelo[cite: 7] |

## 2. Rutas de error (Error Handling)[cite: 7]

**2.1 Workflow dedicado de errores**[cite: 7]
Existe un workflow separado ("Distribuidora - Error Handler") disparado por un Error Trigger nativo de n8n[cite: 7]. Cuando cualquier nodo del flujo principal falla, este workflow[cite: 7]:
1. Arma un registro con fecha/hora, nombre del nodo que falló, tipo de error y resumen del payload[cite: 7].
2. Lo guarda en la tabla Logs_Error de Airtable[cite: 7].
3. Manda una alerta por Telegram al administrador[cite: 7].

**2.2 Filtro anti-loop**[cite: 7]
El nodo "Filtro Anti-Loop (Debounce)" usa `$getWorkflowStaticData('node')` para recordar el último timestamp[cite: 7]. Si el mismo cliente manda un mensaje a menos de 3 segundos del anterior, el mensaje se descarta para evitar bucles infinitos[cite: 7].

**2.3 Comparación de tipos correcta**[cite: 7]
Los nodos IF del flujo usan validación estricta de tipo booleano (`is true` / `is false`), evitando fallos silenciosos por errores de tipo[cite: 7].

**2.4 Bugs reales encontrados y corregidos**[cite: 7]
* **Fórmulas de Airtable sin comillas:** Se corrigió envolviendo el valor `$fromAI()` entre comillas dobles[cite: 7].
* **Alucinación del agente:** Se agregó una regla explícita al System Message indicando que el agente no puede confirmar un registro sin haber llamado a la tool[cite: 7].
* **Campos de tipo incorrecto:** Se pasó la creación del pedido a un nodo determinístico para evitar rechazos en Airtable[cite: 7].
* **Timeout de infraestructura:** Las respuestas lentas generaban error 524 en Cloudflare[cite: 7].

**2.5 Resultados reales del "camino infeliz"**[cite: 7]
Se probó enviar un mensaje de compra sin datos del cliente. Gmail fallaba y rompía la ejecución[cite: 7]. Se solucionó agregando el nodo IF "¿Tiene mail?" con la condición `{{!! ($json.email_cliente && $json.email_cliente.trim().length > 0) }}`[cite: 7].

## 3. Puntos de Human-in-the-Loop (HITL)[cite: 7]

**3.1 Dónde se activa**[cite: 7]
El sistema nunca confirma un pedido de forma autónoma. Se marca `requiere_aprobacion: true` en la salida y el flujo se desvía obligatoriamente hacia el nodo de aprobación humana[cite: 7].

**3.2 Cómo funciona**[cite: 7]
Se usa el nodo nativo de Telegram "Send and Wait for Response", que envía un resumen con botones y pausa la ejecución del workflow hasta que el humano responda[cite: 7].

**3.3 Qué pasa según la decisión**[cite: 7]
* **Aprobado:** El Estado en Airtable pasa a "Aprobado por Humano" y se envía Email de confirmación[cite: 7].
* **Rechazado:** El Estado pasa a "Rechazado" y se envía un Email informando que el pedido necesita ajustes[cite: 7].

## 4. Prompt dinámico (variables del sistema)[cite: 7]
El System Message no está hardcodeado; usa expresiones de n8n para inyectar datos de contexto en tiempo real[cite: 7].
