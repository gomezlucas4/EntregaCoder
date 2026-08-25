# Matriz de Costos y Optimización de Modelos de IA[cite: 6]
Asistente de Pedidos IA - Distribuidora de Alimentos[cite: 6]

> **Nota técnica: alias de modelo actualizado**[cite: 6]
> El nodo DeepSeek Chat Model en n8n usaba el alias `deepseek-chat`, descontinuado por DeepSeek el 24 de julio de 2026[cite: 6]. Se detectó y corrigió durante el desarrollo: el modelo configurado ahora es `deepseek-v4-flash` (confirmado en producción ver `model_name` en el output real del nodo, ejecución del 20/ago/2026)[cite: 6]. Se documenta acá como ejemplo del monitoreo de dependencias externas que forma parte de la resiliencia del sistema[cite: 6].

## 1. Modelos evaluados y por qué[cite: 6]
El sistema tiene una única tarea que consume IA de forma constante: el AI Agent que interpreta mensajes del cliente, decide qué tools llamar y redacta la respuesta[cite: 6]. Evaluamos 4 opciones reales para esa tarea[cite: 6]:

| Modelo | Input (por 1M tokens) | Output (por 1M tokens) | Fortaleza | Debilidad para este caso |
| :--- | :--- | :--- | :--- | :--- |
| **DeepSeek V4 Flash** (elegido)[cite: 6] | $0.22 (off-peak) / $0.44 (peak)[cite: 6] | $0.66 (off-peak) / $1.32 (peak)[cite: 6] | Tool-calling competente, contexto de 1M tokens, ya tenía fondos depositados[cite: 6] | Precio variable según horario (surcharge en horario pico); ecosistema/soporte más chico que OpenAI/Anthropic[cite: 6] |
| **GPT-4o-mini**[cite: 6] | $0.15[cite: 6] | $0.60[cite: 6] | El más barato del mercado en este momento, muy estable[cite: 6] | Peor comprensión de instrucciones largas en español rioplatense en pruebas informales que Claude/DeepSeek[cite: 6] |
| **Claude Haiku 4.5**[cite: 6] | $1.00[cite: 6] | $5.00[cite: 6] | Tool-calling muy confiable, buen seguimiento de reglas estrictas[cite: 6] | ~5x más caro que DeepSeek para el mismo volumen[cite: 6] |
| **Claude Sonnet 5**[cite: 6] | $2.00 (precio de lanzamiento, vence 31/ago/2026 → pasa a $3.00)[cite: 6] | $10.00 ($15.00 desde sept.)[cite: 6] | El más sólido para razonamiento complejo[cite: 6] | Injustificado para un agente de atención al cliente de tareas simples; ~10x más caro que DeepSeek[cite: 6] |

*Precios verificados: DeepSeek (deepseek.ai, aipricing.guru - 19-20/ago/2026), OpenAI (openrouter.ai, pricepertoken.com), Anthropic (platform.claude.com/docs - pricing oficial)[cite: 6].*

## 2. Cálculo de costo mensual estimado (con datos reales de producción)[cite: 6]
Una ejecución completa del AI Agent (turno con búsqueda de stock + identificación de cliente + registro de pedido) consumió[cite: 6]:
* 3.356 tokens de input (system prompt + memoria de la conversación + definiciones de las 4 tools + mensaje del cliente)[cite: 6].
* 6.395 tokens de output (esto incluye el razonamiento y las llamadas a tools intermedias del agente)[cite: 6].

Con 4.000 turnos/mes, el consumo estimado es[cite: 6]:
* Total input mensual: 4.000 x 3.356 ≈ 13,4M tokens[cite: 6].
* Total output mensual: 4.000 x 6.395 ≈ 25,6M tokens[cite: 6].

| Modelo | Costo input | Costo output | Total mensual estimado |
| :--- | :--- | :--- | :--- |
| **GPT-4o-mini**[cite: 6] | $2.01[cite: 6] | $15.35[cite: 6] | $17.36[cite: 6] |
| **DeepSeek V4 Flash (off-peak)**[cite: 6] | $2.95[cite: 6] | $16.88[cite: 6] | $19.83[cite: 6] |
| **DeepSeek V4 Flash (peak)**[cite: 6] | $5.91[cite: 6] | $33.77[cite: 6] | $39.68[cite: 6] |
| **Claude Haiku 4.5**[cite: 6] | $13.42[cite: 6] | $127.90[cite: 6] | $141.32[cite: 6] |
| **Claude Sonnet 5** (precio actual)[cite: 6] | $26.84[cite: 6] | $255.80[cite: 6] | $282.64[cite: 6] |

**Conclusión honesta:** Con datos reales, GPT-4o-mini queda ~$2.5/mes más barato que DeepSeek[cite: 6]. Elegimos igual DeepSeek V4 Flash porque: (a) ya había fondos depositados, (b) el volumen de la entrega final es bajo, y (c) en las pruebas reales del flujo, DeepSeek siguió bien las instrucciones de tool-calling y la regla anti-alucinación[cite: 6].

## 3. Matriz de asignación de modelo por tipo de tarea[cite: 6]

| Tarea | Modelo recomendado | Justificación |
| :--- | :--- | :--- |
| **Conversación con el cliente + tool-calling (implementado)**[cite: 6] | DeepSeek V4 Flash[cite: 6] | Balance costo/calidad correcto para diálogo simple con 4 tools[cite: 6]. |
| **Clasificación de urgencia de errores en Logs_Error (mejora futura)**[cite: 6] | GPT-4o-mini[cite: 6] | Tarea de clasificación de una sola etiqueta, alto volumen, no requiere razonamiento[cite: 6]. |
| **Interpretación de documentos complejos (a futuro)**[cite: 6] | Claude Sonnet 5[cite: 6] | Lectura densa y estructurada de documentos reales justifica el costo mayor[cite: 6]. |
| **Generación de reporte mensual de ventas (no interactivo)**[cite: 6] | Batch API (Claude o OpenAI, 50% de descuento)[cite: 6] | No es una tarea en tiempo real; procesarla en modo batch corta el costo a la mitad[cite: 6]. |

## 4. Palancas de optimización aplicadas[cite: 6]
* **Prompt acotado:** El System Message del agente se mantiene enfocado en las 3 responsabilidades del bot[cite: 6].
* **Memoria con ventana limitada:** `contextWindowLength:12` evita que la conversación acumule tokens indefinidamente[cite: 6].
* **Salida estructurada (Output Parser):** Evita que el modelo divague en texto libre extenso[cite: 6].
* **Filtro anti-loop:** Evita ejecuciones duplicadas por mensajes repetidos del mismo cliente[cite: 6].
