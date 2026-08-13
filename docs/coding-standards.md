# Coding Standards

Reglas concretas de GDScript/Godot 4 para este proyecto, basadas en la
[guía de estilo oficial de GDScript](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html)
y las [mejores prácticas oficiales de Godot](https://docs.godotengine.org/en/stable/tutorials/best_practices/index.html).
No son sugerencias — es el único estilo válido en este proyecto, y `/start-phase` debe seguirlo
al pie de la letra sin desviarse por preferencia propia.

## 1. Tipado

Siempre usar tipado estático explícito en variables, parámetros y valores de retorno.

```gdscript
var health: int = 100

func take_damage(amount: int) -> void:
	health -= amount
```

- Nunca declarar una variable con `var nombre = valor` si el tipo se puede anotar.
- Usar inferencia (`:=`) solo cuando el tipo del lado derecho es obvio y no ambiguo
  (`var speed := 300.0`), nunca cuando el valor inicial pueda confundir el tipo real.
- Toda función pública declara su tipo de retorno, incluyendo `-> void`.

## 2. Nomenclatura

- Archivos y escenas: `snake_case.gd`, `snake_case.tscn`.
- Clases (`class_name`) y nodos en el árbol de escena: `PascalCase`.
- Funciones y variables: `snake_case`.
- Constantes y valores de enum: `SCREAMING_SNAKE_CASE`.
- Miembros privados (uso interno de la clase, no pensados para acceso externo): prefijo `_`
  (`_internal_state`).
- Señales: en pasado o como evento, nunca con prefijo `on_` al declararlas
  (`health_depleted`, no `on_health_depleted`).
- Handlers de señal conectados: prefijo `_on_<Emisor>_<señal>` (`_on_attack_button_pressed`).

## 3. Orden de miembros dentro de una clase

Siempre en este orden (el orden en que Godot ejecuta los callbacks de ciclo de vida se respeta
dentro del bloque 10):

1. `class_name` / `extends`
2. Docstring de la clase (`##`) si la clase es reutilizable
3. `signal`
4. `enum`
5. `const`
6. Variables `@export`
7. Variables públicas
8. Variables privadas (prefijo `_`)
9. Variables `@onready`
10. Callbacks de Godot (`_ready`, `_process`, `_physics_process`, `_input`, `_unhandled_input`...)
11. Métodos públicos
12. Métodos privados (prefijo `_`)

## 4. Referencias entre nodos y señales

- Para comunicar nodos sin relación padre-hijo directa, usar señales — nunca una referencia
  guardada a mano a un nodo "hermano" o lejano en el árbol.
- Prohibido `get_node("../../OtroNodo")` o cualquier ruta relativa de más de un nivel: es frágil
  ante cualquier reordenamiento de la escena. Para hijos directos, usar nodos marcados como
  "Unique Name" (`%NombreUnico`) y cachear la referencia en `@onready`:

  ```gdscript
  @onready var health_bar: ProgressBar = %HealthBar
  ```

- Evitar herencia de escenas de más de 2 niveles (`extends "res://otra_escena.tscn"`) salvo que
  `docs/gdd.md` lo pida explícitamente. Preferir composición (nodos hijos, señales) sobre
  herencia profunda.

## 5. Autoloads (singletons)

Usar autoloads únicamente para:
- Gestión/transición de escenas.
- Audio global (música, buses).
- Configuración de input remapeable.
- Estado que sobrevive al cambio de escena (progreso guardado, settings).

Nunca usar un autoload como bolsa genérica de variables compartidas entre nodos que sí tienen
una relación de escena resoluble con señales o referencias directas.

## 6. Datos de diseño

- Ningún valor de diseño (velocidad, vidas, daño, tiempos, distancias) va hardcodeado dentro de
  la lógica. Debe ser una variable `@export` en el nodo que lo define.
- Cuando el mismo dato se repite entre varias instancias (stats de distintos tipos de enemigo,
  configuración de niveles), modelarlo como una clase `Resource` custom (`class_name` + `.tres`),
  nunca como un `Dictionary` suelto.

## 7. Testeabilidad

- La lógica de juego que no depende directamente del árbol de escena (cálculos, reglas de estado)
  debe vivir en clases que NO extiendan `Node`/`Node2D` cuando sea posible — extender
  `RefCounted` — para poder testearla con GUT sin instanciar la escena completa.
- Cuando la lógica sí debe vivir en un `Node` (por señales de física, ciclo de vida), el método
  invocado desde `_process`/`_physics_process` debe ser una llamada delgada a un método público
  puro y testeable por separado, no lógica inline dentro del callback.

## 8. Performance

- Prohibido `get_node()`, `load()` o `preload()` dinámico dentro de `_process`,
  `_physics_process`, o cualquier función invocada por frame. Toda referencia se cachea en
  `@onready` o se precarga en `_ready`.
- `_physics_process` para todo lo relacionado a movimiento y colisiones. `_process` solo para lo
  puramente visual (animación no física, actualización de UI).

## 9. Comentarios

- Sin comentarios que describan QUÉ hace el código — los nombres de variables/funciones ya lo
  dicen. Comentar solo el POR QUÉ cuando hay una razón no obvia (limitación de Godot, workaround,
  decisión de diseño que no se explica sola).
- La API pública de una clase reutilizable se documenta con un docstring `##` de una línea sobre
  la declaración, no con bloques largos.

## 10. Manejo de errores

- `assert()` para invariantes que nunca deberían romperse durante el desarrollo.
- `push_error()` / `push_warning()` para condiciones inesperadas en runtime que no deben crashear
  el juego.
- Ningún fallo silencioso sin justificación: un `if node == null: return` solo es válido cuando
  el caso es esperado (ej. un nodo opcional que puede no existir) — en ese caso el propio código
  debe dejar claro por qué es opcional (nombre de variable, comentario corto si no es obvio).

## 11. Formato

- Indentación con tabs, nunca espacios (convención oficial de GDScript).
- Una declaración por línea, sin `;` para separar statements.
- **Máximo 100 caracteres por línea. Regla dura, sin excepciones** — si una línea no entra,
  se reestructura (variables intermedias, quiebre de la expresión), nunca se deja pasar.

## 12. Tamaño de archivo

- **Máximo 300 líneas por archivo `.gd`. Regla dura.** Un archivo que se acerca a ese límite es
  una señal de que está haciendo más de una cosa — no un caso a ignorar ni a resolver comprimiendo
  código para que "entre".
- Al llegar al límite, separar en módulos según responsabilidad: extraer una clase interna a su
  propio archivo con `class_name`, mover un sub-sistema completo a un script separado que el
  original usa por composición, o partir la escena en sub-escenas más chicas con su propio script.
  Nunca partir un archivo a la mitad de forma arbitraria solo para bajar el conteo de líneas.
- Este límite aplica por archivo, no por clase: si una clase legítimamente necesita más de 300
  líneas, es una señal de que esa clase misma debe dividirse en colaboradores más chicos.
