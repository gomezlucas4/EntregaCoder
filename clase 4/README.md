# Checkpoint 4 — Integraciones Avanzadas e Interconexión de Sistemas

**Curso:** AI Automation — Coderhouse  
**Alumno:** Lucas Nicolas Gomez  
**Actividad:** Sincronización del Cerebro Agéntico con Ecosistemas de Negocio

## Descripción

Automatización desarrollada en **n8n** para gestionar consultas comerciales recibidas por Gmail, integrando inteligencia artificial, HubSpot CRM y Slack.

El workflow procesa los correos entrantes, extrae información del cliente mediante IA, crea o actualiza contactos en HubSpot, genera borradores de respuesta en Gmail y notifica al equipo de operaciones a través de Slack.

## Tecnologías utilizadas

| Herramienta | Función |
|---|---|
| n8n | Orquestación de la automatización |
| Gmail | Recepción de consultas y creación de borradores |
| HubSpot CRM | Búsqueda, creación y actualización de contactos |
| Groq | Ejecución del modelo de IA para extracción de datos |
| Slack | Notificaciones al equipo de operaciones |

## Funcionamiento

1. **Gmail Trigger:** Detecta correos entrantes.
2. **Filtro anti auto-reply:** Descarta respuestas automáticas y remitentes `no-reply`.
3. **Limpieza y validación:** Conserva los datos necesarios y verifica el correo del remitente.
4. **Extracción con IA:** Identifica nombre, apellido, tipo de consulta, productos y resumen.
5. **Búsqueda en HubSpot:** Comprueba si el contacto ya existe para evitar duplicados.
6. **Creación o actualización:** Registra la información correspondiente en el CRM.
7. **Gmail Create Draft:** Prepara una respuesta para revisión humana, sin enviarla automáticamente.
8. **Limpieza para Slack:** Reduce el payload a los campos necesarios para la notificación.
9. **Slack:** Envía un resumen de la consulta al canal `#canal-ejemplo`.

## Controles implementados

- **Prevención de bucles:** Filtro IF para correos automáticos.
- **Prevención de duplicados:** Búsqueda previa de contactos en HubSpot.
- **Validación de entrada:** Control del formato del correo electrónico.
- **Human-in-the-loop (HITL):** Borradores de Gmail sujetos a revisión humana.
- **Optimización de payloads:** Uso de nodos Set para evitar transmitir datos innecesarios.
- **Autenticación segura:** Conexiones a servicios externos mediante credenciales gestionadas en n8n.

## Pruebas realizadas

Se verificó mediante ejecuciones manuales:

- Recepción y procesamiento de correos en Gmail.
- Extracción estructurada de información mediante IA.
- Creación y actualización de contactos en HubSpot.
- Generación de borradores en Gmail.
- Envío exitoso de notificaciones a Slack.

## Instalación y uso

1. Descargar `checkpoint4_lucas_nicolas_gomez.json`.
2. Importar el archivo en n8n.
3. Configurar las credenciales de Gmail, HubSpot, Groq y Slack.
4. Seleccionar el canal de Slack correspondiente.
5. Ejecutar una prueba con un correo entrante.

> **Importante:** El workflow exportado no incluye las credenciales necesarias para conectarse a los servicios externos. Cada usuario debe configurar sus propias conexiones.
