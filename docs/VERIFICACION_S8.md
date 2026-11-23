# Verificación del terreno — Sesión 8

Fecha: 2026-11-23. Solo lectura; el único archivo escrito es este.

## Versiones

| Herramienta | Versión | Estado |
|---|---|---|
| `claude --version` | 2.1.327 | OK (mínimo 2.1.283) |
| `node --version` | v24.11.1 | OK |
| `git --version` | 2.51.0.windows.1 | OK |
| `claude plugin --help` | existe (`configure`, `details`, `enable`, `disable`, `eval`, …) | OK |

## Repositorio

- `git status -sb`: `## main...origin/main`, sin cambios pendientes.
- `git log --oneline -1`: `ba4b95f docs: close Sesión 7 with the team setup`.
- `HEAD` = `origin/main` = `ba4b95f`: subido (según la última referencia local del remoto; no se hizo `git fetch`).

## Piezas

| Pieza | Dónde | Estado |
|---|---|---|
| Skill de la sesión 4 | `.claude/skills/carrito-localstorage/SKILL.md` | existe, `name: carrito-localstorage` |
| Tech Lead de la sesión 5 | `.claude/agents/tech-lead.md` | existe, `name: tech-lead`, `tools: Read, Grep, Glob` |
| Hook PreToolUse | `.claude/hooks/bloquear-peligroso.js` | existe |
| Hook Stop | `.claude/hooks/exigir-pruebas.js` | existe |
| Probador | `.claude/hooks/probar-hooks.js` | existe |

`.claude/settings.json` registra los dos hooks con `$CLAUDE_PROJECT_DIR` y el matcher `Bash|PowerShell|Read|Grep|Glob`.

## La red de la sesión 6

- Versión: **2.1**. Están los tres archivos, el hook peligroso vigila Bash y PowerShell, y el hook de pruebas corrige la ruta `c:` en minúscula.
- Salida final de `node .claude/hooks/probar-hooks.js`: `23 de 23 casos dan lo esperado.` (exit 0).

## Dos rasgos de los hooks (sin cambiarlos)

1. `bloquear-peligroso.js` compara contra `"tech-lead"` exacto: `e.agent_type === "tech-lead"`. Dentro de un plugin (`<plugin>:tech-lead`) no lo frenaría.
2. `exigir-pruebas.js` corre `npm test` siempre que haya cambios fuera de `docs/`, sin mirar si el proyecto tiene script `test`. En una carpeta sin ese script, bloquearía el Stop.

Los dos están bien para este proyecto; la Fase 8.1 los adapta solo en la copia que va al plugin (decisiones 1 y 2).

## Output style

No hay: `.claude/output-styles/` no existe. Se anota y no se inventa (decisión 7).

## Tablas en `docs/`

| Tabla | Estado |
|---|---|
| `MEDICIONES.md` | existe |
| `COSTOS.md` | existe |
| `REVISION.md` | existe |
| `PERMISOS.md` | existe |
| `SECRETOS.md` | existe |
| `INYECCION.md` | **no existe**: la Fase 7.4 no se hizo (`docs/BITACORA.md:925`) |
| `DECISIONES.md` | existe |

## Plugin

`plugin/` y `.claude-plugin/` todavía no existen.

## Veredicto

Terreno listo para la 8.1 y la 8.2; falta `docs/INYECCION.md` (Fase 7.4 pendiente), que solo afecta al showcase de la 8.3.
