# 📦 Ecosistema IA Autónomo - Distribuidora Inteligente

Un sistema integral de gestión de pedidos y atención al cliente impulsado por Inteligencia Artificial, construido con herramientas No-Code/Low-Code. Este proyecto demuestra una arquitectura de automatización resiliente, capaz de procesar intenciones, consultar bases de datos en tiempo real y ejecutar validaciones humanas antes de acciones críticas.

## 🚀 Características Principales

*   **Atención al Cliente Inteligente:** Un Agente de IA que entiende intenciones (consultas de stock, registro de clientes o armado de pedidos) y responde con contexto y memoria por sesión.
*   **Triaje y Enrutamiento Automático:** Clasificación dinámica de mensajes para derivarlos al flujo de datos correspondiente.
*   **Orquestación y "Semáforo" Humano (HITL):** Aprobación mandatoria de pedidos vía Telegram antes del registro definitivo en la base de datos y envío de notificaciones por email.
*   **Resiliencia y Monitoreo:** Filtro anti-loop de mensajes, subflujo diario para alertas de stock bajo y un gestor de errores global (Error Handler) que captura fallos de API y los persiste.

## 🛠️ Stack Tecnológico

El ecosistema integra las siguientes tecnologías y herramientas:

*   **Orquestador:** [n8n](https://n8n.io/) (Flujo principal y manejo de lógica).
*   **Motor de Inteligencia Artificial:** [DeepSeek V4 Flash](https://www.deepseek.com/) (Vía n8n LangChain Node).
*   **Cerebro de Datos (Base de Datos):** [Airtable](https://airtable.com/) (Tablas relacionales: Productos, Clientes, Pedidos, Logs_Error).
*   **Canales de Intervención y Salida:** 
    *   **Telegram API:** Aprobación humana (HITL) mediante notificaciones con botones nativos.
    *   **Gmail:** Confirmación de pedidos o notificación de ajustes a los clientes.

## 🏗️ Arquitectura del Sistema

El proyecto se divide en dos Workflows principales que interactúan con la Base de Datos centralizada.

1.  **Workflow Principal (Distribuidora):**
    *   **Trigger:** Recepción de mensajes en el chat.
    *   **Anti-Loop:** Sistema de *debounce* por `sessionId`.
    *   **Agente IA:** Consulta stock, busca o crea clientes mediante *Tools* conectadas a Airtable. Genera un JSON de salida estricto.
    *   **Routing & HITL:** Si el pedido requiere aprobación, se envía una alerta a Telegram con opciones interactivas. Si se aprueba, se actualiza el registro en Airtable y se notifica vía Gmail.

2.  **Workflow de Resiliencia (Error Handler):**
    *   Workflow independiente configurado para atrapar fallos del flujo principal.
    *   Registra el error, el nodo que falló y el payload en la tabla `Logs_Error` de Airtable.
    *   Envía una alerta crítica de sistema a Telegram.

## 📂 Contenido de este Repositorio

*   `Workflow_Distribuidora.json`: Exportación completa del blueprint del flujo principal de n8n.
*   `Distribuidora_Error_Handler.json`: Exportación del flujo encargado de capturar y registrar fallos globales.
*   `Diagrama_Arquitectura_Logica.pdf`: Mapeo visual y estratégico de todo el ecosistema (Triggers, Ruteo, APIs y Destinos).
*   `Manual_Operativo_Datos.md`: Documentación del esquema relacional de Airtable y las estructuras JSON de transferencia.
*   `Matriz_de_Costos.md`: Análisis comparativo y justificación económica de la selección de modelos LLM.
*   `Seguridad_y_Resiliencia.md`: Documento detallando la aplicación de políticas de minimización de datos y el análisis de casos de fallo controlados.
*   `/evidencias`: Carpeta con capturas de pantalla de los flujos operando en vivo.

## 💡 Cómo replicar este proyecto

1.  Importar los archivos `.json` en tu propia instancia de n8n.
2.  Crear una base en Airtable siguiendo el esquema detallado en `Manual_Operativo_Datos.md` e insertar tu API Key (Personal Access Token) en n8n.
3.  Configurar las credenciales para la API de tu LLM preferido (OpenAI/Anthropic/DeepSeek), Gmail (OAuth2) y el Bot de Telegram.
4.  Establecer el *Error Handler Workflow* como el mecanismo de captura de errores predeterminado en las opciones (Settings) del workflow principal.
