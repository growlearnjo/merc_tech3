# Plugin `red-de-seguridad` — Fase 8.1

Fecha: 2026-11-23. Claude Code 2.1.327.

- Plugin: `red-de-seguridad`, versión 1.0.0, en `plugin/`.
- Marketplace: `merc-tech3-plugins`, en `.claude-plugin/marketplace.json`, con `"source": "./plugin"`.
- No se instaló en este proyecto: sus hooks ya corren desde `.claude/settings.json`, y correrían dos veces (decisión 4).

## Qué entra, qué no, y por qué

Criterio único: **¿sirve en un proyecto que no es el mío?** Antes de decidir,
se anotó qué parte de cada pieza depende de esta app.

| Pieza | Qué depende de mi app | ¿Entra? | Por qué |
|---|---|---|---|
| Skill `carrito-localstorage` (S4) | Casi todo: `CartProvider.tsx:35` y `:46`, `auth.ts:29-43`, la tabla `cart_items`, la regla 7 del CLAUDE.md, `docs/00-ANTES.md` §5, y una verificación con `grep … src` que espera `CartProvider.tsx` | **No** | En otro proyecto no existen esos archivos ni esa regla. Se activaría con pedidos de "guardar en el navegador" y daría instrucciones sobre archivos que allí no existen |
| Tech Lead `tech-lead.md` (S5) | Poco: la descripción dice "la rúbrica de la tienda", con ejemplos de la app; R2 busca `S/ [0-9]` (soles); la orden 2 da por hecho que hay `docs/` | **Sí**, copia tal cual | La rúbrica R1–R4 es general ("ubica cuál es la capa de datos del proyecto", "Read del manifiesto"). Solo usa herramientas de lectura. Lo que es de la tienda no rompe nada fuera |
| Hook `bloquear-peligroso.js` (S6) | La parte 1 (comandos destructivos) es general. La parte 2 compara `agent_type === "tech-lead"` exacto | **Sí, adaptado** en la copia | Dentro del plugin el agente es `red-de-seguridad:tech-lead` (decisión 1) |
| Hook `exigir-pruebas.js` (S6) | Supone que existe un script `test` (aquí, Vitest) | **Sí, adaptado** en la copia | Sin script `test`, bloquearía cada Stop (decisión 2) |
| Probador `.claude/hooks/probar-hooks.js` (S6) | Prueba mis 23 casos | **No tal cual** | Viaja el probador del plugin del anexo: los mismos 23 casos más 4 que solo aparecen fuera (decisión 13) |
| Registro de hooks en `.claude/settings.json` | `$CLAUDE_PROJECT_DIR/.claude/hooks/…` | **No**; lo reemplaza `plugin/hooks/hooks.json` | El plugin usa `${CLAUDE_PLUGIN_ROOT}` (decisión 3), con el mismo matcher `Bash\|PowerShell\|Read\|Grep\|Glob` (decisión 14) |
| Permisos de `.claude/settings.json` (S6) | Todos: mis scripts (`typecheck`, `db:reset`), mis archivos protegidos (`docs/00-ANTES.md`, `docs/MEDICIONES.md`, `.claude/hooks/**`, `.env*`) | **No** | Es la matriz de este proyecto. No tiene sentido en otro |
| Servidor MCP `conversaciones` (`.mcp.json`) | `--confine-roots` apunta a una carpeta de mi máquina | **No** | Llevaría una ruta de mi máquina dentro de `plugin/` |
| Output style | — | **No existe** | `.claude/output-styles/` no existe. Se anota y no se inventa (decisión 7) |
| Workflow `ci.yml` (S7) | `NPM_VERSION` de mi lockfile; mis scripts `typecheck`, `test` y `build` | **No** | Es de GitHub Actions, no de Claude Code |
| Workflow `claude-review.yml` (S7) | El secreto `CLAUDE_CODE_OAUTH_TOKEN` de mi repositorio | **No** | Es de GitHub Actions, no de Claude Code |

## Lo que quedó en `plugin/`

