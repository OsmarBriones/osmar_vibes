---
description: Agrega una nueva fase a todo.md (y sus test cases) a partir de un pedido del usuario
---

El usuario quiere agregar una nueva fase de desarrollo a partir de este pedido:

<pedido>
$ARGUMENTS
</pedido>

## Si el pedido está vacío

Si `<pedido>` está vacío o no contiene información suficiente para identificar qué debería agregar
la nueva fase, NO continúes con los siguientes pasos. Responde únicamente pidiendo que se vuelva a
ejecutar el comando incluyendo el pedido, por ejemplo:
`/add-phase agregar un sistema de power-ups que aparecen aleatoriamente en el nivel`. No modifiques
ningún archivo en este caso.

## Precondición

Si `docs/gdd.md`, `docs/todo.md` o `docs/test-cases.md` no existen, detente y dile al usuario que
debe correr `/new-game <idea>` primero — `/add-phase` extiende un proyecto ya iniciado, no lo crea.

## Si el pedido es válido

Sigue estos pasos en orden:

### 1. Checklist de fase lista

Una fase nueva está lista para agregarse cuando estos 5 puntos tienen una respuesta concreta. Este
es el único criterio de "listo" — no busques más precisión que esta, el resto se resuelve durante
`/start-phase`:

1. **Qué agrega** — la capa/funcionalidad concreta que esta fase suma al juego (el verbo o sistema
   principal de la fase).
2. **Encaje con el diseño actual** — cómo interactúa con las mecánicas ya descritas en
   `docs/gdd.md` (o con fases ya definidas en `docs/todo.md`), sin contradecirlas.
3. **Alcance de la fase** — qué tareas concretas incluye, al nivel de detalle de una fase existente
   de `docs/todo.md`.
4. **Posición** — si va al final de `docs/todo.md` (caso por defecto) o debe insertarse en un lugar
   específico porque otra fase pendiente depende de ella.
5. **Qué queda fuera** — al menos una exclusión explícita de esta fase (para no confundirla con
   trabajo de otra fase futura).

Proceso:

1. Lee `docs/gdd.md` y `docs/todo.md` completos antes de analizar el pedido — necesitas el
   contexto real del juego, no solo el texto de `<pedido>`.
2. Intenta **inferir** cada uno de los 5 puntos directamente de `<pedido>` y del estado actual de
   `docs/gdd.md`/`docs/todo.md`. Si un punto es razonable de asumir (por ejemplo, "va al final"
   cuando nada sugiere lo contrario), infiérelo sin preguntar.
3. Si los 5 puntos quedaron resueltos por inferencia, no preguntes nada y continúa al paso 2.
4. Si falta uno o más, haz **una sola ronda** de preguntas concretas usando la herramienta de
   preguntas al usuario, cubriendo únicamente los puntos que faltan. No hagas una segunda ronda de
   seguimiento: con las respuestas de esa ronda, resuelve el checklist completo usando tu propio
   criterio para cualquier detalle menor que quede suelto.
5. Nunca dejes un punto del checklist sin resolver antes de continuar — si el usuario no contesta
   algo con precisión, toma la decisión más razonable tú mismo y sigue adelante.

### 2. Actualizar `docs/gdd.md` si hace falta

Si la fase introduce una mecánica, sistema o regla que **no** está descrita en `docs/gdd.md` (una
mecánica core nueva, un cambio a la condición de victoria/derrota, un control nuevo, etc.), actualiza
`docs/gdd.md` primero para reflejarla, en la sección que corresponda (`Mecánicas core`, `Controles`,
etc.) — el GDD nunca puede quedar desactualizado respecto a lo que las fases implementan. Si la fase
es solo contenido/pulido sobre mecánicas ya descritas, no toques `docs/gdd.md`.

### 3. Agregar la fase a `docs/todo.md`

Determina el número de fase: si va al final, es `N+1` donde `N` es la última fase existente. Si va
insertada en una posición específica (paso 1, punto 4), renumera las fases siguientes y sus
referencias en `docs/test-cases.md` (secciones `## Fase X: ...`) para mantener la secuencia
consistente — no dejes números de fase duplicados ni salteados.

Agrega la fase con el mismo formato que las existentes:

```markdown
## Fase N: <nombre de la fase>
- [ ] Tarea concreta
- [ ] Tarea con un asset final pendiente (HUMAN)
```

Reglas, iguales a las de `/new-game`:
- Cada tarea es concreta y verificable con un checkbox markdown.
- Si una tarea incluye algo que solo el usuario puede terminar (arte final, audio con licencia, una
  decisión de negocio/monetización), márcala con `(HUMAN)` al final de la línea. No abuses de esta
  marca — casi todo (incluyendo arte y audio placeholder) es agent-doable.
- La fase agrega una capa completa y coherente, nunca deja algo a medias.

### 4. Agregar los test cases a `docs/test-cases.md`

Agrega la sección correspondiente a la nueva fase, en la misma posición relativa que ocupa en
`docs/todo.md`, con el mismo formato que las existentes:

```markdown
## Fase N: <nombre de la fase>
- [ ] Como jugador, <afirmación observable> → `test_<nombre>`
```

Mismas reglas que en `/new-game`: cada historia testable se lee como intención de diseño (qué puede
hacer o percibir el jugador), es 100% verificable por código, y mapea 1:1 con un método `test_*` en
`game/tests/unit/`. Si una idea de test cae en lo subjetivo ("se ve bien", "se siente bien"), no la
incluyas.

### 5. Resumen final

Al terminar, muestra un resumen breve (no el contenido completo de los archivos) de: nombre y
número de la fase agregada, si se actualizó `docs/gdd.md` y por qué, y en qué posición quedó dentro
de `docs/todo.md`. Recuerda al usuario que `/start-phase` va a tomar las fases pendientes en orden,
así que si esta fase no es la siguiente a implementar, no se va a ejecutar todavía.
</content>
