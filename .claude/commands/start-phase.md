---
description: Implementa la siguiente fase pendiente de todo.md, la testea y hace commit
---

Vas a desarrollar la siguiente fase pendiente del proyecto siguiendo `docs/todo.md`,
`docs/test-cases.md` y `docs/gdd.md` como fuente de verdad. Sigue estos pasos en orden y no
te saltes ninguno.

### 0. Precondición

Si `docs/gdd.md`, `docs/todo.md` o `docs/test-cases.md` no existen, detente y dile al usuario que
debe correr `/new-game <idea>` primero.

### 1. Identificar la fase

Lee `docs/todo.md` y encuentra la primera fase que tenga al menos una tarea sin marcar (`- [ ]`).
Si todas las fases están completas, informa al usuario que el proyecto está terminado según
`todo.md` y detente — no hagas nada más.

Lee también la sección correspondiente en `docs/test-cases.md`: esos casos son el criterio de
"hecho" para esta fase, no una sugerencia.

### 2. Implementar

Implementa cada tarea pendiente de la fase en `game/`, siguiendo las convenciones de
`.claude/CLAUDE.md` (estructura de carpetas, estilo GDScript). Consulta `docs/gdd.md` cuando
necesites detalles de diseño (valores, comportamiento esperado, etc.) en vez de inventarlos.

No implementes tareas de fases futuras todavía, aunque sea tentador.

### 3. Escribir/actualizar tests

Por cada test case de `docs/test-cases.md` de esta fase, crea o actualiza el test GUT
correspondiente en `game/tests/unit/`, siguiendo la convención `test_<feature>.gd`.

### 4. Correr los tests

Localiza el ejecutable de Godot (ver `.claude/CLAUDE.md`, sección Testing) y corre la suite:

```
<godot> --headless --path game -s res://addons/gut/gut_cmdln.gd -gconfig=.gutconfig.json
```

Si es la primera ejecución en esta sesión y `game/.godot/` no existe todavía, corre primero
`<godot> --headless --path game --import`.

Si algún test falla, corrige la implementación y vuelve a correr los tests. Repite hasta que
todos pasen. No avances al siguiente paso con tests en rojo.

### 5. Actualizar checklists

Una vez que todos los tests de la fase pasan:
- Marca todas las tareas de la fase como completadas en `docs/todo.md` (`- [x]`).
- Marca todos los test cases de la fase como completados en `docs/test-cases.md` (`- [x]`).

### 6. Commit

Haz un commit que incluya únicamente los cambios de esta fase (código del juego, tests, y las
actualizaciones a `todo.md`/`test-cases.md`), con el formato definido en `.claude/CLAUDE.md`:

```
Fase N: <título de la fase>

<resumen breve de qué se implementó>
```

### 7. Resumen final

Reporta al usuario: qué fase se completó, qué tests pasaron, y si el proyecto tiene fases
pendientes o ya está completo. No arranques automáticamente la siguiente fase — eso requiere
volver a correr `/start-phase`.
