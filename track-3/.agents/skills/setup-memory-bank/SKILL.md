---
name: memory-bank-setup
description: >
  Agrega memoria a largo plazo entre sesiones a un agente de ADK usando Vertex AI Memory Bank de Agent Platform, y conéctala al agente. Úsalo cuando el usuario quiera "agregar memoria", "agregar un Memory Bank", "recordar datos/preferencias entre sesiones", "hacer que mi agente me recuerde entre conversaciones", o pregunte por qué los recuerdos no persisten o no se muestran en Cloud Console. Cubre la conexión del lado del agente (PreloadMemoryTool + un callback de generación de memoria), la creación de la instancia administrada de Memory Bank, cómo apuntar el servicio de memoria del runtime hacia ella (ADK Web local y Agent Runtime desplegado), la verificación en la Console, y la consideración importante de que `agents-cli deploy` NO configura un servicio de memoria por sí mismo.
---

# Vertex AI Memory Bank — memoria entre sesiones + integración con ADK

Las **sesiones (sessions)** recuerdan una conversación. **Memory Bank** recuerda hechos y preferencias *a través de múltiples* conversaciones (por ejemplo, "el usuario es intolerante al gluten", "llámame Dr. Vance", "responde siempre en el sistema métrico"). En cada turno, Memory Bank lee la conversación, extrae fragmentos duraderos y los almacena asociados a un `user_id` para que las sesiones futuras puedan recordarlos.

Esta skill agrega Memory Bank a un agente de ADK (incluyendo proyectos estructurados con `agents-cli`) y lo conecta de extremo a extremo.

## Modelo mental (cómo funciona de extremo a extremo)

```
Escritura (por turno): eventos de sesión ─▶ after_agent_callback ─▶ add_session_to_memory()
                                          ─▶ Memory Bank extrae y almacena hechos duraderos

Lectura  (por turno): PreloadMemoryTool ─▶ search_memory(user_id) al inicio del turno
                                         ─▶ recuerdos relevantes inyectados en la instrucción del sistema
```

Dos partes móviles:
1. **El código del agente** — una *herramienta* de memoria (lectura) + un *callback* (escritura). Es idéntico para cualquier entorno de ejecución.
2. **Un servicio de memoria** que apunta a una **instancia de Memory Bank**. Esta es la parte que varía entre el entorno local y el desplegado, y la parte que `agents-cli` **no** configura de forma automática.

## El detalle principal que suele confundir

**Una "instancia de Memory Bank" es simplemente una instancia de Agent Engine (Reasoning Engine).** Se crea con `client.agent_engines.create()`; su nombre de recurso es `projects/<p>/locations/<loc>/reasoningEngines/<ID>` y `<ID>` es el ID de Memory Bank que pasas en todas partes como `agentengine://<ID>`.

Consecuencias:
- Memory Bank es un **recurso administrado en la nube** — no puedes ver recuerdos reales y persistentes en una ejecución local puramente en memoria. ADK utiliza `InMemoryMemoryService` por defecto a menos que lo apuntes explícitamente a una instancia de Memory Bank.
- **`agents-cli deploy` no conecta un servicio de memoria.** Configura un servicio de *sesiones* en Agent Runtime, pero deja el servicio de memoria con el valor por defecto de ADK. Agregar la herramienta + callback es necesario pero **no suficiente** — también debes apuntar un servicio de memoria a una instancia de Memory Bank (pasos a continuación).
- Dado que es un recurso en la nube, realiza el trabajo de memoria **después** (o en paralelo) de un primer deploy, o crea una instancia independiente para pruebas locales — no antes de que exista algún Agent Engine.

## Prerrequisitos (única vez)

```bash
PROJECT=tu-project-id
LOCATION=us-central1          # usa una región compatible con Memory Bank
gcloud config set project "$PROJECT"
gcloud services enable aiplatform.googleapis.com --project="$PROJECT"
gcloud auth application-default login   # ya realizado si iniciaste sesión mediante `agy`
```

