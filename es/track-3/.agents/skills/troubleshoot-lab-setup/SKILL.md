---
name: troubleshoot-lab-setup
description: >-
  Verifica que el entorno del lab de Gemini World esté configurado correctamente y diagnostica o soluciona errores comunes. Úsalo al INICIO del lab justo después de iniciar sesión para confirmar que todo esté listo ("verifica mi configuración", "¿estoy listo para empezar?", "comprueba mi entorno", "¿inicié sesión correctamente?"), Y SIEMPRE que el usuario encuentre un error y no sepa por qué — permiso denegado / 403 / PERMISSION_DENIED, "API not enabled", un deploy que falla, el frontend no puede comunicarse con el agente, errores en /chat, fallas en code sandbox, fallas en generación de imágenes, o un mensaje vago como "no está funcionando". Revisa las causas habituales — haber iniciado sesión en Antigravity con la cuenta correcta (flujo de GCP/lab, no con la cuenta personal de Google), que el proyecto de GCP esté configurado, que se hayan ejecutado tanto gcloud auth login como application-default login, que la API esté habilitada y que la identidad cuente con roles/aiplatform.user. No lo uses para programar funciones del agente ni para consultas de desarrollo que no involucren errores.
---

# Solución de problemas de configuración del lab

Casi todas las fallas en este lab se deben a un puñado de problemas en el entorno — no al código del usuario. Esta skill cuenta con **dos modos**:

- **Preflight (Verificación previa)** — se ejecuta al *inicio* del lab, justo después de que el usuario inicia sesión, para confirmar que todo esté listo antes de comenzar a construir. Detecta problemas de manera anticipada en lugar de a mitad del módulo.
- **Error triage (Diagnóstico de errores)** — se ejecuta cuando el usuario se topa con un error. Ejecuta primero las comprobaciones indicadas a continuación, soluciona lo que esté mal y luego dirígete al síntoma correspondiente.

Es preferible *verificar* el estado con un comando de lectura antes de *modificar* cualquier cosa, e informarle al usuario lo que encontraste.

## Preflight: comprobación de estado al inicio del lab

Cuando el usuario pida verificar su configuración (o cuando estés iniciando el lab), ejecuta las **comprobaciones 0 a 5 a continuación en orden** y muestra un resumen claro de aprobado/fallido, por ejemplo:

```
Setup check:
✅ Antigravity signed in as <lab account>
✅ Project set to <qwiklabs project id>
✅ gcloud auth + ADC present
✅ Vertex AI / aiplatform API enabled
✅ roles/aiplatform.user granted
✅ agents-cli skills loaded in AGY
You're ready to start. ✅
```

Soluciona cualquier elemento que falle y vuelve a ejecutar esa comprobación antes de continuar. Omite los síntomas de deploy/frontend/sandbox durante la verificación previa — esos surgen más adelante.

## Las comprobaciones (ejecútalas ante casi CUALQUIER error y durante el preflight)

Estas son las causas más habituales. Revísalas todas antes de profundizar.

**0. ¿El usuario inició sesión en Antigravity con la cuenta CORRECTA?** Una trampa muy común: iniciar sesión en Antigravity con una cuenta personal de **Google** (por ejemplo, una cuenta `@gmail.com` o `@google.com`) en lugar de las credenciales del **lab de GCP / Qwiklabs**. Todo *parece* estar bien, pero las llamadas al proyecto fallan con errores de permisos porque AGY está actuando bajo la identidad equivocada.

- Pregúntale al usuario qué cuenta utilizó para iniciar sesión en Antigravity y confirma que sea la **cuenta del lab de Qwiklabs**, no su cuenta personal de Google.
- Comprueba que la cuenta activa de gcloud coincida con la cuenta del lab:
  ```bash
  gcloud auth list        # la cuenta ACTIVA (*) debe ser la cuenta del lab
  ```
