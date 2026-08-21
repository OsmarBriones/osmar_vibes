# Proyecto: Godot Vibecoding Template

Este repo es un **template** para desarrollar juegos en Godot 4 (GDScript) guiado por Claude Code,
mediante los workflows `/new-game` y `/start-phase`. El juego real vive en `game/`; la especificación
del juego vive en `docs/`.

## Fuente de verdad

- `docs/gdd.md` — diseño del juego. Nunca se contradice al implementar; si algo no está claro, se
  actualiza el GDD primero.
- `docs/todo.md` — fases de desarrollo, cada una con checklist de tareas (`- [ ]` / `- [x]`).
- `docs/test-cases.md` — historias testables por fase (afirmaciones desde la perspectiva del
  jugador, pero 100% verificables por código), mapeadas 1:1 a tests de GUT.
- `docs/coding-standards.md` — el único estilo de código válido en este proyecto (tipado, orden
  de miembros, nomenclatura, uso de señales/autoloads, testeabilidad, etc.). Se sigue al pie de
  la letra, no como sugerencia.
- `docs/architecture.md` — cómo se descompone la lógica del juego en sistemas (manager global vs
  componente por entidad) y cómo se comunican entre sí. Define el CÓMO se organiza, no el QUÉ
  sistemas tiene el juego (eso lo define `docs/gdd.md`).
- `docs/blockers.md` — ambigüedades reales del GDD que `/start-phase` no puede resolver por su
  cuenta. No existe hasta que hace falta; se crea la primera vez que surge un blocker.

Una tarea de `docs/todo.md` marcada `(HUMAN)` tiene una parte que solo el usuario puede terminar
(arte final, audio con licencia, una decisión de negocio). `/start-phase` implementa la parte
agent-doable con un placeholder y deja el checkbox sin marcar hasta que el usuario complete su
parte.

## Estructura de `game/`

```
game/
  addons/gut/           # framework de testing, no tocar
  scenes/               # archivos .tscn
    components/         # componentes por entidad (ver docs/architecture.md)
  scripts/              # archivos .gd que no son de un nodo específico (autoloads, utils, resources)
    systems/            # managers globales/autoload (ver docs/architecture.md)
  assets/               # sprites, audio, fuentes
  tests/unit/           # un test_<feature>.gd por área funcional, extends GutTest
```

- Cada escena con lógica propia tiene su script adjunto en la misma carpeta que la escena (no en
  `scripts/`), salvo utilidades/autoloads compartidos.
- El estilo del código GDScript (tipado, nomenclatura, orden de miembros, uso de señales,
  performance, etc.) está definido en `docs/coding-standards.md` — no se improvisa.

## Testing (GUT)

- Cada test extiende `GutTest` y vive en `game/tests/unit/test_<feature>.gd`.
- Un caso de `docs/test-cases.md` = un método `test_*` (o varios si el caso cubre varios escenarios).
- Para correr los tests en modo headless:

  1. Localizar el ejecutable de Godot: primero `godot` en el PATH del sistema; si no existe, buscar
     un `Godot*.exe` dentro de `.tools/` en la raíz del repo (ahí es donde el usuario del template
     coloca su propia copia portable — nunca se commitea). Si no se encuentra ninguno de los dos,
     sugerir `/setup-godot` para instalarlo automáticamente (ver más abajo) en vez de simplemente
     fallar.
  2. Si es la primera vez que se abre el proyecto (no existe `game/.godot/`), importar primero:
     `<godot> --headless --path game --import`
  3. Correr los tests:
     `<godot> --headless --path game -s res://addons/gut/gut_cmdln.gd -gconfig=.gutconfig.json`
  4. Un run exitoso termina con `All tests passed!` y exit code `0`. Cualquier otra cosa es una fase
     no completada — no se marca como hecha en `todo.md` ni se hace commit.

`/setup-godot` automatiza el paso 1: detecta SO/arquitectura, determina la versión requerida (la
declarada en `game/project.godot`, o si el proyecto todavía no existe, el default centralizado en
`.claude/godot-version.jsonc`), y descarga/extrae una copia portable a `.tools/` — pidiendo
confirmación explícita antes de bajar nada. El fallback manual (instalar Godot a mano y copiarlo a
`.tools/`) sigue siendo válido, por si el entorno bloquea descargas (CI, redes corporativas).

## Commit y push

`/commit-and-push` es el **único camino autorizado** para ejecutar `git commit` o `git push` en
este proyecto. Ningún otro comando o skill (incluido `/start-phase`) ejecuta esos comandos
directamente — deben invocar `/commit-and-push`, salvo que:

- el propio comando lo declare explícitamente en sus instrucciones (documentando por qué se salta
  el gate), o
- el usuario lo pida explícitamente en el chat, fuera de la ejecución automatizada de un comando.

`/commit-and-push` corre `/review-all` (arquitectura, coding-standards, docs, y cualquier
`review-*.md` que se agregue) antes de commitear. Solo si todos los reviewers aprueban se hace el
commit y el push.

`/commit-and-push` acepta `--skip-reviews` para saltar ese gate y commitear/pushear directo. Esa
bandera **no la puede usar ningún workflow por su cuenta** (incluido `/start-phase`) — solo es
válida si el usuario la pide explícitamente en el chat, o si el comando que invoca
`/commit-and-push` la declara explícitamente en sus propias instrucciones justificando por qué.

`/start-phase` (paso 7) invoca `/commit-and-push` y, si es rechazado, corrige lo señalado por el
reporte y reintenta hasta el máximo de intentos definido en `.claude/review-policy.jsonc` antes de
detenerse y pedir intervención humana.

### `.claude/review-policy.jsonc`

Los reviewers y `/start-phase` leen este archivo en caliente en cada corrida — no hay que
"aplicarlo" ni sincronizarlo con nada, el archivo **es** la configuración vigente. Si no existe o
está mal formado, cada comando cae a sus valores por defecto (documentados en cada uno).

```jsonc
{
  "reviewers_activos": ["review-architecture", "review-coding-standards", "review-docs"],
  "max_intentos": 5,
  "estrictez": "cualquier-hallazgo"
}
```

- `reviewers_activos` — qué `review-*.md` corre `/review-all` (default: todos los que existan en
  `.claude/commands/`).
- `max_intentos` — cuántas veces reintenta `/start-phase` tras un `RECHAZADO` antes de parar y
  pedir intervención humana (default: `5`).
- `estrictez` — `"cualquier-hallazgo"` rechaza con cualquier hallazgo, sin importar severidad
  (default); `"solo-reglas-duras"` solo rechaza por hallazgos graves (reglas duras, correctitud,
  incongruencias que bloquean fases), y deja los menores como advertencia sin bloquear.

Convención de mensaje de commit por fase completada (usada por `/start-phase`):

```
Fase N: <título de la fase>

<resumen breve de qué se implementó>
```

No se mezclan cambios de dos fases en un mismo commit.

## Reglas generales

- No agregar dependencias/plugins de Godot fuera de GUT sin que quede explícito en `docs/gdd.md`.
- No se avanza a la siguiente fase de `todo.md` si los tests de la fase actual no pasan.
- El GDD, el todo y los test-cases se escriben en español (idioma del canal); el código, comentarios
  de código y mensajes de commit se escriben en inglés (convención estándar de la industria).
