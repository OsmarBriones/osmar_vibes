---
description: Revisa que el código de game/ cumpla docs/coding-standards.md (--all para revisar todo, no solo lo no commiteado)
---

Vas a revisar que el código de `game/` cumpla con `docs/coding-standards.md` — el único estilo de
código válido en este proyecto (tipado, orden de miembros, nomenclatura, uso de señales/autoloads,
testeabilidad, límites de tamaño de archivo, etc.). Esta skill es de **solo revisión**: reporta
hallazgos, no corrige código salvo que el usuario lo pida explícitamente después de ver el reporte.

Argumentos recibidos: `$ARGUMENTS`

## 0. Precondición

Si `docs/coding-standards.md` no existe, avisa al usuario y detente — no hay contra qué revisar.

## 1. Determinar alcance

Si `$ARGUMENTS` contiene `--all`, el alcance es **todo** el código relevante de `game/` (todos los
`.gd` bajo `game/scripts/`, `game/scenes/` y `game/tests/unit/`, excluyendo `game/addons/gut/`).

Si no, el alcance es **solo lo no commiteado**: corre `git status`/`git diff` acotado a `game/`
(excluyendo `game/addons/gut/` y `game/.godot/`) para listar archivos `.gd` nuevos, modificados,
staged o unstaged. Si no hay ningún cambio sin commitear, informa al usuario que no hay nada que
revisar en este modo y sugiere `--all` si quiere revisar el código existente, y detente.

## 2. Leer contexto

Lee `docs/coding-standards.md` completo.

## 3. Revisar cada archivo del alcance

Por cada archivo, verificando contra `docs/coding-standards.md` punto por punto, y clasificando
cada hallazgo como **grave** o **menor** (la clasificación se usa en el paso 4):

- **Reglas duras** (siempre grave): ningún archivo `.gd` del alcance supera 100 caracteres por
  línea ni 300 líneas totales. Si un archivo excede el límite de líneas, la sugerencia de fix debe
  ser separarlo en módulos según responsabilidad, no solo señalarlo.
- **Tipado estático y testeabilidad** (grave): variables/parámetros/retornos sin tipar donde el
  documento lo exige, o dependencias acopladas de forma que impiden testear el nodo/sistema como
  pide el documento.
- **Nomenclatura, orden de miembros, uso de señales/autoloads y demás convenciones** (menor):
  desviaciones de estilo que no afectan la corrección ni la testeabilidad, solo la consistencia del
  código con el resto del proyecto.

## 4. Leer política de estrictez

Lee `.claude/review-policy.jsonc` si existe. Usa su clave `estrictez`:

- `"cualquier-hallazgo"` (default si el archivo no existe, no tiene la clave, o está mal formado):
  cualquier hallazgo, grave o menor, cuenta para el veredicto.
- `"solo-reglas-duras"`: solo los hallazgos **graves** cuentan para el veredicto; los **menores**
  se listan igual en el reporte pero marcados como advertencia, sin afectar el veredicto.

## 5. Reportar y emitir veredicto

Presenta los hallazgos ordenados de más a menos severo (las reglas duras primero), cada uno con:
archivo:línea, qué regla de `docs/coding-standards.md` se incumple, su clasificación
(grave/menor), y una sugerencia de fix concreta. Si no hay hallazgos, dilo explícitamente ("no se
encontraron violaciones de estilo en el alcance revisado") — no inventes problemas para tener algo
que reportar.

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
`solo-reglas-duras`, solo los graves — nunca queda por debajo de las reglas duras del punto
anterior). `APROBADO` en caso contrario. Este veredicto es lo que usa `/commit-and-push` para
decidir si permite o bloquea el commit — no lo omitas ni lo suavices.

Al final, pregunta al usuario si quiere que apliques los fixes ahora. No los apliques sin
confirmación. (Si te está invocando otro comando de forma automatizada, como `/review-all` o
`/commit-and-push`, seguí las instrucciones que te dieron sobre no preguntar y devolver el reporte
como texto.)
</content>