Memory Bank funciona en regiones específicas — consulta
https://docs.cloud.google.com/gemini-enterprise-agent-platform/resources/agent-locations.
Mantén `GOOGLE_CLOUD_LOCATION` alineado con la región en la que crees la instancia.

## Paso 1 — Conectar el agente (herramienta + callback)

Edita la definición del agente (en un proyecto de `agents-cli`, este es `app/agent.py`).
Agrega un **callback de generación de memoria** y una **herramienta de memoria**:

```python
from google.adk.agents import Agent
from google.adk.agents.callback_context import CallbackContext
from google.adk.tools.preload_memory_tool import PreloadMemoryTool
# Herramienta alternativa: from google.adk.tools.load_memory_tool import LoadMemoryTool


# ESCRITURA: después de cada turno, envía la sesión a Memory Bank para su extracción.
async def generate_memories_callback(callback_context: CallbackContext):
    await callback_context.add_session_to_memory()
    return None


root_agent = Agent(
    model="gemini-2.5-flash",
    name="root_agent",
    instruction=(
        "You are a helpful assistant. You remember the user's stated "
        "preferences and facts from previous conversations and use them to "
        "personalize your responses."
    ),
    # LECTURA: PreloadMemoryTool recupera recuerdos al inicio de cada turno y
    # los inyecta en la instrucción del sistema (no requiere llamada explícita a la herramienta).
    # Usa LoadMemoryTool() en su lugar si prefieres que el modelo los consulte bajo demanda.
    tools=[PreloadMemoryTool()],
    after_agent_callback=generate_memories_callback,
)
```

Ese es todo el cambio en el código del agente. Es independiente del runtime — el mismo código funciona localmente y desplegado. **`add_session_to_memory()` y las herramientas no tienen efecto frente a un servicio en memoria**, por lo que solo generan recuerdos persistentes una vez que se conecta una instancia real de Memory Bank (siguientes pasos).

> Envía a memoria únicamente los turnos relevantes. `add_session_to_memory()` al final de un turno es la forma más simple; para un control más preciso utiliza `callback_context.add_events_to_memory(events=...)` con un subconjunto de eventos.

## Paso 2 — Crear una instancia de Memory Bank

Si **ya realizaste el deploy** con `agents-cli`, ya cuentas con un Agent Engine — puedes reutilizar su ID como el ID de Memory Bank (pasa al Paso 3). De lo contrario, crea una instancia independiente (`scripts/create_memory_bank.py`):

```python
import vertexai

PROJECT_ID = "tu-project-id"
LOCATION   = "us-central1"

client = vertexai.Client(project=PROJECT_ID, location=LOCATION)

# Una instancia de Memory Bank ES una instancia de Agent Engine. La configuración
# predeterminada es adecuada para el lab; extrae datos y preferencias generales del usuario automáticamente.
memory_bank = client.agent_engines.create()

resource_name = memory_bank.api_resource.name       # projects/.../reasoningEngines/NNN
memory_bank_id = resource_name.split("/")[-1]        # NNN  ← usa esto en todas partes
print("MEMORY_BANK_ID:", memory_bank_id)
print("resource name :", resource_name)
```

Guarda el `MEMORY_BANK_ID` impreso. (Para personalizar *qué* temas se extraen — `USER_PERSONAL_INFO`, `USER_PREFERENCES`, `EXPLICIT_INSTRUCTIONS`, `KEY_CONVERSATION_DETAILS` — consulta la sección "Configure your Memory Bank instance" en la documentación; la configuración predeterminada no requiere ajustes).

## Paso 3 — Apuntar un servicio de memoria a la instancia

Se debe proporcionar al servicio de memoria el URI `agentengine://<MEMORY_BANK_ID>`. Elige la fila que coincida con la forma en que se ejecuta el agente:

| Runtime | Cómo conectar el servicio de memoria |
|---|---|
| **Local ADK Web** (el más rápido para probar) | `adk web --memory_service_uri=agentengine://MEMORY_BANK_ID` |
| **Desplegado en Agent Runtime** | Configura el servicio de memoria en la app (abajo) para que el contenedor desplegado use Memory Bank y no el valor en memoria por defecto |
| **Local `Runner` / script** | `VertexAiMemoryBankService(project=..., location=..., agent_engine_id=MEMORY_BANK_ID)` pasado a `Runner(memory_service=...)` |

