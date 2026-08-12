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

### 1. Clarificación mínima

Si hay ambigüedades importantes que cambiarían sustancialmente el alcance o las mecánicas core
(por ejemplo: cámara 2D vs 3D, single player vs multiplayer, plataformas objetivo), haz como
máximo 3-4 preguntas concretas usando la herramienta de preguntas al usuario. Si la idea ya es
razonablemente clara, no preguntes nada y avanza directo — prioriza no bloquear el flujo.

### 2. Generar `docs/gdd.md`

Crea (o sobrescribe si ya existe) `docs/gdd.md` con esta estructura, todo en español, tan
específico como sea posible (nombres de mecánicas concretas, números de referencia cuando aplique
como velocidades o vidas, no solo descripciones vagas):

```markdown
# GDD: <Nombre del juego>

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
