# Godot + Claude Code Vibecoding Template

Template para desarrollar juegos en [Godot 4](https://godotengine.org/) (GDScript) guiado por
[Claude Code](https://claude.com/claude-code), mediante un flujo de fases con tests automatizados.

## Cómo funciona

1. Abre este proyecto con Claude Code.
2. Corre `/new-game <idea del juego>` con una descripción de tu idea, por ejemplo:

   ```
   /new-game un roguelike top-down donde controlas una espada que vuela sola
   ```

   Esto genera tres documentos en `docs/`:
   - `gdd.md` — el diseño detallado del juego.
   - `todo.md` — las fases de desarrollo (la Fase 1 siempre es un vertical slice jugable).
   - `test-cases.md` — los casos de prueba automatizados por fase.

3. Corre `/start-phase` para implementar la siguiente fase pendiente. El comando implementa el
   código, escribe/actualiza los tests, corre la suite completa con
   [GUT](https://github.com/bitwes/Gut), y solo marca la fase como terminada si todos los tests
   pasan.
4. Repite `/start-phase` hasta terminar todas las fases de `todo.md`.

## Requisitos

- [Claude Code](https://claude.com/claude-code) instalado.
- [Godot 4.x](https://godotengine.org/download) — descarga la versión portable (`.exe`/`.zip`,
  sin instalador) y colócala en una carpeta `.tools/` en la raíz del proyecto (ya está en
  `.gitignore`), o simplemente déjala en tu `PATH` como `godot`.

## Estructura del proyecto

```
.claude/
  commands/       # los slash commands /new-game y /start-phase
  CLAUDE.md        # convenciones del proyecto (estilo de código, estructura, testing)
game/
  addons/gut/     # framework de testing
  scenes/         # escenas .tscn
  scripts/        # scripts compartidos (autoloads, utilidades)
  assets/         # sprites, audio
  tests/unit/     # tests GUT, uno por área funcional
docs/
  gdd.md               # documento de diseño del juego (generado por /new-game)
  todo.md              # fases de desarrollo (generado por /new-game)
  test-cases.md        # casos de prueba por fase (generado por /new-game)
  coding-standards.md  # el estilo de código GDScript del proyecto
```

## Testing

Los tests corren con [GUT](https://github.com/bitwes/Gut) en modo headless:

```
godot --headless --path game -s res://addons/gut/gut_cmdln.gd -gconfig=.gutconfig.json
```

(La primera vez, o si Godot no reconoce las clases del addon, corre antes
`godot --headless --path game --import`.)

## Licencia

MIT — usa este template libremente para tus propios proyectos.
