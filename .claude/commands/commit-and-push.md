---
description: Único camino autorizado para hacer commit y push — corre todos los reviews y solo procede si aprueban
---

Este es el **único workflow autorizado para hacer `git commit` y `git push`** en este proyecto.
Ningún otro comando o skill debe ejecutar esos comandos directamente, salvo que:

- ese comando lo declare explícitamente en sus propias instrucciones (documentando por qué se
  salta este gate), o
- el usuario lo pida explícitamente en el chat, fuera de la ejecución automatizada de un comando.

Si estás ejecutando cualquier otro comando de este proyecto y llegás al punto de commitear o
pushear, invocá `/commit-and-push` en vez de correr `git commit`/`git push` vos mismo.

Argumentos recibidos: `$ARGUMENTS` (opcionalmente: un mensaje de commit explícito, `--all` para
pasarlo al review, y/o `--skip-reviews`).

## `--skip-reviews`

Esta bandera salta por completo el paso 2 (`/review-all`) y va directo al commit/push del paso 4,
sin veredicto de por medio.

**Restricción dura: ningún workflow o comando de este proyecto puede pasar `--skip-reviews` por su
cuenta**, ni siquiera `/start-phase` u otro que invoque `/commit-and-push` internamente. Solo es
válida cuando:

- el **usuario** la escribe explícitamente en el chat (p. ej. `/commit-and-push --skip-reviews`,
  o le pide en texto plano "saltate los reviews y commitea"), o
- otro comando la usa porque **su propio archivo de instrucciones lo declara explícitamente**,
  citando por qué se justifica saltar el gate en ese caso puntual — a la fecha, ningún comando de
  este repo declara eso.

Si estás invocando `/commit-and-push` desde otro comando (por ejemplo `/start-phase`) y no hay una
instrucción explícita de esas dos en el archivo que te está guiando, **no pases `--skip-reviews`**
aunque parezca conveniente para evitar un rechazo — corré el review igual.

## 1. Precondición

Corre `git status`. Si no hay cambios (ni staged, ni unstaged, ni untracked) para commitear, avisa
que no hay nada que hacer y detente.

## 2. Correr el review

Si `$ARGUMENTS` contiene `--skip-reviews` (y aplica según la restricción de arriba), saltá este
paso por completo: no invoques `/review-all`, y tratá el resultado como si el veredicto fuera
`APROBADO (reviews saltados por --skip-reviews)` — dejá esa aclaración explícita en el resultado
final del paso 5, no la ocultes.

Si no, invocá `/review-all` (pasando `--all` si vino en `$ARGUMENTS`) sobre el estado actual del
repo. Ese comando devuelve un reporte consolidado terminado en una línea `Veredicto: APROBADO` o
`Veredicto: RECHAZADO`.

## 3. Si el veredicto es RECHAZADO

No ejecutes `git commit` ni `git push` bajo ninguna circunstancia. Devuelve como resultado de este
comando: el veredicto `RECHAZADO` y el reporte consolidado completo (qué reviewer(s) rechazaron y
por qué), para que quien te invocó (el usuario, o el comando que llamó a `/commit-and-push`, p. ej.
`/start-phase`) decida cómo corregirlo. Tu trabajo termina acá.

## 4. Si el veredicto es APROBADO

1. Determiná el mensaje de commit: si `$ARGUMENTS` trae uno explícito, usalo (siguiendo el formato
   `Fase N: <título>` cuando la convención de `.claude/CLAUDE.md` aplique); si no, redactalo en
   base al diff.
2. Agregá al stage solo los archivos relevantes a este cambio (nunca `git add -A`/`git add .` a
   ciegas — revisá qué se está agregando).
3. Hacé el commit con ese mensaje.
4. Hacé `git push` de la rama actual. Si la rama no tiene upstream configurado, usá
   `git push -u origin <rama-actual>`.
5. Confirmá con `git status`/`git log -1` que el commit y el push se completaron.

## 5. Resultado

Reportá: veredicto (`APROBADO`/`RECHAZADO`, o `APROBADO (reviews saltados por --skip-reviews)` si
aplicó), qué reviewers participaron (o que no corrió ninguno), y si hubo commit/push, el hash del
commit y la rama a la que se pusheó.
</content>
