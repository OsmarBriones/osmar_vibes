---
description: Corre todos los reviewers disponibles (arquitectura, coding-standards, docs) en paralelo y consolida el reporte
---

Vas a correr **todos los reviewers disponibles** del proyecto en paralelo, cada uno como un
subagente independiente, y a consolidar sus reportes en uno solo. Esta skill no revisa nada por su
cuenta — delega en los reviewers existentes para no duplicar su lógica.

Argumentos recibidos: `$ARGUMENTS`

## 1. Descubrir reviewers disponibles

Busca todos los archivos `.claude/commands/review-*.md` **excepto este mismo archivo**
(`review-all.md`). Cada uno encontrado es un reviewer candidato — hoy son
`review-architecture.md`, `review-coding-standards.md` y `review-docs.md`, pero no los
hardcodees: si en el futuro se agrega un `review-*.md` nuevo, esta skill debe recogerlo
automáticamente sin que nadie la edite.

Luego lee `.claude/review-policy.jsonc` si existe. Si tiene una clave `reviewers_activos` (lista de
nombres de archivo sin extensión, p. ej. `["review-architecture", "review-docs"]`), filtra la lista
de candidatos a solo esos — un reviewer que exista como archivo pero no esté en
`reviewers_activos` **no se corre**. Si el archivo de política no existe, no tiene esa clave, la
lista está vacía, o está mal formado, usa **todos** los candidatos encontrados (comportamiento por
defecto, sin filtrar).

Si tras aplicar el filtro no queda ningún reviewer para correr, avisa al usuario (aclarando si fue
por falta de reviewers en el repo o por la política) y detente.

## 2. Lanzar un subagente por reviewer, en paralelo

Para cada reviewer encontrado, lanza un subagente (tipo `general-purpose`) **en el mismo mensaje**
que los demás, para que corran en paralelo. El prompt de cada subagente debe:

- Indicarle que lea el archivo del reviewer (ruta completa) y siga sus instrucciones al pie de la
  letra, tratando `$ARGUMENTS` como si fuera el argumento con el que se invocó ese comando (por
  ejemplo, si el usuario pasó `--all` a `/review-all`, cada subagente debe correr su reviewer con
  `--all` también).
- Aclararle explícitamente que **no** pregunte nada al usuario ni aplique fixes por su cuenta —
  solo debe devolver su reporte de hallazgos como texto final (esa confirmación se hace una sola
  vez, consolidada, en el paso 4), y que ese reporte debe terminar con la línea de veredicto exacta
  (`Veredicto: APROBADO` o `Veredicto: RECHAZADO`) que su propio archivo de instrucciones le pide
  emitir.
- Pedirle que estructure su respuesta como: reviewer, alcance revisado, lista de hallazgos
  (archivo:línea, regla incumplida, fix sugerido) o "sin hallazgos" si no encontró nada, y la línea
  de veredicto al final.

Espera a que los subagentes de este paso terminen antes de continuar.

## 3. Consolidar

Junta los reportes de todos los reviewers en un único resultado, agrupado por reviewer, y dentro de
cada uno ordenado de más a menos severo. No mezcles hallazgos de distintos reviewers entre sí — cada
uno queda bajo su propio encabezado.

Calcula el **veredicto agregado**: `RECHAZADO` si al menos un reviewer devolvió `RECHAZADO` o no
emitió veredicto (trátalo como rechazo por seguridad); `APROBADO` solo si todos los reviewers
encontrados en el paso 1 devolvieron `APROBADO`. Escribe ese veredicto agregado como la última
línea del resultado, en el mismo formato exacto (`Veredicto: APROBADO` / `Veredicto: RECHAZADO`) —
es lo que consumen otros comandos como `/commit-and-push` que invocan `/review-all` en vez de
correr los reviewers uno por uno.

## 4. Reportar y preguntar una sola vez

Muestra el reporte consolidado al usuario, con el veredicto agregado bien visible. Si al menos un
reviewer encontró hallazgos, pregunta una única vez (no una por reviewer) si quiere que se apliquen
los fixes sugeridos, y de ser así aplicá los cambios vos directamente sobre los archivos
correspondientes — no relances subagentes para esto. Si ningún reviewer encontró nada, dilo
explícitamente y no preguntes nada más.

Si te está invocando otro comando de forma automatizada (por ejemplo `/commit-and-push`), seguí sus
instrucciones en vez de preguntarle nada al usuario vos mismo: devolvé el reporte consolidado y el
veredicto agregado como tu resultado final.
</content>