- Si AGY inició sesión con la cuenta incorrecta, indícale al usuario que cierre sesión en Antigravity y vuelva a iniciar sesión mediante el flujo de inicio de sesión de GCP/lab (según las instrucciones de configuración del lab), y luego vuelve a ejecutar la verificación previa.

**1. ¿Está configurado el proyecto correcto?** Cloud Shell / AGY pueden adoptar silenciosamente un proyecto incorrecto por defecto (por ejemplo, `cloudshell-gca`), lo que hace que los comandos de habilitación o deploy fallen.

```bash
gcloud config get-value project          # ¿cuál está activo ahora mismo?
echo "$GOOGLE_CLOUD_PROJECT"             # lo que el entorno cree que es
```

Un ID de proyecto de Qwiklabs correctamente configurado **contiene la cadena `qwiklabs`** (por ejemplo, `qwiklabs-gcp-01-abc123def456`). Si el proyecto activo no contiene `qwiklabs`, es casi seguro que sea el proyecto equivocado — trátalo como una señal de alerta.

Si alguno es incorrecto, está vacío o no contiene `qwiklabs`, **pídele al usuario su ID de proyecto de Qwiklabs** (desde el panel del lab) y fíjalo ejecutando:

```bash
export GOOGLE_CLOUD_PROJECT=<su project id de qwiklabs>
gcloud config set project "$GOOGLE_CLOUD_PROJECT"
```

Confirma también que el proyecto esté fijado dentro de la configuración de AGY para que no cambie a mitad de la ejecución. Durante el preflight, fija siempre el proyecto de esta manera — no asumas que ya está configurado.

**2. ¿El usuario se ha autenticado?** Se requieren dos inicios de sesión independientes — la falta de inicio de sesión en Application Default Credentials (ADC) es la causa principal de errores 403 desde el código.

```bash
gcloud auth list                          # ¿hay una cuenta activa?
```

Si no está autenticado, ejecuta AMBOS:

```bash
gcloud auth login
gcloud auth application-default login
```

