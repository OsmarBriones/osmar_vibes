---
description: Genera gdd.md, todo.md y test-cases.md a partir de una idea de juego
---

El usuario quiere iniciar un nuevo proyecto de juego a partir de esta idea:

<idea>
$ARGUMENTS
</idea>

## Si la idea está vacía

Si `<idea>` está vacía o no contiene información suficiente para identificar un tipo de juego
(género, mecánica o premisa mínima), NO continúes con los siguientes pasos. Responde únicamente
pidiendo que se vuelva a ejecutar el comando incluyendo la idea, por ejemplo:
`/new-game un roguelike top-down donde controlas una espada que vuela sola`. No generes ningún
archivo en este caso.

## Si la idea es válida

Sigue estos pasos en orden:

### 1. Checklist de idea lista

Una idea está lista para generar el GDD cuando estos 5 puntos tienen una respuesta concreta.
Este es el único criterio de "listo" — no busques más precisión que esta, el resto se resuelve
durante `/start-phase`:

1. **Mecánica core** — qué hace el jugador momento a momento (el verbo principal del juego).
2. **Cámara/perspectiva** — 2D top-down, lateral, isométrico, 3D, etc.
3. **Condición de victoria/derrota o tipo de loop** — cómo se gana/pierde, o si es un loop
   infinito tipo arcade/score/supervivencia sin final formal.
4. **Controles principales** — qué inputs usa el jugador y para qué.
5. **Qué queda fuera del MVP** — al menos una exclusión explícita de alcance.

Proceso:

1. Lee `<idea>` e intenta **inferir** cada uno de los 5 puntos directamente de lo que escribió el
   usuario. Si un punto es razonable de asumir (por ejemplo, "un platformer" implica cámara lateral
   2D), infiérelo sin preguntar — no lo marques como pendiente solo por no estar dicho literal.
2. Si los 5 puntos quedaron resueltos por inferencia, no preguntes nada y continúa al paso 2.
3. Si falta uno o más, haz **una sola ronda** de preguntas concretas usando la herramienta de
   preguntas al usuario, cubriendo únicamente los puntos que faltan (nunca preguntes por algo ya
   inferido). No hagas una segunda ronda de seguimiento: con las respuestas de esa ronda, resuelve
   el checklist completo usando tu propio criterio para cualquier detalle menor que quede suelto.
4. Nunca dejes un punto del checklist sin resolver antes de continuar — si el usuario no contesta
   algo con precisión, toma la decisión más razonable tú mismo y sigue adelante.

### 2. Generar `docs/gdd.md`

Crea (o sobrescribe si ya existe) `docs/gdd.md` con esta estructura, todo en español, tan
específico como sea posible (nombres de mecánicas concretas, números de referencia cuando aplique
como velocidades o vidas, no solo descripciones vagas):

```markdown
# GDD: <Nombre del juego>

## Checklist de idea lista
- [x] Mecánica core: <resumen de una línea>
- [x] Cámara/perspectiva: <resumen de una línea>
- [x] Condición de victoria/derrota o loop: <resumen de una línea>
- [x] Controles principales: <resumen de una línea>
- [x] Fuera del MVP: <resumen de una línea>

## Pilares de diseño
(3-5 frases que definen qué hace único a este juego y qué NO es)

## Premisa
(2-4 párrafos)

## Cámara y perspectiva

## Mecánicas core
(lista detallada, cada mecánica con: qué hace, cómo se controla, qué la hace divertida)

## Loop de juego
(qué hace el jugador segundo a segundo, minuto a minuto)

## Controles
(tabla input -> acción)

## Progresión / estructura de niveles

## Dirección de arte y audio
(referencias, mood, paleta — sin necesidad de assets finales)

## Alcance del MVP
(qué queda explícitamente FUERA de la primera versión completa)
```

### 3. Generar `docs/todo.md`

Crea (o sobrescribe) `docs/todo.md` con fases numeradas. Reglas obligatorias:

- **La Fase 1 es siempre un vertical slice jugable**: la mecánica core mínima, de principio a fin,
  jugable, sin pulido, sin arte final, sin menús. El objetivo es tener algo jugable lo antes posible
  sobre lo que las siguientes fases construyen.
- Cada fase siguiente agrega una capa completa (una mecánica más, contenido, pulido, UI, etc.), nunca
  deja algo a medias entre fases.
- Cada fase tiene una lista de tareas concretas y verificables con checkboxes markdown.
- Formato:

```markdown
# TODO

## Fase 1: Vertical slice jugable
- [ ] Tarea concreta
- [ ] Tarea concreta

## Fase 2: <nombre>
- [ ] ...
```

### 4. Generar `docs/test-cases.md`

Crea (o sobrescribe) `docs/test-cases.md`. Por cada fase de `todo.md`, define los casos de prueba
automatizados (GUT) que confirman que la fase está terminada. Formato:

```markdown
# Test Cases

## Fase 1: Vertical slice jugable
- [ ] `test_<nombre>`: descripción de qué verifica y cuál es el resultado esperado
```

Cada test case debe ser algo verificable por código (estado de una variable, valor de retorno,
señal emitida, nodo presente en la escena), no "se ve bien" o "se siente bien".

### 5. Resumen final

Al terminar, muestra un resumen breve (no el contenido completo de los archivos) de: nombre del
juego, número de fases generadas, y recuerda al usuario que el siguiente paso es correr
`/start-phase` para comenzar con la Fase 1.
