---
description: Revisa que el código de game/ cumpla docs/architecture.md (--all para revisar todo, no solo lo no commiteado)
---

Vas a revisar que el código de `game/` cumpla con `docs/architecture.md` — cómo se descompone la
lógica del juego en sistemas (manager global/autoload vs componente por entidad) y cómo se
comunican entre sí. Esta skill es de **solo revisión**: reporta hallazgos, no corrige código salvo
que el usuario lo pida explícitamente después de ver el reporte.

Argumentos recibidos: `$ARGUMENTS`

## 0. Precondición

Si `docs/architecture.md` no existe, avisa al usuario y detente — no hay contra qué revisar.

## 1. Determinar alcance

Si `$ARGUMENTS` contiene `--all`, el alcance es **todo** el código relevante de `game/` (todos los
`.gd` bajo `game/scripts/` y `game/scenes/`, excluyendo `game/addons/gut/`).

Si no, el alcance es **solo lo no commiteado**: corre `git status`/`git diff` acotado a `game/`
(excluyendo `game/addons/gut/` y `game/.godot/`) para listar archivos `.gd` nuevos, modificados,
staged o unstaged. Si no hay ningún cambio sin commitear, informa al usuario que no hay nada que
revisar en este modo y sugiere `--all` si quiere revisar el código existente, y detente.

## 2. Leer contexto

Lee `docs/architecture.md` completo y `docs/gdd.md` (para entender qué sistemas existen en el
juego y qué rol le corresponde a cada uno según el diseño).

## 3. Revisar cada archivo del alcance

Por cada archivo, verificando contra `docs/architecture.md` punto por punto, y clasificando cada
hallazgo como **grave** o **menor** (la clasificación se usa en el paso 4):

- **Patrón** (grave): ¿es un manager global implementado como componente por entidad, o viceversa?
  ¿tiene acoplamiento directo indebido (referencias hardcodeadas a nodos de otra rama del árbol,
  `get_node("../../...")`, llamadas directas a métodos internos de otro sistema) en vez del
  mecanismo de comunicación que define `docs/architecture.md` (señales, autoloads, etc.)?
- **Responsabilidad** (grave): ¿el archivo acumula responsabilidades que no le corresponden a ese
  sistema según su rol descrito en `docs/architecture.md`?
- **Ubicación** (menor): ¿está en la carpeta correcta según su rol? (`scripts/systems/` para
  managers globales/autoload, `scenes/components/` para componentes por entidad, script adjunto a
  su propia escena si tiene lógica propia). Incorrecto pero no rompe el patrón en sí — es
  desprolijidad de organización, no una violación estructural.

## 4. Leer política de estrictez

Lee `.claude/review-policy.jsonc` si existe. Usa su clave `estrictez`:

- `"cualquier-hallazgo"` (default si el archivo no existe, no tiene la clave, o está mal formado):
  cualquier hallazgo, grave o menor, cuenta para el veredicto.
- `"solo-reglas-duras"`: solo los hallazgos **graves** cuentan para el veredicto; los **menores**
  se listan igual en el reporte pero marcados como advertencia, sin afectar el veredicto.

## 5. Reportar y emitir veredicto

Presenta los hallazgos ordenados de más a menos severo, cada uno con: archivo:línea, qué regla de
`docs/architecture.md` se incumple, su clasificación (grave/menor), y una sugerencia de fix
concreta. Si no hay hallazgos, dilo explícitamente ("no se encontraron violaciones de arquitectura
en el alcance revisado") — no inventes problemas para tener algo que reportar.

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
