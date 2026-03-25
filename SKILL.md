---
name: token-budget-advisor
description: >
  Analiza prompts y ofrece opciones de profundidad/tokens ANTES de responder.
  Usa esta skill cuando el usuario quiera controlar el consumo de tokens, ajustar la
  profundidad de respuesta, elegir entre respuestas cortas/largas, u optimizar su prompt.
  Se activa con: "tokens", "presupuesto de tokens", "profundidad", "consumo",
  "respuesta corta vs larga", "token budget", "cuántos tokens", "ahorrar tokens",
  "responde al 50%", "dame la versión corta", "quiero controlar cuánto usas",
  "ajusta tu respuesta", o cualquier variante. Si el usuario quiere controlar extensión,
  detalle o profundidad — incluso sin mencionar tokens explícitamente — esta skill aplica.
---

# Token Budget Advisor

Skill que intercepta el flujo de respuesta para ofrecer al usuario una elección
informada sobre cuánta profundidad/tokens quiere consumir, ANTES de responder.

## Flujo de trabajo

### Paso 1: Analizar el prompt

Ejecuta el estimador sobre el prompt del usuario. El script está en `scripts/token_estimator.py`
dentro del directorio de esta skill.

```bash
python3 <SKILL_DIR>/scripts/token_estimator.py --text "PROMPT_DEL_USUARIO"
```

Para prompts largos o con comillas, usa un archivo temporal:

```bash
cat > /tmp/_tba_prompt.txt << 'PROMPT_EOF'
PROMPT_DEL_USUARIO
PROMPT_EOF
python3 <SKILL_DIR>/scripts/token_estimator.py --file /tmp/_tba_prompt.txt
```

Reemplaza `<SKILL_DIR>` con la ruta real donde está instalada esta skill (normalmente
`.claude/skills/token-budget-advisor` o la ruta que aparezca en tu configuración).

El script devuelve JSON con: `input_tokens`, `detected_language`, `detected_type`,
`complexity`, `response_estimates` (por nivel 25/50/75/100), y `total_estimates`.

### Paso 2: Presentar opciones al usuario

Presenta la información de forma clara ANTES de responder al prompt real.
Consulta `references/calibration.md` para entender qué incluye y omite cada nivel.

Formato recomendado:

```
📊 Análisis de tu prompt
━━━━━━━━━━━━━━━━━━━━━━━━
📝 Input: ~X tokens | 🔍 Tipo: [tipo] | 📏 Complejidad: [nivel]

🎯 Elige tu nivel de profundidad:

🟢 Esencial (25%)  → ~Y tokens
   Respuesta directa al grano. Para consultas rápidas.

🟡 Moderado (50%)  → ~Z tokens
   Contexto + 1 ejemplo. Para entender y actuar.

🟠 Detallado (75%) → ~W tokens
   Completo con alternativas. Para aprender a fondo.

🔴 Exhaustivo (100%) → ~V tokens
   Análisis total. Para investigación y documentación.

⚠️ Estimación heurística (~85-90% precisión, ±15%).
```

### Paso 3: Esperar la elección

Pregunta al usuario qué nivel prefiere. Si usa Claude Code en terminal, presenta las
opciones como texto claro y espera su respuesta.

### Paso 4: Responder según el nivel elegido

| Nivel | Extensión | Qué incluir |
|-------|-----------|-------------|
| 25% Esencial | 2-4 frases máximo | Solo la respuesta directa. Sin preámbulos. |
| 50% Moderado | 1-3 párrafos | Respuesta + contexto mínimo + 1 ejemplo si aplica. |
| 75% Detallado | Respuesta estructurada | Múltiples ejemplos, pros/contras, alternativas. |
| 100% Exhaustivo | Sin restricción | Todo: análisis completo, código completo, todas las perspectivas. |

## Atajos

Si el usuario ya indica un nivel directamente, no preguntes — responde al nivel indicado:

- "al 25%" / "versión corta" / "resumen" / "tldr" → 25%
- "al 50%" / "moderado" / "normal" → 50%
- "al 75%" / "detallado" / "completo" → 75%
- "al 100%" / "exhaustivo" / "todo" / "sin límite" → 100%

Si el usuario estableció un nivel en un mensaje anterior de la misma sesión,
mantén ese nivel para las siguientes respuestas sin volver a preguntar (a menos
que pida cambiarlo).
