---
description: Descarga e instala una copia portable de Godot en .tools/, detectando SO/arquitectura y la versión que necesita el proyecto
---

Vas a asegurar que haya un ejecutable de Godot utilizable para este proyecto, instalando una copia
portable en `.tools/` si hace falta. Sigue estos pasos en orden y no te saltes ninguno — este
comando trae y va a ejecutar un binario, no es una acción trivial.

### 0. Determinar la versión requerida

1. Si `game/project.godot` existe, leé la clave `config/features` de la sección `[application]` y
   tomá el primer valor con forma `"<major>.<minor>"` (por ejemplo `"4.7"` en
   `PackedStringArray("4.7", "GL Compatibility")`) — esa es la versión requerida.
2. Si `game/project.godot` no existe todavía (repo recién clonado, antes de `/new-game`), leé
   `.claude/godot-version.jsonc`, clave `version_default`, y usá ese valor. Si el archivo no
   existe, no tiene esa clave, o está mal formado, detenete y avisá al usuario que hace falta ese
   archivo o `game/project.godot` para determinar qué versión instalar — no inventes un número de
   versión.

`.claude/godot-version.jsonc` es la **única fuente de este valor por defecto** en el template — no
la dupliques hardcodeada en ningún otro lado.

### 1. Detectar SO y arquitectura

- **Windows**: `x86_64` (caso casi universal) vs `arm64`. Godot no publica build nativa `arm64`
  para la mayoría de las versiones 4.x — si detectás Windows ARM64, usá el build `win64` (x86_64,
  corre bajo emulación) y dejalo explícito en el resumen final.
- **macOS**: el build `universal` sirve tanto para `x86_64` como `arm64`, no hace falta distinguir.
- **Linux**: `x86_64` vs `arm64`.

### 2. Buscar un Godot ya utilizable

1. Probá `godot --version` (PATH del sistema). Si existe y su versión mayor.menor coincide con la
   requerida (paso 0), reportá que ya hay un Godot válido en el PATH y **detenete acá** — no
   descargues nada.
2. Si no, buscá un ejecutable `Godot*` dentro de `.tools/` en la raíz del repo (`Godot*.exe` en
   Windows). Si encontrás uno y `<godot> --version` coincide en mayor.menor con la requerida,
   reportá que ya está instalado y **detenete acá**.
3. Si encontrás un `Godot*` en `.tools/` pero de una versión **distinta** a la requerida, no lo
   borres ni lo sobreescribas bajo ninguna circunstancia — puede ser una versión que el usuario
   dejó ahí a propósito. Continuá al resto de los pasos, pero en el paso 3 (confirmación) y en el
   resumen final dejá explícito que ya existe otra versión ahí y que vas a instalar la nueva con un
   nombre de archivo distinto (el .zip de cada versión ya viene nombrado con su versión, así que no
   se pisan). Si esto va a dejar más de un ejecutable `Godot*.exe` en `.tools/`, advertí además que
   `.claude/CLAUDE.md` no define cuál de los dos toma `/start-phase` si ambos matchean el patrón —
   sugerí al usuario borrar manualmente el que no quiera conservar.

### 3. Confirmar antes de descargar

Nunca descargues sin confirmación explícita del usuario, incluso si esta es la primera vez que se
corre este comando en la máquina. Antes de pedirla:

1. Consultá la API pública de GitHub Releases
   (`https://api.github.com/repos/godotengine/godot/releases/tags/<version>-stable`) para obtener
   la lista real de assets de esa versión — no adivines nombres de archivo, varían entre versiones.
   Si GitHub no responde, probá como fuente alternativa `downloads.tuxfamily.org/godotengine/<version>/`.
2. De esa lista, identificá el asset que corresponde al SO/arquitectura del paso 1 (patrones
   típicos: `Godot_v<version>-stable_win64.exe.zip`, `_win32.exe.zip`, `_macos.universal.zip`,
   `_linux.x86_64.zip`, `_linux.arm64.zip` — pero usá el nombre real que devuelva la API, no estos
   literales).
3. Identificá también el asset de checksums (típicamente con "sha512" en el nombre). Si no existe
   uno publicado para esa versión, seguí sin checksum pero decilo explícitamente en la confirmación
   y en el resumen final — nunca lo ocultes.

Pedí confirmación mostrando: versión exacta, plataforma/arquitectura detectada, nombre y tamaño
(campo `size` de la API) del archivo a descargar, y su URL. Si el usuario no confirma, detenete sin
descargar nada.

### 4. Descargar y verificar checksum

1. Descargá el asset a un archivo temporal en el directorio de scratchpad de la sesión (no
   directamente a `.tools/`), para no dejar una extracción a medio hacer si algo falla a mitad de
   camino.
2. Si hay checksum disponible (paso 3.3), calculá el SHA512 del archivo descargado
   (`Get-FileHash -Algorithm SHA512` en Windows, `shasum -a 512` en Mac/Linux) y comparalo contra el
   valor publicado. Si no coincide, detenete, borrá el archivo descargado, y avisá al usuario con un
   mensaje claro de qué falló — no continúes instalando un binario que no pudo verificarse.
3. Si la descarga falla por red/timeout/DNS, no falles en silencio: mostrá el error concreto y
   pasá a la sección "Si falla la descarga" al final de este documento. No reintentes
   indefinidamente por tu cuenta.

### 5. Extraer a `.tools/`

1. Extraé el contenido del .zip a `.tools/` en la raíz del repo (creá la carpeta si no existe;
   `.tools/` ya está en `.gitignore`, no cambia nada ahí).
2. **Windows**: el ejecutable extraído ya empieza con `Godot`, que es el patrón que espera
   `.claude/CLAUDE.md` (`Godot*.exe`) — no hace falta renombrarlo.
3. **Mac**: el .zip trae un bundle `Godot.app` — el binario ejecutable dentro es
   `.tools/Godot.app/Contents/MacOS/Godot`; dale permisos de ejecución (`chmod +x`).
4. **Linux**: el binario extraído (`Godot_v<version>-stable_linux.<arch>`) necesita permisos de
   ejecución (`chmod +x`).
5. Borrá el .zip temporal descargado en el paso 4.

### 6. Validar

Corré `<godot> --version` sobre el ejecutable recién extraído y confirmá que el mayor.menor
coincide con la versión requerida (paso 0). Si no coincide o el comando falla, avisá al usuario que
la instalación no se pudo validar — no la des por buena en el resumen final.

### 7. Resumen final

Reportá: versión instalada (o ya presente) y de dónde salió (PATH / ya estaba en `.tools/` /
recién descargada), plataforma/arquitectura detectada, si hubo verificación de checksum o no, y
cualquier advertencia relevante (arquitectura emulada, otra versión ya presente en `.tools/`,
checksum no disponible para esta versión). Este comando **nunca** hace `git commit` ni `git push`
— `.tools/` está en `.gitignore`, no hay nada que commitear.

## Si falla la descarga

Si hay un error de red, timeout, o ni la API ni la descarga de GitHub/tuxfamily responden, no
falles en silencio: mostrá el error concreto tal como se dio, y recordá el fallback manual —
instalar Godot a mano desde https://godotengine.org/download y copiar el ejecutable a `.tools/`
con nombre `Godot*.exe` (Windows) o el binario correspondiente con permisos de ejecución
(Mac/Linux), tal como documenta `.claude/CLAUDE.md` sección "Testing (GUT)". Este comando no
reintenta la descarga indefinidamente por su cuenta — informa y se detiene.
