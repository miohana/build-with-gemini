---
name: publish-to-github
description: >-
  Publica el proyecto finalizado del participante en SU PROPIO GitHub personal y entrégale un formulario prellenado para reclamar swag y participar en la galería. Úsalo al final del lab cuando el usuario quiera "publicar mi proyecto en GitHub", "subir mi código a GitHub", "cargar mi agente/proyecto", "poner mi código en mi propio GitHub", "entregar mi proyecto", "enviar mi proyecto para el swag", o "entrar a la galería". Inicia sesión en el GitHub personal del usuario a través del flujo de dispositivos de la CLI gh (un código de un solo uso, sin necesidad de pegar claves SSH ni tokens), crea un repositorio PÚBLICO y hace push — pero SIEMPRE pide al usuario que confirme el nombre del repositorio y la cuenta antes de hacer push. Luego imprime un enlace de Google Forms con la URL del repositorio prellenada. Incluye ./publish.sh para las partes deterministas. No es para desplegar el agente (agents-cli) ni el frontend (Cloud Run) — esto publica el código fuente en GitHub.
---

# Publicar el proyecto en el GitHub personal del participante

El lab se ejecuta dentro de una **workstation efímera** — cuando el lab termina, la máquina y el código que contiene desaparecen. Esta skill lleva el proyecto del participante de forma segura a **su propio GitHub personal** (para que lo conserve y pueda compartirlo), y luego le entrega un **formulario de envío prellenado** para reclamar swag y participar en la galería de proyectos.

La clave para eliminar fricciones es el **flujo de dispositivos de GitHub CLI (device flow)**: el participante introduce un código de un solo uso en `github.com/login/device` desde *cualquier* dispositivo — sin claves SSH ni tokens que copiar y pegar. El token de `gh` vive y muere con la workstation; el repositorio permanece para siempre en su cuenta.

> **La regla principal — nunca hacer push sin confirmación explícita.** Esto crea un repositorio en el GitHub *personal* del participante. Antes de crear el repositorio o hacer push, DETENTE y obtén un "sí" explícito sobre el nombre del repositorio y la confirmación de que se trata de su cuenta. No permitas aprobaciones a ciegas — menciona el nombre y la cuenta claramente primero.

## Cómo funciona

```
archivos del proyecto ─▶ publish.sh prep ─▶ gh auth login (código de dispositivo) ─▶ publish.sh commit
                      ─▶ [CONFIRMAR con el usuario] ─▶ gh repo create --public --push ─▶ URL del repo
                      ─▶ publish.sh formlink <repo_url> ─▶ formulario de envío prellenado
```

`./publish.sh` realiza las partes deterministas y no interactivas (instalar `gh`, escribir `.gitignore`, escanear en busca de secretos, iniciar un historial de git limpio, construir el enlace del formulario). Las partes **interactivas** — el inicio de sesión por flujo de dispositivos y el push — son guiadas por el agente para que el participante apruebe explícitamente el push.

Ejecuta todo desde la **raíz del proyecto** (donde residen `app/`, `project_brief.md`, `agents-cli-manifest.yaml`). Idealmente el participante ya completó el paso "Comparte lo que construiste", de modo que ya existen un `README.md` y un GIF de demostración en el repositorio.

## Paso 1 — Preparación (instalar gh, stage, escanear). Sin commit, sin push.

```bash
bash .agents/skills_es/publish-to-github/publish.sh prep
export PATH="$HOME/.local/bin:$PATH"   # en caso de que prep acabe de instalar gh allí
```

Esto instala `gh` si no está presente (en `~/.local/bin`, sin sudo), escribe un `.gitignore` (ignora `.venv/`, `__pycache__/`, `node_modules/`, `*.webm`, secretos; **conserva el GIF de demostración**), escanea los archivos en stage en busca de secretos accidentales (cancela si encuentra alguno), inicia un **historial de git limpio** si la carpeta sigue siendo el repositorio clonado del lab, y agrega todo a stage. **NO** realiza commit ni push.

Si el escaneo de secretos se detiene, elimina o agrega a `.gitignore` el archivo señalado y vuelve a ejecutar.

## Paso 2 — Iniciar sesión en el GitHub PROPIO del participante (flujo de dispositivos)

Comprueba primero y luego inicia sesión únicamente si es necesario:

```bash
gh auth status >/dev/null 2>&1 || printf 'y\n' | gh auth login --hostname github.com --git-protocol https --web
```

El `printf 'y\n'` responde con antelación a la única pregunta interactiva de `gh` — *"Authenticate Git with your GitHub credentials? (Y/n)"* — la cual de otro modo bloquearía una ejecución no interactiva (responder sí configura el asistente de credenciales de git para que el push posterior funcione). `gh auth login` imprimirá un **código de un solo uso** y la URL `https://github.com/login/device`.
Transmite ambos al participante e indícale:

> Abre **github.com/login/device** en cualquier dispositivo, inicia sesión con **tu cuenta personal de GitHub** (no la cuenta del lab de Qwiklabs) e ingresa este código: `XXXX-XXXX`.