### Prueba local (recomendada primero)

```bash
export GOOGLE_CLOUD_PROJECT="tu-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"
# Ejecuta desde la carpeta que contiene el paquete del agente (por ejemplo, la raíz del proyecto
# con app/ adentro). Esto anula el valor predeterminado en memoria de ADK.
adk web --memory_service_uri=agentengine://MEMORY_BANK_ID
```

`agents-cli playground` ejecuta `adk web` pero no reenvía `--memory_service_uri`, por lo que para probar un Memory Bank *real* de forma local, ejecuta `adk web` directamente con el flag.

### Desplegado en Agent Runtime (proyecto de agents-cli)

`agents-cli deploy` no adjuntará un servicio de memoria, por lo que debes definir uno explícitamente en la aplicación para que el contenedor desplegado utilice Memory Bank. En la definición de la aplicación ADK (la configuración de `AdkApp` / `get_fast_api_app` — consulta las skills `google-agents-cli-deploy` y `google-agents-cli-adk-code` para ubicar el archivo exacto según la versión del proyecto), proporciona un constructor de servicio de memoria:

```python
from google.adk.memory import VertexAiMemoryBankService

def memory_bank_service_builder():
    return VertexAiMemoryBankService(
        project="tu-project-id",
        location="us-central1",
        agent_engine_id="MEMORY_BANK_ID",   # reutiliza el ID del engine desplegado, o uno independiente
    )
# Pasa memory_service_builder=memory_bank_service_builder a AdkApp,
# o --memory_service_uri=agentengine://MEMORY_BANK_ID al comando de deploy de ADK.
```

Luego vuelve a desplegar. Confirma que el agente desplegado realmente persista los recuerdos (Paso 4) — no asumas que la configuración por defecto lo hizo.

## Paso 4 — Verificar

1. **Habla con el agente** (en ADK Web local o en el playground desplegado): menciona un hecho duradero, por ejemplo, *"Recuerda que soy alérgico a la penicilina."*
2. **Inicia una NUEVA sesión** y pregunta algo que requiera ese dato — el agente debería recordarlo sin necesidad de que se lo repitas.
3. **Observa el recuerdo almacenado en Cloud Console:**
   https://console.cloud.google.com/agent-platform/memory-bank
   (Vertex AI → Agent Engines → tu instancia → Memory Bank.) Espera unos segundos — la extracción se ejecuta en segundo plano después del turno.

## Solución de problemas

- **Los recuerdos nunca persisten / no hay nada en la Console** → estás utilizando el valor predeterminado `InMemoryMemoryService`. La herramienta + callback por sí solos no crean un banco; debes pasar `--memory_service_uri=agentengine://<ID>` (local) o configurar `VertexAiMemoryBankService` en la app desplegada. Esta es la causa #1.
- **Funciona localmente pero no al estar desplegado** → `agents-cli deploy` no conectó un servicio de memoria; agrega `memory_service_builder` / `memory_service_uri` y vuelve a desplegar (Paso 3, fila de desplegado).
- **Recuerda en la misma sesión pero no entre sesiones distintas** → estás viendo el *estado de la sesión*, no la memoria a largo plazo. Confirma que `PreloadMemoryTool` esté en `tools` y que `after_agent_callback` esté configurado, además de que ambas sesiones utilicen el **mismo `user_id`** (los recuerdos tienen un alcance definido por `user_id` + `app_name`).
- **`NOT_FOUND` / errores de permisos en el servicio de memoria** → la región de `MEMORY_BANK_ID` debe coincidir con `GOOGLE_CLOUD_LOCATION` y debe ser una región admitida por Memory Bank; la cuenta que realiza la llamada o service account necesita `roles/aiplatform.user`.
- **`'await' outside function` en un script independiente** → envuelve las llamadas asíncronas en `asyncio.run(...)`; ADK está diseñado con arquitectura asíncrona desde su base.