```
plugin/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   └── tech-lead.md
├── hooks/
│   └── hooks.json
├── scripts/
│   ├── bloquear-peligroso.js   (versión que viaja)
│   ├── exigir-pruebas.js       (versión que viaja)
│   └── probar-hooks.js
└── README.md
```

## El probador, dos corridas

### Corrida 1 — los hooks de `.claude/hooks/` copiados tal cual: 25 de 27

Antes de correrlo se comprobó que las copias eran idénticas a los originales
(`diff --strip-trailing-cr` sin diferencias).

```
$ node plugin/scripts/probar-hooks.js
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: git reset --hard después de &&
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: git reset --hard después de ;
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: git push --force
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: git push con +rama
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: git clean -fd
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: git checkout -- .
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: git restore .
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: rm -rf
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: rm -r -f
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: rm --recursive --force
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: Remove-Item -Recurse -Force
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: rd /s /q
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Evento que no es JSON
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Bash: git status pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · PowerShell: git status pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Bash: git restore --staged pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Bash: rm de un archivo pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · PowerShell: Remove-Item -Force de un archivo pasa
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Tech Lead: Read de docs/ (ruta de Windows)
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Tech Lead: Grep con path docs
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Tech Lead: Read de package.json pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Conversación principal: Read de docs/ pasa
FALLA  exit=0 (esperado 2)  bloquear-peligroso.js · Plugin: <plugin>:tech-lead, Read de docs/
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Plugin: <plugin>:tech-lead, Read de package.json pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Otro agente con nombre parecido, Read de docs/ pasa
OK     exit=0 (esperado 0)  exigir-pruebas.js · Stop en la segunda vuelta pasa
FALLA  exit=2 (esperado 0)  exigir-pruebas.js · Stop en un proyecto con cambios y sin script test pasa

25 de 27 casos dan lo esperado.
```

**Qué falló y por qué solo fuera de mi proyecto:**

1. **`red-de-seguridad:tech-lead` lee `docs/` y pasa** (exit 0, se esperaba 2).
   El hook compara `e.agent_type === "tech-lead"` exacto. En mi proyecto el
   agente se llama `tech-lead` y el hook lo frena. Dentro del plugin se llama
   `red-de-seguridad:tech-lead`: la comparación no coincide y el hook lo deja
   pasar sin avisar.
2. **Stop en un proyecto con cambios y sin script `test` bloquea** (exit 2, se
   esperaba 0). El hook corre `npm test` siempre que hay cambios fuera de
   `docs/`. En mi proyecto siempre existe el script (Vitest). En un proyecto sin
   él, `npm test` falla con "Missing script" y el agente nunca podría darse por
   terminado.

### Qué cambió entre una corrida y otra

Solo `plugin/scripts/`. Los hooks de `.claude/hooks/` no se tocaron.

```diff
# bloquear-peligroso.js
-  if (e.agent_type === "tech-lead" && ["Read", "Grep", "Glob"].includes(tool)) {
+  // Dentro de un plugin se llama "<plugin>:tech-lead": se reconoce con o sin prefijo.
+  const esTechLead = /(^|:)tech-lead$/.test(String(e.agent_type || ""));
+  if (esTechLead && ["Read", "Grep", "Glob"].includes(tool)) {

# exigir-pruebas.js
+  // Un proyecto sin script "test" no tiene pruebas que exigir: el plugin no se las inventa.
+  let pkg = {};
+  try {
+    pkg = JSON.parse(require("fs").readFileSync(require("path").join(cwd, "package.json"), "utf8"));
+  } catch {}
+  if (!pkg.scripts || !pkg.scripts.test) process.exit(0);
```

A cada uno se le agregó además "Versión que viaja en el plugin." en la primera
línea de comentario. El texto es el del anexo de `Ed5_sesion8.md`, exacto.

### Corrida 2 — la versión que viaja: 27 de 27

