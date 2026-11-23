# red-de-seguridad

Un revisor que no edita y dos hooks que frenan lo que una regla de permisos no
alcanza a ver. Sale del curso Claude Code Ed. 5 (sesiones 5 y 6), empaquetado
en la sesión 8.

## Qué trae

| Pieza | Dónde | Qué hace |
|---|---|---|
| `tech-lead` | `agents/tech-lead.md` | Revisor de solo lectura (`Read`, `Grep`, `Glob`). Pasa una rúbrica de cuatro preguntas (responsabilidad única, valores ajustables, pruebas, permisos) y entrega hallazgos por severidad con una nota. Dentro del plugin se llama `red-de-seguridad:tech-lead`. |
| Hook `PreToolUse` | `scripts/bloquear-peligroso.js` | En Bash y en PowerShell bloquea `git reset --hard`, `git checkout --`, `git restore` (salvo `--staged`), `git push --force` o con `+rama`, `git clean -f` y el borrado recursivo de carpetas. Impide que el `tech-lead`, con o sin el prefijo del plugin, lea `docs/`. |
| Hook `Stop` | `scripts/exigir-pruebas.js` | Si cambió algo fuera de `docs/` y el proyecto tiene script `test`, corre `npm test`. Si falla, el agente no puede darse por terminado. Si el proyecto no tiene script `test`, no exige nada. |
| Probador | `scripts/probar-hooks.js` | `node scripts/probar-hooks.js` les pasa 27 eventos falsos a los hooks y compara el código de salida. No ejecuta ningún comando. |

Los hooks se registran en `hooks/hooks.json` con `${CLAUDE_PLUGIN_ROOT}` y el
matcher `Bash|PowerShell|Read|Grep|Glob`. Necesitan `node` y, para el hook de
pruebas, `git`.

## Cómo se instala

Desde la carpeta del proyecto donde lo quieres, siempre con `--scope local` para
no tocar tu configuración global:

```bash
claude plugin marketplace add <RUTA_DEL_REPOSITORIO> --scope local
claude plugin install red-de-seguridad@merc-tech3-plugins --scope local
```

`<RUTA_DEL_REPOSITORIO>` es la carpeta que contiene `.claude-plugin/marketplace.json`.

Para quitarlo:

```bash
claude plugin uninstall red-de-seguridad@merc-tech3-plugins --scope local
```

No lo instales en un proyecto que ya tiene estos mismos hooks en su
`.claude/settings.json`: correrían dos veces.

## Qué quedó fuera, y por qué

El criterio fue uno: ¿sirve en un proyecto que no es el de origen?

| Pieza | Por qué no viaja |
|---|---|
| Skill `carrito-localstorage` | Cita archivos, líneas y una regla de la tienda de origen (el carrito en `localStorage`). En otro proyecto no hay nada de eso. |
| Matriz de permisos | Nombra los scripts y los archivos protegidos del proyecto de origen. |
| Servidor MCP | Su configuración apunta a una ruta de la máquina de origen. |
| Output style | No existía ninguno. |
| CI y revisión de PR | Son workflows de GitHub Actions, no piezas de Claude Code. |

## Lo que todavía es de la tienda

El `tech-lead` viaja tal cual. Su descripción habla de "la rúbrica de la tienda"
y R2 busca montos en soles (`S/ [0-9]`). En otro proyecto ese grep no encuentra
nada, y el resto de la rúbrica es general.
