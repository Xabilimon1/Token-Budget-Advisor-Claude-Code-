# ⚖️ Token Budget Advisor

**Skill para Claude Code** que analiza tu prompt y te ofrece opciones de profundidad/tokens **antes** de que Claude responda.

```
Tú: "Explícame cómo funciona la arquitectura Transformer..."

Claude: 📊 Análisis de tu prompt
       ━━━━━━━━━━━━━━━━━━━━━━━━
       📝 Input: ~54 tokens | 🔬 Complejidad: Compleja | 🌐 Español

       🟢 Esencial (25%)  → ~430 tokens  — Solo la respuesta directa
       🟡 Moderado (50%)  → ~600 tokens  — Contexto + 1 ejemplo
       🟠 Detallado (75%) → ~780 tokens  — Completo con alternativas
       🔴 Exhaustivo (100%) → ~920 tokens — Análisis total

       ¿Qué nivel prefieres?
```

## Instalación

### Opción 1: Clonar en tu proyecto (recomendado)

```bash
# Desde la raíz de tu proyecto
git clone https://github.com/TU_USUARIO/token-budget-advisor.git .claude/skills/token-budget-advisor
```

### Opción 2: Subcarpeta manual

```bash
mkdir -p .claude/skills
cp -r token-budget-advisor/ .claude/skills/token-budget-advisor/
```

### Opción 3: Git submodule

```bash
git submodule add https://github.com/TU_USUARIO/token-budget-advisor.git .claude/skills/token-budget-advisor
```

## Estructura

```
token-budget-advisor/
├── SKILL.md                    # Instrucciones principales (lo que Claude lee)
├── scripts/
│   └── token_estimator.py      # Motor de estimación de tokens
├── references/
│   └── calibration.md          # Datos de calibración y ratios por idioma
├── examples/
│   └── sample_prompts.json     # Ejemplos de prompts con análisis esperado
├── README.md                   # Este archivo
├── LICENSE                     # MIT
└── .gitignore
```

## ¿Cómo funciona?

1. **Escribes tu prompt** normalmente en Claude Code
2. **La skill intercepta** y analiza el prompt antes de responder
3. **Te presenta 4 niveles** de profundidad con estimaciones de tokens
4. **Tú eliges** y Claude responde ajustándose a ese nivel

### Motor de estimación

El estimador usa un enfoque híbrido sin dependencias externas:

- **Textos cortos (<50 chars)**: conteo segmentado por tipo de token (palabras, números, puntuación)
- **Textos largos (≥50 chars)**: ratio calibrada por tipo de contenido detectado
- **Promedio ponderado**: entre estimación por caracteres y por palabras

Precisión medida: **~85-90%** respecto a tokenizadores reales (error promedio ~13.5%).

### Detección automática

| Característica | Valores detectados |
|---|---|
| **Idioma** | Español, Inglés, Código, Mixto |
| **Tipo de contenido** | Natural, Código, JSON, Markdown |
| **Complejidad** | Simple, Media, Media-Alta, Compleja, Creativa |

### Ratios de tokenización calibradas

| Tipo de contenido | Chars/Token |
|---|---|
| Inglés natural | ~4.0 |
| Español natural | ~3.5 |
| Código | ~3.0 |
| JSON | ~2.8 |
| Markdown | ~3.3 |

## Uso directo del estimador

El script también se puede usar standalone:

```bash
# Analizar texto directo
python3 scripts/token_estimator.py --text "Tu prompt aquí"

# Analizar desde archivo
python3 scripts/token_estimator.py --file mi_prompt.txt
```

Salida JSON:

```json
{
  "input_tokens": 54,
  "detected_language": "es",
  "detected_type": "natural",
  "complexity": "compleja",
  "response_estimates": {
    "25": 431,
    "50": 604,
    "75": 776,
    "100": 920
  }
}
```

## Limitaciones

- **Sin tokenizador real**: Usa heurísticas calibradas, no el tokenizador BPE de Claude. Precisión ~85-90%.
- **No accede a límites de sesión**: No puede saber cuántos tokens te quedan en tu plan — eso es server-side.
- **Estimación de respuesta**: Se basa en la complejidad del prompt, no en lo que Claude realmente generará.
- **Sin dependencias de red**: Funciona 100% offline, Python 3.8+ estándar.

## Licencia

MIT — úsalo, modifícalo, compártelo.