```
$ node plugin/scripts/probar-hooks.js
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: git reset --hard después de &&
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: git reset --hard después de ;
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: git push --force
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: git push con +rama
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: git clean -fd
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: git checkout -- .
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: git restore .
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: rm -rf
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: rm -r -f
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Bash: rm --recursive --force
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: Remove-Item -Recurse -Force
OK     exit=2 (esperado 2)  bloquear-peligroso.js · PowerShell: rd /s /q
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Evento que no es JSON
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Bash: git status pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · PowerShell: git status pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Bash: git restore --staged pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Bash: rm de un archivo pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · PowerShell: Remove-Item -Force de un archivo pasa
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Tech Lead: Read de docs/ (ruta de Windows)
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Tech Lead: Grep con path docs
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Tech Lead: Read de package.json pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Conversación principal: Read de docs/ pasa
OK     exit=2 (esperado 2)  bloquear-peligroso.js · Plugin: <plugin>:tech-lead, Read de docs/
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Plugin: <plugin>:tech-lead, Read de package.json pasa
OK     exit=0 (esperado 0)  bloquear-peligroso.js · Otro agente con nombre parecido, Read de docs/ pasa
OK     exit=0 (esperado 0)  exigir-pruebas.js · Stop en la segunda vuelta pasa
OK     exit=0 (esperado 0)  exigir-pruebas.js · Stop en un proyecto con cambios y sin script test pasa

27 de 27 casos dan lo esperado.
```

El caso "Otro agente con nombre parecido" (`mi-tech-lead-2`) pasa en las dos
corridas: la expresión `/(^|:)tech-lead$/` no captura agentes que solo contienen
"tech-lead" en el nombre.

## Validación

La ruta del proyecto se abrevia como `<RUTA_PROYECTO>`. El resto es la salida tal
cual.

```
$ claude plugin validate ./plugin
Validating plugin manifest: <RUTA_PROYECTO>\plugin\.claude-plugin\plugin.json

Provides:

  ❯ hooks: PreToolUse Bash|PowerShell|Read|Grep|Glob command "node \"${CLAUDE_PLUGIN_ROOT}/scripts/bloquear-peligroso.js\""
  ❯ hooks: Stop command "node \"${CLAUDE_PLUGIN_ROOT}/scripts/exigir-pruebas.js\""
  ❯ agents: red-de-seguridad:tech-lead

Plugin directory checks for <RUTA_PROYECTO>\plugin. The plugin directory is the catalog that plugins are submitted to; this is a subset of what it runs.

✘ Found 1 problem the plugin directory refuses a submission for:

  ❯ LICENSE: License missing (LICENSE_MISSING). This plugin has no LICENSE file and plugin.json has no `license`. The directory requires one of them.
    Fix: Add a LICENSE file to the plugin folder, or set `license` (an SPDX id) in plugin.json. A LICENSE at the repository root also counts for a plugin in a sub-folder.

⚠ Found 1 plugin directory warning:

  ❯ .claude-plugin/plugin.json icon: No icon (ICON_MISSING). No icon found in the plugin. The validator looked for `icon` in plugin.json and for a file at .claude-plugin/icon.png or .svg, assets/icon.png or .svg, or a similar path.
    Fix: Add a square PNG to the plugin folder at .claude-plugin/icon.png, or set `icon` in plugin.json to the path of a square PNG or JPEG in the plugin: 512 to 2048 px on each side, under 2 MB. SVG and WebP are not accepted. That file may become the listing icon only once: the first time the plugin is saved or submitted in the developer portal. Adding or changing it later does not change the icon.

About these checks:

  ❯ The repository root was taken to be <RUTA_PROYECTO>, the nearest parent folder with a .git folder or file. No license file that counts (a regular file with one of the file names the plugin directory accepts) was found there either. For a plugin submitted from a git repository, the repository root is the top folder of that repository, with only what is committed there, so check that the folder named above is that top folder.

These plugin directory checks do not change the result below. To fail on problems the directory refuses a submission for, run this command with --profile directory.

The directory also runs a safety scan and a policy review that this command does not.

✔ Validation passed
```

```
$ claude plugin validate .
Validating marketplace manifest: <RUTA_PROYECTO>\.claude-plugin\marketplace.json

✔ Validation passed
```

La validación pasa en los dos. Avisa de `LICENSE` y del ícono, y la propia salida
dice que esos chequeos son del directorio público y "do not change the result".
No se agregan: el plugin no se publica en ningún directorio (decisión 6).
