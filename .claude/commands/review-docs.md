---
description: Revisa que docs/ esté actualizado y sea congruente (--all también lo contrasta contra el código ya implementado)
---

Vas a revisar que los documentos de `docs/` estén actualizados y sean congruentes entre sí. A
diferencia de `/review-architecture` y `/review-coding-standards`, esta skill no revisa código: su
objeto de revisión son los propios documentos. Es de **solo revisión**: reporta hallazgos, no
corrige nada salvo que el usuario lo pida explícitamente después de ver el reporte.

Argumentos recibidos: `$ARGUMENTS`

## 0. Precondición

Si `docs/gdd.md` no existe, avisa al usuario que debe correr `/new-game <idea>` primero y detente.

## 1. Determinar alcance

Reúne todos los documentos que existan de este conjunto: `docs/gdd.md`, `docs/todo.md`,
`docs/test-cases.md`, `docs/architecture.md`, `docs/coding-standards.md`, `docs/blockers.md`.

Si `$ARGUMENTS` contiene `--all`, además de la consistencia doc-a-doc (paso 2), contrasta los
documentos contra el estado real del código ya implementado en `game/` (paso 3). Sin la bandera,
saltea el paso 3 — solo revisa consistencia entre documentos.

## 2. Consistencia entre documentos

Lee todos los documentos del alcance y verifica, clasificando cada hallazgo como **grave** o
**menor** (la clasificación se usa en el paso 4):

- **`docs/todo.md` ↔ `docs/test-cases.md`** (grave): cada fase de `todo.md` tiene su sección
  espejo en `test-cases.md` (mismo número y nombre de fase), y viceversa — ninguna fase queda sin
  casos de prueba ni ningún bloque de casos de prueba referencia una fase inexistente.
- **`docs/gdd.md` ↔ `docs/todo.md`** (grave): toda mecánica, control o sistema descrito en el GDD
  tiene al menos una tarea en alguna fase de `todo.md` que lo cubra (nada del GDD queda huérfano de
  plan de implementación); y ninguna tarea de `todo.md` contradice o implementa algo distinto de lo
  que dice el GDD.
- **`docs/architecture.md` ↔ `docs/gdd.md`/`docs/todo.md`** (grave): los sistemas que
  `architecture.md` da por sentado que existen (managers, componentes) tienen correlato en el
  GDD/todo; no describe patrones para sistemas que el juego no tiene.
- **`docs/blockers.md`** (menor, si existe): cada entrada referencia una fase que sigue existiendo
  y sin completar en `todo.md`; si una entrada quedó resuelta implícitamente (el GDD ya no es
  ambiguo en ese punto, o la fase ya está marcada como completa) o referencia una fase que ya no
  existe, es un hallazgo — debería haberse limpiado.
- **Checklist de idea lista de `docs/gdd.md`** (menor): sigue reflejando el contenido real del
  resto del documento (por ejemplo, si el alcance del MVP cambió pero el checklist no se
  actualizó).

## 3. Consistencia contra el código (solo con `--all`)

Para cada tarea marcada como completada (`- [x]`) en `docs/todo.md`, verifica que exista código
real en `game/` que la implemente (no solo un placeholder olvidado) y que el test case
correspondiente en `docs/test-cases.md` esté también marcado `- [x]` y tenga un método `test_*`
real en `game/tests/unit/`. Señala como hallazgo **grave** cualquier tarea marcada como completa
sin respaldo en el código o en los tests, y cualquier test case marcado como completo sin un método
de test correspondiente — una fase marcada como hecha sin estarlo es el tipo de incongruencia más
peligrosa que revisa esta skill.

## 4. Leer política de estrictez

Lee `.claude/review-policy.jsonc` si existe. Usa su clave `estrictez`:

- `"cualquier-hallazgo"` (default si el archivo no existe, no tiene la clave, o está mal formado):
  cualquier hallazgo, grave o menor, cuenta para el veredicto.
- `"solo-reglas-duras"`: solo los hallazgos **graves** cuentan para el veredicto; los **menores**
  se listan igual en el reporte pero marcados como advertencia, sin afectar el veredicto.

## 5. Reportar y emitir veredicto

Presenta los hallazgos ordenados de más a menos severo (los graves primero), cada uno con: los
documentos/líneas involucrados, en qué consiste la incongruencia, su clasificación (grave/menor), y
una sugerencia de fix concreta (qué documento editar y cómo). Si no hay hallazgos, dilo
explícitamente ("los documentos están consistentes y actualizados") — no inventes problemas para
tener algo que reportar.

Cierra siempre el reporte con una línea de veredicto, en este formato exacto:

```
Veredicto: APROBADO
```

o

```
Veredicto: RECHAZADO
```

Aplicá el veredicto según la política leída en el paso 4: `RECHAZADO` si hay al menos un hallazgo
que cuenta para el veredicto según esa política (bajo `cualquier-hallazgo`, cualquiera; bajo
`solo-reglas-duras`, solo los graves). `APROBADO` en caso contrario. Este veredicto es lo que usa
`/commit-and-push` para decidir si permite o bloquea el commit — no lo omitas ni lo suavices.

Al final, pregunta al usuario si quiere que apliques los fixes ahora. No los apliques sin
confirmación. (Si te está invocando otro comando de forma automatizada, como `/review-all` o
`/commit-and-push`, seguí las instrucciones que te dieron sobre no preguntar y devolver el reporte
como texto.)
</content>
