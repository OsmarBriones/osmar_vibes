---
description: Corre review-all y aplica los fixes sugeridos en un ciclo hasta que no haya más hallazgos (no commitea ni pushea)
---

Vas a dejar el working tree limpio de hallazgos de review, corriendo `/review-all` y aplicando sus
fixes sugeridos en un ciclo, hasta que un intento devuelva `APROBADO` o se agote el máximo de
reintentos. **Este comando nunca ejecuta `git commit` ni `git push`** — solo `/commit-and-push`
está autorizado para eso (ver `.claude/CLAUDE.md`). Es un paso previo opcional para llegar limpio a
ese gate; `/commit-and-push` sigue siendo quien decide si el commit procede.

Argumentos recibidos: `$ARGUMENTS` (opcionalmente `--all`, que se pasa tal cual a `/review-all` en
cada intento).

## 1. Precondición

Corre `git status`. Si no hay cambios (ni staged, ni unstaged, ni untracked) y `$ARGUMENTS` no trae
`--all`, avisa que no hay nada que revisar y detente. Si trae `--all`, seguí igual (se revisa todo
el código relevante, no solo el diff).

## 2. Leer política de reintentos

Lee `.claude/review-policy.jsonc` si existe. Usá su clave `max_intentos` como tope de este ciclo;
si el archivo no existe, no tiene esa clave, o está mal formado, usá **5** por defecto.

## 3. Ciclo de review y fix

Repetí lo siguiente, llevando la cuenta del número de intento (arranca en 1):

1. Invocá `/review-all` (pasando `--all` si vino en `$ARGUMENTS`) sobre el estado actual del repo.
2. Si el veredicto agregado es `APROBADO`: el ciclo terminó con éxito, andá al paso 4.
3. Si es `RECHAZADO`: leé el reporte consolidado y aplicá vos mismo, directamente sobre los
   archivos correspondientes, todos los fixes sugeridos por cada reviewer que encontró hallazgos —
   no relances subagentes para esto, y no le preguntes al usuario si querés aplicarlos (ese es el
   propósito de este comando: aplicarlos en loop sin intervención). Si un fix sugerido toca lógica
   de `game/` (no solo estilo/docs) y hay tests de GUT relevantes disponibles, corré la suite (ver
   `.claude/CLAUDE.md`, sección Testing) antes de seguir al siguiente intento, para no introducir
   una regresión mientras corregís el hallazgo.
4. Si aplicaste fixes y todavía no llegaste al tope de `max_intentos`, sumá 1 al contador de
   intento y volvé al punto 1 (nuevo `/review-all` sobre el estado ya corregido). Si llegaste al
   tope sin haber conseguido `APROBADO`, salí del ciclo igual — el paso 4 reporta este caso como
   fallido.

## 4. Resultado

Reportá al usuario:

- Si terminó en `APROBADO`: en qué intento (de cuántos posibles), qué reviewers corrieron, y un
  resumen de qué hallazgos se corrigieron a lo largo del ciclo (si hubo alguno). Aclará que el
  working tree está listo para `/commit-and-push` pero que ese paso no se ejecutó — es un comando
  aparte.
- Si se agotó `max_intentos` sin `APROBADO`: decilo explícitamente, mostrá el reporte consolidado
  del último intento (qué reviewer(s) siguen rechazando y por qué), y aclará que necesita revisión
  humana — no reintentes más allá del tope vos mismo.

No marques nada en `docs/todo.md` ni `docs/test-cases.md` — eso es responsabilidad de `/start-phase`,
no de este comando.
