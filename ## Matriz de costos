Matriz de Costos y Optimización de Modelos de IA
Asistente de Pedidos IA — Distribuidora de Alimentos
✅ Nota técnica: alias de modelo actualizado
El nodo DeepSeek Chat Model en n8n usaba el alias deepseek-chat, descontinuado por DeepSeek el 24 de julio de 2026. Se detectó y corrigió durante el desarrollo: el modelo configurado ahora es deepseek-v4-flash (confirmado en producción — ver model_name en el output real del nodo, ejecución del 20/ago/2026). Se documenta acá como ejemplo del monitoreo de dependencias externas que forma parte de la resiliencia del sistema (ver documento de Seguridad y Resiliencia).
1. Modelos evaluados y por qué
El sistema tiene una única tarea que consume IA de forma constante: el AI Agent que interpreta mensajes del cliente, decide qué tools llamar (consultar stock, identificar cliente, armar el pedido) y redacta la respuesta. Evaluamos 4 opciones reales para esa tarea:
Modelo
Input (por 1M tokens)
Output (por 1M tokens)
Fortaleza
Debilidad para este caso
DeepSeek V4 Flash (elegido)
$0.22 (off-peak) / $0.44 (peak)
$0.66 (off-peak) / $1.32 (peak)
Tool-calling competente, contexto de 1M tokens, ya tenía fondos depositados
Precio variable según horario (surcharge en horario pico); ecosistema/soporte más chico que OpenAI/Anthropic
GPT-4o-mini
$0.15
$0.60
El más barato del mercado en este momento, muy estable
Peor comprensión de instrucciones largas en español rioplatense que Claude/DeepSeek en pruebas informales
Modelo Input (por 1M
tokens)
Output (por 1M
tokens) Fortaleza Debilidad para este
caso
Claude Haiku 4.5
$1.00
$5.00
Tool-calling muy confiable, buen seguimiento de reglas estrictas (justo lo que necesitábamos para la regla anti-alucinación)
~5x más caro que DeepSeek para el mismo volumen
Claude Sonnet 5
$2.00 (precio de lanzamiento, vence 31/ago/2026 → pasa a $3.00)
$10.00 (→ $15.00 desde sept.)
El más sólido para razonamiento complejo
Injustificado para un agente de atención al cliente de tareas simples; ~10x más caro que DeepSeek
Precios verificados: DeepSeek (deepseek.ai, aipricing.guru — 19-20/ago/2026), OpenAI (openrouter.ai, pricepertoken.com), Anthropic (platform.claude.com/docs — pricing oficial).
2. Cálculo de costo mensual estimado (con datos reales de producción)
En vez de estimar a ojo, usamos una medición real tomada del sistema en funcionamiento: una ejecución completa del AI Agent (turno con búsqueda de stock + identificación de cliente + registro de pedido) consumió:
● 3.356 tokens de input (system prompt + memoria de la conversación + definiciones de las 4 tools + mensaje del cliente)
● 6.395 tokens de output (esto incluye el razonamiento y las llamadas a tools intermedias del agente, no solo el texto final que ve el cliente — un turno agentic con tool-calling siempre consume más output que un chat de una sola respuesta, porque el modelo "piensa" y arma cada llamada a herramienta como parte del output facturable)
Con el mismo supuesto de volumen que antes (500 conversaciones/mes, 8 turnos promedio → 4.000 turnos/mes):
● Total input mensual: 4.000 × 3.356 ≈ 13,4M tokens
● Total output mensual: 4.000 × 6.395 ≈ 25,6M tokens
Modelo
Costo input
Costo output
Total mensual estimado
GPT-4o-mini
$2.01
$15.35
$17.36
DeepSeek V4 Flash (off-peak)
$2.95
$16.88
$19.83
DeepSeek V4 Flash (peak)
$5.91
$33.77
$39.68
Claude Haiku 4.5
$13.42
$127.90
$141.32
Claude Sonnet 5 (precio actual)
$26.84
$255.80
$282.64
Lectura del dato real: el output domina el costo total en un flujo agentic (por lo explicado arriba), así que el precio por token de output pesa mucho más que el de input a la hora de elegir modelo — esto no se ve si solo se mira el precio de "input" como suele destacarse en el marketing de cada proveedor.
Conclusión honesta: con datos reales, GPT-4o-mini queda ~$2.5/mes más barato que DeepSeek (una diferencia mayor en términos absolutos que la que se veía con la estimación a ojo, aunque sigue siendo un monto bajo en este volumen). Elegimos igual DeepSeek V4 Flash porque: (a) ya había fondos depositados en la cuenta antes de este cambio de precios, (b) el volumen de la entrega final es bajo (menos de USD 20/mes en cualquiera de las dos opciones), y (c) en las pruebas reales del flujo, DeepSeek siguió bien las instrucciones de tool-calling y la regla anti-alucinación una vez reforzado el prompt. Si este proyecto pasara a producción real con mayor volumen, GPT-4o-mini sería la alternativa a evaluar primero por precio equivalente y mayor estabilidad de tarifa (sin recargo por horario) — y valdría la pena además revisar si se puede acortar el ciclo de tool-calling para bajar el output promedio por turno, ya que es la variable que más pesa en el costo real.
3. Matriz de asignación de modelo por tipo de tarea
No todas las tareas de IA de un sistema como este necesitan el mismo modelo. Esta es la asignación recomendada si el proyecto creciera:
Tarea
Modelo recomendado
Justificación
Conversación con el cliente + tool-calling (implementado)
DeepSeek V4 Flash
Balance costo/calidad correcto para diálogo simple con 4 tools
Clasificación de urgencia de errores en Logs_Error (mejora futura)
GPT-4o-mini
Tarea de clasificación de una sola etiqueta, alto volumen, no requiere razonamiento — el modelo más barato alcanza y sobra
Interpretación de documentos complejos (ej. facturas de proveedores escaneadas, a futuro)
Claude Sonnet 5
Lectura densa y estructurada de documentos reales justifica el costo mayor, con bajo volumen (no es por mensaje de chat)
Generación de reporte mensual de ventas y tendencias (no interactivo, se puede correr de madrugada)
Batch API (Claude o OpenAI, 50% de descuento)
No es una tarea en tiempo real — procesarla en modo batch corta el costo a la mitad sin ningún impacto en la experiencia del cliente
4. Palancas de optimización aplicadas en este proyecto
● Prompt acotado: el System Message del agente se mantiene enfocado en las 3 responsabilidades del bot (stock, cliente, pedido) en vez de instrucciones genéricas largas, para no inflar el costo de input en cada turno.
● Memoria con ventana limitada: contextWindowLength:12 evita que la conversación acumule tokens de forma indefinida turno a turno.
● Salida estructurada (Output Parser): evita que el modelo divague en texto libre extenso; la respuesta siempre tiene un tamaño acotado y predecible.
● Filtro anti-loop: evita ejecuciones (y por lo tanto tokens facturados) duplicadas por mensajes repetidos del mismo cliente en pocos segundos.
