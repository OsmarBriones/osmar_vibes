# Architecture

Este documento define cómo se descompone la lógica de cualquier juego generado por este template
en **sistemas** — no cuáles sistemas existen (eso lo define `docs/gdd.md` juego a juego), sino
el criterio para decidir cómo se construye cada uno. No es una sugerencia: `/start-phase` sigue
este criterio al implementar cada tarea, igual que sigue `docs/coding-standards.md`.

## Qué es un sistema

Un sistema es una pieza de lógica de juego con una responsabilidad concreta y un límite claro
(vida, combate, inventario, puntaje, spawneo de enemigos, lo que sea que pida el GDD). Un sistema
nunca conoce los detalles internos de otro — solo se comunica hacia afuera mediante señales
(emitidas) y una API pública de métodos (invocada).

## Los dos patrones válidos

Godot admite dos formas de estructurar un sistema. Ambas son válidas — la decisión depende de una
sola pregunta: **¿el estado de este sistema varía por entidad, o es único en todo el juego?**

### 1. Manager global (autoload)

Un singleton que existe una sola vez y opera sobre cualquier entidad que se le pase.

**Usar cuando** el sistema no tiene estado por-entidad — hay un solo score, una sola pausa, una
sola transición de escena activa a la vez.

**Ejemplos de este tipo de sistema** (genéricos, no ligados a un juego en particular): gestión de
escenas, audio global, input remapeable, score/progreso global, pausa.

```
game/scripts/systems/<nombre>_system.gd   # class_name <Nombre>System, autoload
```

### 2. Componente por entidad (estilo ECS liviano)

Un nodo hijo que cada entidad que lo necesita agrega a su propia escena. El mismo componente se
reutiliza entre entidades distintas (el jugador y un enemigo pueden compartir `HealthComponent`).

**Usar cuando** el sistema tiene estado que es propio de cada instancia — cada entidad tiene su
propia vida, su propio estado de combate, su propio inventario.

**Ejemplos de este tipo de sistema** (genéricos): vida, combate, movimiento, inventario, IA de
un enemigo puntual.

```
game/scenes/components/<nombre>_component.tscn + .gd   # class_name <Nombre>Component
```

### Criterio de decisión

1. ¿Puede haber más de una instancia de este sistema activa a la vez, cada una con su propio
   estado (una vida por personaje, no una vida global)? → Componente.
2. ¿Hay exactamente una instancia en todo el juego, sin importar cuántas entidades existan? →
   Autoload.
3. Si un sistema empieza como autoload y una fase futura necesita una segunda instancia
   independiente (ej. "ahora dos jugadores necesitan su propia vida"), se migra a componente en
   esa fase — no se fuerza el patrón original solo por no tocarlo.

## Comunicación entre sistemas

- Un componente nunca llama directamente a un método de otro componente de una entidad distinta
  ni a otro sistema — emite una señal, y quien esté interesado se suscribe.
- Un autoload puede exponer métodos públicos que cualquier componente invoque directamente (por
  ejemplo, `ScoreSystem.add_points(10)`), ya que su rol es justamente ser un punto único de acceso.
- Ningún sistema hace `get_node()` para alcanzar a otro sistema por ruta de escena — un componente
  que necesita un autoload lo referencia por su nombre de autoload; un autoload que necesita
  reaccionar a un componente se suscribe a su señal cuando la entidad se registra.

## Testeabilidad

Cada sistema (autoload o componente) implementa su lógica core en una clase que no dependa del
árbol de escena cuando sea posible (ver `docs/coding-standards.md` §7), con su propio
`game/tests/unit/test_<nombre>_system.gd` o `test_<nombre>_component.gd`. Un sistema sin test
propio no está terminado, sin importar si el resto de la fase pasa.

## Dónde vive cada sistema

```
game/
  scripts/systems/       # managers globales (autoload), un archivo por sistema
  scenes/components/      # componentes por entidad, escena + script por componente
  tests/unit/              # un test por sistema, sin importar el patrón
```

Esto es una especialización de la estructura ya definida en `.claude/CLAUDE.md` — `systems/` y
`components/` son subcarpetas de `scripts/` y `scenes/` respectivamente, no carpetas nuevas al
mismo nivel.
