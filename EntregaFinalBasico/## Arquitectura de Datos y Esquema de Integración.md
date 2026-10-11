# Arquitectura de Datos y Esquema de Integración[cite: 5]

Sistema de Asistente de Pedidos IA - Distribuidora de Alimentos[cite: 5]. Este documento detalla la estructura de tablas vinculadas y el protocolo de transferencia de datos diseñado para la automatización de la gestión de pedidos mediante Inteligencia Artificial[cite: 5].

## 1. Esquema de Tablas Vinculadas[cite: 5]

La base de datos está diseñada para garantizar la integridad referencial y facilitar la trazabilidad de cada operación[cite: 5].

| Tabla | Campos Principales | Descripción y Relaciones |
| :--- | :--- | :--- |
| **Clientes**[cite: 5] | ID, Nombre, Email, Teléfono, SessionId[cite: 5] | Almacena los datos maestros de clientes[cite: 5]. El campo SessionId se utiliza para la persistencia del contexto en la IA[cite: 5]. |
| **Productos**[cite: 5] | ID, Nombre, Precio, Stock, Mínimo[cite: 5] | Catálogo de productos con gestión de inventario[cite: 5]. Incluye alertas de stock bajo basadas en el campo Minimo[cite: 5]. |
| **Pedidos**[cite: 5] | ID, Cliente_ID, Producto_ID, Cantidad, Precio Total, Estado, Fecha[cite: 5] | Registro transaccional[cite: 5]. Los campos Cliente_ID y Producto_ID establecen relaciones 1:N con sus respectivas tablas[cite: 5]. El Estado permite gestionar el flujo de aprobación humana (HITL)[cite: 5]. |
| **Logs_Error**[cite: 5] | ID_Log, Fecha_Hora, Nodo_n8n, Tipo_Error, Payload_Resumen[cite: 5] | Auditoría técnica para el manejo de excepciones y errores en los flujos de n8n[cite: 5]. |

## 2. Protocolo de Transferencia (JSON)[cite: 5]

Para la comunicación entre el Agente de IA y los nodos determinísticos de registro, se utiliza la siguiente estructura de datos[cite: 5]:

{
  "respuesta_cliente": "texto",
  "requiere_aprobacion": false,
  "resumen_pedido": "texto",
  "email_cliente": "texto",
  "id_cliente": "CLI-001 o vacio",
  "productos_solicitados": "PRD-001, PRD-004 o vacio",
  "cantidades": "2, 1 o vacio",
  "total_estimado": 0
}

**Detalle de Campos:**[cite: 5]
* **respuesta_cliente:** El mensaje final generado por la IA para el usuario[cite: 5].
* **requiere_aprobacion:** Indicador booleano que activa la ruta de "Human-in-the-Loop" si el pedido excede ciertos parámetros[cite: 5].
* **resumen_pedido:** Descripción legible de los ítems y cantidades para validación rápida[cite: 5].
* **id_cliente:** Referencia única al registro de la tabla Clientes; si está vacío, el flujo gatilla la identificación del usuario[cite: 5].
* **productos_solicitados / cantidades:** Listas separadas por comas que permiten el procesamiento masivo de ítems en el nodo de creación de pedidos[cite: 5].

## 3. Consideraciones de Seguridad y Resiliencia[cite: 5]

* **Minimización de Datos:** El LLM solo accede a los registros específicos que coinciden con la búsqueda puntual, nunca a la tabla completa[cite: 5].
* **Gestión de Errores:** Cualquier fallo en la API de IA o en los nodos de integración se registra automáticamente en la tabla Logs_Error y dispara una alerta por Telegram[cite: 5].
* **Filtro Anti-Loop:** Se implementa un mecanismo de "Debounce" para evitar ejecuciones duplicadas si el cliente envía múltiples mensajes en menos de 3 segundos[cite: 5].