**3. ¿Está habilitada la API necesaria?** Las operaciones de deploy, sandbox y llamadas a modelos requieren que la API de Agent Platform / Vertex AI esté activada. Los errores "API not enabled" suelen deberse a esto o a un proyecto incorrecto (ver #1).

```bash
gcloud services list --enabled | grep -i aiplatform
gcloud services enable aiplatform.googleapis.com   # habilitar si falta
```

**4. ¿La identidad tiene el rol de IAM correcto?** Muchas funciones requieren `roles/aiplatform.user`. Si una llamada devuelve `PERMISSION_DENIED`, verifica el rol en la identidad que esté realizando la llamada (tu usuario, la service account del agente o la service account de Cloud Run).

```bash
gcloud projects add-iam-policy-binding "$GOOGLE_CLOUD_PROJECT" \
  --member="user:<tu@ejemplo.com>" \
  --role="roles/aiplatform.user"
```

Ajusta el rol a lo que realmente *hace* la identidad que llama. Un **agente desplegado** se ejecuta con su propia service account **sin acceso a Firestore ni a Cloud Storage por defecto**, por lo que cualquier agente que lea Firestore debe tener otorgado `roles/datastore.user` a esa service account — **otórgalo al momento del deploy** (ver "Después de hacer deploy" más abajo), no solo `aiplatform.user` a tu usuario.

**5. ¿Están cargadas las skills de agents-cli en AGY?** Los pasos de scaffolding, deploy, evaluación y Memory Bank del lab dependen de que las skills `google-agents-cli-*` estén registradas en Antigravity. Estas son **independientes** de las skills del workshop incluidas en el repositorio bajo `.agents/skills/` — se instalan mediante el paso `agents-cli setup`, no de manera automática.

- En AGY, ejecuta `/skills` y confirma que veas las skills de ciclo de vida de `google-agents-cli-*` (por ejemplo, scaffold, deploy, adk-code) **además de** las skills del workshop. Si solo aparecen las skills del workshop, las skills de agents-cli no están cargadas.
- Si faltan, instálalas desde Cloud Shell:
  ```bash
  agents-cli setup --skip-auth
  ```
- Luego **reinicia Antigravity** para que reconozca las nuevas skills instaladas (ver "Antigravity / MCP se comporta de forma extraña justo después de instalar" más abajo), y vuelve a ejecutar `/skills` para confirmar que ahora aparezcan.

> Consejo: si tienes instalado el **Developer Knowledge MCP**, úsalo para confirmar el nombre exacto de la API, el rol y el comando de un producto en lugar de adivinar.

## Síntoma → Solución

### 403 / PERMISSION_DENIED / "permission denied"
Casi siempre se debe a uno de estos motivos: haber iniciado sesión en Antigravity con la cuenta incorrecta (personal) en lugar de la cuenta del lab (comprobación #0), falta de ADC (comprobación #2), falta del rol `roles/aiplatform.user` en la identidad que realiza la llamada (comprobación #4), o el proyecto equivocado (comprobación #1). Descártalos en ese orden.

### "API not enabled" / "SERVICE_DISABLED"
O estás en el proyecto equivocado (#1) o la API está desactivada (#3). Corrige el proyecto primero y luego habilita la API.

### `adk web`/`adk run` falla al inicio / `cannot import name '<x>' from 'google.cloud'` / ModuleNotFound
Ejecutaste un comando **`adk` directo**, el cual resuelve a un Python *global* que no tiene instaladas las dependencias de tu proyecto. Ejecútalo a través del `.venv` del proyecto con **`uv run`**:

```bash
uv run adk web --port 8080 --allow_origins "*" --reload_agents
```

Síntomas reveladores: `cannot import name 'firestore' from 'google.cloud'`, o un `ModuleNotFoundError` para un paquete que *sabes* que está instalado (por ejemplo, `a2ui-agent-sdk`). El comando directo `adk` usa un intérprete distinto; `uv run adk` utiliza el venv del proyecto donde residen las dependencias reales.

### `agents-cli deploy` falla
1. ¿El proyecto está fijado? (#1) — esta es la causa más común.
2. ¿Las APIs están habilitadas y autenticadas? (#2, #3)
3. Vuelve a leer el error exacto; si menciona un permiso, dirígete a #4.
Lista los despliegues existentes para ver el estado actual: `agents-cli deploy --list`.

### Después de hacer deploy: otorga a la service account del agente los roles que necesitan sus herramientas
Haz esto como parte de **cada** deploy donde el agente lea Firestore o Cloud Storage — de forma proactiva, sin esperar a que falle. El agente desplegado se ejecuta con su propia service account (por defecto la service agent de Reasoning Engine, `service-PROJECT_NUMBER@gcp-sa-aiplatform-re.iam.gserviceaccount.com`), la cual **no tiene roles de acceso a datos por defecto**. Para un agente respaldado por Firestore, otórgale `roles/datastore.user` (este comando autocompleta el número de tu proyecto):

```bash
gcloud projects add-iam-policy-binding "$GOOGLE_CLOUD_PROJECT" \
  --member="serviceAccount:service-$(gcloud projects describe "$GOOGLE_CLOUD_PROJECT" --format='value(projectNumber)')@gcp-sa-aiplatform-re.iam.gserviceaccount.com" \
  --role="roles/datastore.user"
```

Para un agente que escribe archivos en Cloud Storage (por ejemplo, uno que sube imágenes generadas), otorga a su service account permisos de escritura en el bucket. Limita el alcance del permiso al bucket, no a todo el proyecto:

```bash
gcloud storage buckets add-iam-policy-binding "gs://<tu-bucket-de-imagenes>" \
  --member="serviceAccount:service-$(gcloud projects describe "$GOOGLE_CLOUD_PROJECT" --format='value(projectNumber)')@gcp-sa-aiplatform-re.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"
```

Una herramienta de Cloud Storage de solo lectura únicamente necesita `roles/storage.objectViewer`.
Asegúrate también de haber cargado los datos iniciales (seed) en el mismo proyecto donde desplegaste; de lo contrario, el agente desplegado leerá una base de datos vacía.

### El frontend no puede comunicarse con el agente / sin respuesta / errores al enviar
Ejecuta todos los comandos del frontend desde la carpeta `frontend/`.
1. ¿Están instaladas las dependencias? `pip install -r requirements.txt`
2. ¿Está configurada y correcta `AGENT_ENGINE_RESOURCE_NAME`? Debe ser el nombre de recurso de `deployment_metadata.json` (escrito en el paso de deploy). Configura también `AGENT_DIRECTORY` con tu `agent_directory` de `agents-cli-manifest.yaml` (generalmente `app`), ya que la ruta del endpoint de A2A lo incluye:
   ```bash
   echo "$AGENT_ENGINE_RESOURCE_NAME"
   export AGENT_ENGINE_RESOURCE_NAME="<pegar de deployment_metadata.json>"
   export AGENT_DIRECTORY="app"
   ```
3. ¿ADC está presente localmente? (#2)
4. Prueba de coherencia: envía un mensaje y luego pregunta "¿Qué acabo de preguntar?" — si no puede recordarlo, la conexión con el agente desplegado es la que está fallando, no la interfaz de usuario.
5. Si las respuestas en texto sin formato funcionan pero **las respuestas basadas en herramientas no devuelven nada**, no es un problema de la UI — consulta "El agente desplegado no responde" más abajo.

### Errores del frontend: `operation_schemas()` vacío, `no attribute 'stream_query'`, o errores 500 del proxy anterior
Este es el cambio de **agents-cli 1.1.0 (GA)**. GA despliega agentes de ADK en Agent Runtime como agentes A2A y ya no registra el esquema de operaciones de reasoning-engine, por lo que un frontend construido sobre `agent_engines.get(...).stream_query()` falla: `operation_schemas()` está vacío y el controlador no cuenta con `stream_query`/`create_session`. Confirma primero que el agente funcione correctamente sobre A2A:
```bash
agents-cli run --url https://<LOCATION>-aiplatform.googleapis.com/v1/<RESOURCE> --mode a2a "hi"
```
Si responde, el agente está bien y el proxy está usando la ruta obsoleta del SDK. Utiliza la plantilla actual de `build-agent-frontend`, la cual se comunica vía A2A (obtiene la tarjeta del agente y envía mensajes con el cliente de a2a-sdk). No intentes forzar un deploy de ADK/`stream_query`. Todas las plantillas de ADK 1.1.0 están etiquetadas para A2A, y el contenedor sirve A2A sin importar si `is_a2a` es verdadero o falso.

### El agente desplegado no responde / una herramienta funciona en el Playground pero no en el deploy
Una herramienta respaldada por Firestore funciona en el Playground (con tu ADC local) pero el turno del agente desplegado termina sin respuesta. Es casi seguro que se deba a uno de estos puntos:

1. **Omitiste otorgar el rol.** Realiza el paso "Después de hacer deploy" arriba — la service account desplegada necesita `roles/datastore.user`.
2. **Cargaste los datos (seed) y desplegaste en proyectos distintos.** El agente desplegado lee Firestore desde *su propio* proyecto; si cargaste los datos en otro lugar, verá una base de datos vacía. Compara y vuelve a cargar los datos en el proyecto de despliegue si difieren:
   ```bash
   gcloud config get-value project          # dónde cargaste los datos
   echo "$AGENT_ENGINE_RESOURCE_NAME"        # el proyecto en el que se ejecuta el agente
   ```

Confirma que la base de datos de Firestore exista y contenga datos en el proyecto de despliegue (consola de Firebase), y vuelve a probar. Si se queda colgado en silencio = proyecto incorrecto/vacío; `PERMISSION_DENIED` = falta de rol.

### `NotFound: The database (default) does not exist` (agente de Firestore desplegado)
Tu código está construyendo el cliente de Firestore a partir del **número** de proyecto, no del ID. En Agent Engine, **tanto `google.auth.default()` como la variable de entorno `GOOGLE_CLOUD_PROJECT` devuelven el *número* de proyecto** — pero Firestore solo resuelve la base de datos `(default)` mediante el **ID** del proyecto. Por ende, este patrón común genera un error 404 en el agente desplegado aunque funcione de forma local:

```python
# INCORRECTO en Agent Engine: GOOGLE_CLOUD_PROJECT es el NÚMERO de proyecto allí
db = firestore.Client(project=os.getenv("GOOGLE_CLOUD_PROJECT"))
```

Fija explícitamente el **ID** del proyecto — nunca derives el proyecto de Firestore de `google.auth.default()` o `GOOGLE_CLOUD_PROJECT`:

```python
FIRESTORE_PROJECT = "<tu-project-id>"           # el ID, NO el número
db = firestore.Client(project=FIRESTORE_PROJECT)
```

Usa el mismo ID en tu script de carga de datos (seed) y luego **vuelve a desplegar**. (Funciona localmente solo porque tu `GOOGLE_CLOUD_PROJECT`/ADC local ya es el ID; el entorno de ejecución desplegado lo establece al número).

### Errores en `/chat` después de desplegar el frontend en Cloud Run
El servicio de Cloud Run se ejecuta como una **identidad diferente** a la de tu usuario local, por lo que su service account necesita `roles/aiplatform.user`. Otórgalo a la service account de Cloud Run (no a tu cuenta de usuario) y vuelve a intentar.

### Falla de Code sandbox / ejecución de código
1. La API de Agent Platform debe estar habilitada (#3) y la identidad debe contar con `roles/aiplatform.user` (#4).
2. ¿Realmente existe un sandbox? Si no es así, crea uno desde el recurso de agent engine antes de ejecutar código.

### Falla la generación de imágenes o la imagen no aparece
Si la herramienta arroja un error durante la generación, generalmente se trata de un problema de API/permisos (#3, #4) o una incompatibilidad de región/acceso al modelo. Confirma que el modelo y la región estén disponibles para el proyecto y que la API esté habilitada.

Si la imagen se muestra en el Playground pero no en el agente desplegado, o si una tarjeta o el frontend muestran una imagen rota, ten en cuenta que el agente sube las imágenes generadas a un bucket público y muestra la URL pública. Deben cumplirse dos condiciones:

1. La service account desplegada debe poder escribir en el bucket (`roles/storage.objectAdmin`, ver "Después de hacer deploy" arriba). Funciona localmente porque te ejecutas como tú mismo, pero el agente desplegado se ejecuta bajo su propia cuenta.
2. El bucket debe servir objetos de forma pública. Otorga a `allUsers` el rol `roles/storage.objectViewer` en el bucket. Si ese otorgamiento falla con `public access prevention is enforced`, la organización del proyecto bloquea los buckets públicos y el enfoque de URL pública no funcionará allí.

### Antigravity / MCP se comporta de forma extraña justo después de instalar
Si acabas de instalar agents-cli o el Developer Knowledge MCP, **reinicia Antigravity** para que cargue las nuevas skills/MCP y vuelve a intentar.

## Reglas generales
- Confirma primero la **cuenta de Antigravity** — iniciar sesión con una cuenta personal de Google en lugar del flujo de lab/GCP rompe silenciosamente el acceso al proyecto.
- Corrige el **proyecto** a continuación — un proyecto incorrecto provoca la aparición de la mitad de los demás errores.
- Existen **dos** inicios de sesión de autenticación (`login` y `application-default login`); omitir el segundo rompe las llamadas a nivel de código mientras que la CLI sigue pareciendo funcional.
- Asigna el rol de IAM a la **identidad que realmente realiza la llamada** — tu usuario a nivel local, pero la service account en Cloud Run.
- Ante la duda sobre un comando, rol o nombre de API exacto, consulta el Developer Knowledge MCP en lugar de adivinar.
