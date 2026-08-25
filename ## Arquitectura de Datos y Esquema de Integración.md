Arquitectura de Datos y Esquema de Integración
Sistema de Asistente de Pedidos IA — Distribuidora de Alimentos
Este documento detalla la estructura de tablas vinculadas y el protocolo de transferencia de datos diseñado para la automatización de la gestión de pedidos mediante Inteligencia Artifi cial.
1. Esquema de Tablas Vinculadas
La base de datos está diseñada para garantizar la integridad referencial y facilitar la trazabilidad de cada operación.
Tabla
Campos Principales
Descripción y Relaciones
Clientes
ID, Nombre, Email, Teléfono, SessionId
Almacena los datos maestros de clientes. El campo SessionId se utiliza para la persistencia del contexto en la IA.
Productos
ID, Nombre, Precio, Stock, Mínimo
Catálogo de productos con gestión de inventario. Incluye alertas de stock bajo basadas en el campo Mínimo.
Pedidos
ID, Cliente_ID, Producto_ID, Cantidad, Precio Total, Estado, Fecha
Registro transaccional. Los campos Cliente_ID y Producto_ID establecen
Tabla Campos Principales Descripción y
Relaciones
relaciones 1:N con sus respectivas tablas. El Estado permite gestionar el fl ujo de aprobación humana (HITL).
Logs_Error
ID_Log, Fecha_Hora, Nodo_n8n, Tipo_Error, Payload_Resumen
Auditoría técnica para el manejo de excepciones y errores en los fl ujos de n8n.
2. Protocolo de Transferencia (JSON)
Para la comunicación entre el Agente de IA y los nodos determinísticos de registro, se utiliza la siguiente estructura de datos:{
"respuesta_cliente": "texto",
"requiere_aprobacion": false,
"resumen_pedido": "texto",
"email_cliente": "texto",
"id_cliente": "CLI-001 o vacio",
"productos_solicitados": "PRD-001, PRD-004 o vacio",
"cantidades": "2, 1 o vacio",
"total_estimado": 0
}
Detalle de Campos:
● respuesta_cliente: El mensaje fi nal generado por la IA para el usuario.
● requiere_aprobacion: Indicador booleano que activa la ruta de "Human-in-the-Loop" si el pedido excede ciertos parámetros.
● resumen_pedido: Descripción legible de los ítems y cantidades para validación rápida.
● id_cliente: Referencia única al registro de la tabla Clientes; si está vacío, el fl ujo gatilla la identifi cación del usuario.
● productos_solicitados / cantidades: Listas separadas por comas que permiten el procesamiento masivo de ítems en el nodo de creación de pedidos.
3. Consideraciones de Seguridad y Resiliencia
● Minimización de Datos: El LLM solo accede a los registros específi cos que coinciden con la búsqueda puntual, nunca a la tabla completa.
● Gestión de Errores: Cualquier fallo en la API de IA o en los nodos de integración se registra automáticamente en la tabla Logs_Error y dispara una alerta por Telegram.
● Filtro Anti-Loop: Se implementa un mecanismo de "Debounce" para evitar ejecuciones duplicadas si el cliente envía múltiples mensajes en menos de 3 segundos.