El comando **se bloquea mientras espera la autorización**, así que proporciona un tiempo prudencial e indícale que autorice con prontitud. Confirma el éxito con `gh auth status` (muestra la cuenta autenticada — verifica que sea su cuenta personal y no una compartida o del lab).

## Paso 3 — Commit (autor = su identidad de GitHub)

```bash
bash .agents/skills_es/publish-to-github/publish.sh commit
```

Esto establece el autor del commit a partir de su inicio de sesión en GitHub (utilizando el correo electrónico de preservación de privacidad `<login>@users.noreply.github.com`) y realiza el commit del proyecto preparado en stage.

## Paso 4 — CONFIRMAR, luego crear el repositorio y hacer push

Elige un nombre de repositorio. Obtén el nombre predeterminado con:

```bash
bash .agents/skills_es/publish-to-github/publish.sh reponame
```

Esto imprimirá `buildwithgemini-<nombre-carpeta-proyecto>` (en formato slug) — la convención de nomenclatura que el personal del registro busca al canjear el swag. Propón este nombre primero. El participante puede cambiarlo, pero si lo hace, indícale que conserve el prefijo `buildwithgemini-` para que sea reconocido en el mostrador de registro.

Luego **pide confirmación explícita** antes de realizar cualquier acción remota, por ejemplo:

> Estoy a punto de crear un repositorio **público** **`<nombre>`** en tu cuenta de GitHub **`<login>`** y subir tu proyecto allí. ¿Deseas continuar?

Solo después de que responda afirmativamente:

```bash
gh repo create <name> --public --source=. --remote=origin --push
gh repo view --json url -q .url    # imprime la URL del repositorio
```

- Es obligatorio que sea **público** para la galería y para compartirlo en redes sociales.
- Si el nombre ya está ocupado en su cuenta, `gh` arrojará un error — elige otro nombre (por ejemplo, agrega un sufijo corto) y vuelve a confirmar.

## Paso 5 — Entregar el formulario de envío prellenado

```bash
bash .agents/skills_es/publish-to-github/publish.sh formlink "<repo_url>"
```

Esto imprime un enlace de Google Forms con la **URL del repositorio** (y, si están presentes en `project_brief.md`, el **título** y la **descripción del proyecto**) ya completados.
Indícale al participante que lo abra y complete los campos que solo él puede responder — su **nombre y correo electrónico de registro**, si ha **reclamado su insignia GDP**, y si **autoriza a que su proyecto aparezca destacado** — y luego lo envíe.

Explícale los beneficios:

- **Todos los que envíen el formulario** son elegibles para recibir swag (una sudadera tipo crewneck) y el canje de la insignia GDP.
- **Los proyectos destacados** son seleccionados por el equipo y publicados (con un enlace a su repositorio) en la **galería de GitHub del Track 3 de Build with Gemini**. Esta skill no modifica el repositorio de la galería — el equipo lo gestiona a partir de los envíos recibidos.

## Solución de problemas

| Síntoma | Solución |
| --- | --- |
| `gh: command not found` tras prep | `export PATH="$HOME/.local/bin:$PATH"`; si sigue faltando, instálalo manualmente: https://github.com/cli/cli#installation |
| Falla la autoinstalación de gh | No hay `curl`/acceso a internet o la imagen tiene restricciones — instala `gh` manualmente (enlace arriba) y vuelve a ejecutar desde el Paso 2 |
| El inicio de sesión se detiene en `Authenticate Git with your GitHub credentials? (Y/n)` | Usa el comando del Paso 2 (incluye `printf 'y\n'`) o simplemente responde `Y`. El código del dispositivo aparecerá justo después |
| El código del dispositivo expiró / el login quedó colgado | Vuelve a ejecutar `gh auth login --hostname github.com --git-protocol https --web`; autoriza el nuevo código con rapidez |
| Inició sesión con la cuenta de GitHub equivocada | `gh auth logout`, luego vuelve a iniciar sesión con la cuenta personal (o `gh auth switch`). Verifica con `gh auth status` |
| `git commit` falla: sin identidad configurada | Aún no ha iniciado sesión — realiza el Paso 2 primero, luego `publish.sh commit` |
| `repo create` falla: el nombre ya existe | Elige un nombre de repositorio distinto (agrega un sufijo) y vuelve a confirmar antes del push |
| `remote origin already exists` | La carpeta no se reinicializó (ya es su propio repositorio). Haz push al remoto existente (`git push -u origin main`) o apunta el remoto al nuevo repositorio |
| Escaneo de secretos cancelado | Hay una credencial/clave en stage — elimínala o agrégala a `.gitignore`, y vuelve a ejecutar. Nunca hagas commit de `application_default_credentials.json`, `.env` o claves de service account |
| Error de permisos / 403 durante el push | Consulta la skill `troubleshoot-lab-setup`; confirma que la cuenta autenticada de GitHub tenga permisos para crear repositorios |

## Referencia

- Script auxiliar: `./publish.sh` (`prep`, `commit`, `reponame`, `formlink`)
- Formulario de envío: "Build with Gemini: Project Submission and Gallery Entry"
- Flujo de dispositivos de gh: https://cli.github.com/manual/gh_auth_login
