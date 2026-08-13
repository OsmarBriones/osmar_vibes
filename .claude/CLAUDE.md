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

## Estructura de `game/`

```
game/
  addons/gut/       # framework de testing, no tocar
  scenes/           # archivos .tscn
  scripts/          # archivos .gd que no son de un nodo específico (autoloads, utils, resources)
  assets/           # sprites, audio, fuentes
  tests/unit/       # un test_<feature>.gd por área funcional, extends GutTest
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
     coloca su propia copia portable — nunca se commitea).
  2. Si es la primera vez que se abre el proyecto (no existe `game/.godot/`), importar primero:
     `<godot> --headless --path game --import`
  3. Correr los tests:
     `<godot> --headless --path game -s res://addons/gut/gut_cmdln.gd -gconfig=.gutconfig.json`
  4. Un run exitoso termina con `All tests passed!` y exit code `0`. Cualquier otra cosa es una fase
     no completada — no se marca como hecha en `todo.md` ni se hace commit.

## Convención de commits (usada por `/start-phase`)

Un commit por fase completada, mensaje en formato:

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
