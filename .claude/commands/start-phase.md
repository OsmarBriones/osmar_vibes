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

Si existe `docs/blockers.md` con una entrada de esta fase, revisá si ya se puede resolver (el
usuario la respondió en el chat, o actualizó `docs/gdd.md`). Si sí, resolvé la tarea con esa
respuesta y borrá la entrada. Si sigue sin respuesta, saltá esa tarea puntual de nuevo sin
volver a preguntar en cada corrida — ya está registrada.

Si alguna tarea de la fase termina en la marca `(HUMAN)`, significa que tiene una parte que solo
el usuario puede completar (un asset de arte final, audio con licencia, una decisión de negocio).
No te la saltees: implementa la parte que sí es agent-doable con un placeholder razonable (una
figura simple, un sonido generado, un valor por defecto) y trátala como pendiente en el paso 6 —
nunca la marques como completada vos mismo.

### 2. Implementar

Implementa cada tarea pendiente de la fase en `game/`, siguiendo la estructura de carpetas de
`.claude/CLAUDE.md`, el criterio de sistemas de `docs/architecture.md` (manager global vs
componente por entidad, según corresponda a lo que esa tarea toca) y, sin excepción, el estilo de
código de `docs/coding-standards.md`. Consulta `docs/gdd.md` cuando necesites detalles de diseño
(valores, comportamiento esperado, etc.) en vez de inventarlos.

No implementes tareas de fases futuras todavía, aunque sea tentador.

Si te encontrás con una ambigüedad real del GDD — no un detalle menor que puedas decidir con
criterio razonable, sino algo que cambia sustancialmente el resultado y que `docs/gdd.md` no
resuelve ni contradice — no inventes una respuesta. Agrega una entrada a `docs/blockers.md`
(créalo si no existe) con este formato:

```markdown
## Fase N: <título de la tarea bloqueada>
**Pregunta:** <la ambigüedad concreta, en una o dos frases>
**Por qué bloquea:** <qué decisión depende de esto>
```

y seguí con el resto de tareas de la fase que no dependan de esa duda. Si *toda* la fase depende
de esa ambigüedad, detenete ahí, no commitees nada a medias, y avisale al usuario que hay un
blocker que resolver antes de continuar. Una vez que el usuario responda la pregunta (en el chat
o editando `docs/gdd.md` directamente), la siguiente corrida de `/start-phase` debe resolver la
tarea con esa respuesta y borrar la entrada de `docs/blockers.md`.

### 3. Escribir/actualizar tests

Por cada test case de `docs/test-cases.md` de esta fase, crea o actualiza el test GUT
correspondiente en `game/tests/unit/`, siguiendo la convención `test_<feature>.gd`. Salteá los
test cases que dependan de una tarea bloqueada o pendiente de `(HUMAN)`.

### 4. Correr los tests

Localiza el ejecutable de Godot (ver `.claude/CLAUDE.md`, sección Testing) y corre la suite:

```
<godot> --headless --path game -s res://addons/gut/gut_cmdln.gd -gconfig=.gutconfig.json
```

Si es la primera ejecución en esta sesión y `game/.godot/` no existe todavía, corre primero
`<godot> --headless --path game --import`.

Si algún test falla, corrige la implementación y vuelve a correr los tests. Repite hasta que
todos pasen. No avances al siguiente paso con tests en rojo.

### 5. Revisión de estilo

Antes de tocar los checklists o commitear, revisa el diff de esta fase (`game/` solamente) contra
`docs/coding-standards.md` y `docs/architecture.md` punto por punto. Los tests validan
comportamiento, no estilo ni arquitectura — esta revisión es la única que atrapa violaciones de
tipado, nomenclatura, orden de miembros, uso de señales/autoloads, o un sistema construido con el
patrón equivocado (componente que debía ser autoload, o viceversa). Presta especial atención a las dos reglas duras del documento: ningún
archivo `.gd` tocado en esta fase supera 100 caracteres por línea ni 300 líneas totales — si un
archivo pasa el límite de líneas, separalo en módulos según responsabilidad antes de continuar,
no lo dejes pasar. Corrige cualquier violación que encuentres y vuelve a correr los tests del
paso 4 si el fix tocó lógica.

### 6. Actualizar checklists

Marca como completadas (`- [x]`) únicamente las tareas y test cases que de verdad terminaste y
pasaron sus tests:
- En `docs/todo.md`: todas las tareas de la fase, EXCEPTO las que quedaron bloqueadas (`docs/blockers.md`)
  o las `(HUMAN)` cuya parte humana no se completó — esas quedan en `- [ ]`.
- En `docs/test-cases.md`: los test cases correspondientes a las tareas que sí se cerraron.

### 7. Commit

Haz un commit que incluya únicamente los cambios de esta fase (código del juego, tests, y las
actualizaciones a `todo.md`/`test-cases.md`/`docs/blockers.md`), con el formato definido en
`.claude/CLAUDE.md`:

```
Fase N: <título de la fase>

<resumen breve de qué se implementó>
```

Si la fase quedó parcialmente completa (por tareas `(HUMAN)` pendientes o blockers), dilo
explícitamente en el cuerpo del commit: qué quedó pendiente y por qué.

### 8. Resumen final

Reporta al usuario: qué fase se completó, qué tests pasaron, y si el proyecto tiene fases
pendientes o ya está completo. Si quedaron tareas `(HUMAN)` pendientes o se registró algo en
`docs/blockers.md`, dilo explícitamente y qué necesita el usuario para desbloquearlo. No
arranques automáticamente la siguiente fase — eso requiere volver a correr `/start-phase`.
