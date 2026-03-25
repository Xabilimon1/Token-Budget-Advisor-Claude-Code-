# Calibración y datos de referencia

## Ratios de tokenización por tipo de contenido

Estos datos están calibrados empíricamente contra tokenizadores BPE (cl100k_base y similares).
El tokenizador de Claude es propietario pero produce resultados similares.

| Tipo | Chars/Token | Tokens/Palabra | Notas |
|------|-------------|----------------|-------|
| Inglés natural | ~4.0 | ~1.3 | Palabras comunes (≤6 chars) = 1 token |
| Español natural | ~3.5 | ~1.5 | Acentos y ñ reducen eficiencia |
| Código Python/JS | ~3.0 | — | Mucha puntuación = más tokens |
| JSON | ~2.8 | — | Corchetes, comillas, dos puntos |
| Markdown | ~3.3 | — | Formato (##, **, ```) suma tokens |

## Multiplicadores de respuesta por complejidad

Cuántas veces más grande que el input suele ser la respuesta de Claude:

| Complejidad | Multiplicador min | Multiplicador max | Ejemplo |
|-------------|-------------------|--------------------|---------|
| Simple | 3x | 8x | "¿Qué es X?", "¿Sí o no?" |
| Media | 8x | 20x | "¿Cómo funciona X?" |
| Media-Alta | 10x | 25x | Petición de código con contexto |
| Compleja | 15x | 40x | Análisis detallado, comparativas |
| Creativa | 10x | 30x | Historias, ensayos, narrativa |

## Qué incluye y omite cada nivel de profundidad

### 🟢 Esencial (25%)
- **Incluye**: Respuesta directa, conclusión clave, 1-2 frases
- **Omite**: Contexto, ejemplos, matices, alternativas, disclaimers
- **Cuándo elegirlo**: Ya conoces el tema, solo necesitas un dato o confirmación

### 🟡 Moderado (50%)
- **Incluye**: Respuesta + contexto necesario + 1 ejemplo práctico
- **Omite**: Análisis profundo, alternativas, casos borde, referencias
- **Cuándo elegirlo**: Quieres entender lo suficiente para actuar

### 🟠 Detallado (75%)
- **Incluye**: Respuesta completa + múltiples ejemplos + pros/contras + alternativas
- **Omite**: Casos extremos, referencias exhaustivas, perspectivas marginales
- **Cuándo elegirlo**: Necesitas tomar una decisión informada o aprender a fondo

### 🔴 Exhaustivo (100%)
- **Incluye**: Todo — análisis completo, todas las perspectivas, código completo, referencias
- **Omite**: Nada — máxima profundidad posible
- **Cuándo elegirlo**: Investigación, documentación, temas críticos donde no quieres que se omita nada

## Precisión del estimador

- **Error promedio**: ~13.5% en tests de calibración
- **Precisión declarada**: 85-90%
- **Varianza**: ±15% respecto a valores reales
- **Peor caso**: textos muy cortos (<10 chars) o con muchos emojis/caracteres especiales

## Limitaciones conocidas

1. Sin tokenizador real — heurísticas solamente
2. No accede a límites de sesión/plan del usuario
3. Los emojis y caracteres Unicode no-estándar pueden subrepresentarse
4. Las respuestas estimadas son aproximaciones basadas en complejidad, no predicciones exactas
