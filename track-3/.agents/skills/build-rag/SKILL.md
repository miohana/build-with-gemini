---
name: rag-engine-setup
description: >
  Configura un corpus de Vertex AI RAG Engine (Agent Platform) en modo serverless y conéctalo a un agente de ADK. Úsalo cuando el usuario quiera "crear un corpus de RAG", "construir un almacén de RAG", "fundamentar mi agente en documentos", "agregar recuperación (retrieval) a mi agente", o encuentre errores de Spanner/allowlist al crear un corpus. Cubre la subida a Cloud Storage (GCS), el cambio a modo serverless, el parser con LLM (prompt de parsing personalizado), la importación, pruebas de recuperación independientes y cómo exponer la recuperación como una función regular (function tool) (necesario para que coexista con A2UI u otras herramientas de función en Gemini 2.5).
---

# Vertex AI RAG Engine — corpus serverless + integración con ADK

Un **corpus** de RAG Engine es un índice administrado: lo apuntas a documentos en Cloud Storage (o Drive), los divide en fragmentos (chunks), genera embeddings y almacena los vectores en una base de datos vectorial administrada. Posteriormente, un agente consulta el corpus en tiempo de ejecución a través de una **herramienta de recuperación (retrieval tool)** y fundamenta sus respuestas en los pasajes devueltos.

Esta skill construye un corpus en **modo serverless** — la opción más económica, sin listas de permisos (allowlists) y completamente administrada — y muestra cómo el agente lo consume.

## Modelo mental (cómo funciona de extremo a extremo)

```
Ingesta (una vez): documentos en GCS ──▶ LLM parser ──▶ fragmentar ──▶ generar embeddings ──▶ Vector Search administrado
                                        (prompt personalizado)        (text-embedding-005)

Consulta (runtime): turno del usuario ─▶ LLM del agente decide llamar a la herramienta de recuperación
                                      ─▶ la herramienta ejecuta retrieval_query(corpus, text) ─▶ fragmentos top-k
                                      ─▶ fragmentos inyectados en el contexto del modelo
                                      ─▶ el modelo redacta una respuesta fundamentada (opcionalmente cita fuentes)
```

Dos partes móviles: **el corpus** (construido una sola vez, abajo) y **la herramienta de recuperación** que llama el agente (última sección). El agente nunca se comunica directamente con la base de datos vectorial — llama a una herramienta y la herramienta llama al servicio de RAG.

## Modos de despliegue — elige serverless

| Modo | Backend | ¿Requiere allowlist? | Costo | Cuándo usarlo |
|------|---------|-----------|------|------|
| **Serverless** (preview) | Vector Search administrado | No | Sin costo base de servicio | **Predeterminado / tutoriales** |
| Spanner (nivel Basic/Scaled) | `RagManagedDb` (Spanner) | Sí en us-central1/-east1/-east4 | Infraestructura de Spanner | CMEK, residencia de datos, instancias dedicadas |

Serverless está disponible **únicamente en us-central1** y requiere el espacio de nombres **`vertexai.preview.rag`** — el espacio de nombres GA `vertexai.rag` solo expone `tier=` para Spanner y no permite seleccionar serverless. Serverless **no** admite CMEK / residencia de datos / Access Approval.

## Prerrequisitos (única vez)

```bash
PROJECT=tu-project-id
gcloud config set project "$PROJECT"

# Habilita las APIs. vectorsearch es obligatorio porque serverless lo usa como backend.
gcloud services enable aiplatform.googleapis.com vectorsearch.googleapis.com --project="$PROJECT"

# Coloca tus documentos fuente en un bucket de Cloud Storage (GCS) (admite HTML/PDF/TXT/Google Docs).
gcloud storage cp ./mis_docs/*.txt gs://tu-bucket/rag/
```

**Versión del SDK:** utiliza una versión **reciente** de `google-cloud-aiplatform` (≥ 1.90; comprobado en 1.163). La configuración del modo serverless y `rag.RagRetrievalConfig` utilizados abajo **no** existen en versiones anteriores — por ejemplo, 1.71 arroja `AttributeError: module 'vertexai.preview.rag' has no attribute 'RagRetrievalConfig'`. Instalar `google-adk` podría fijar una versión antigua de aiplatform, por lo que actualiza explícitamente: `pip install -U google-cloud-aiplatform`. (`vertexai.preview.rag` ahora emite un aviso de obsolescencia que apunta al nuevo cliente `agentplatform`; el espacio de nombres preview sigue funcionando perfectamente hoy).

## Construir el corpus

`scripts/create_rag_corpus.py`:

```python
from vertexai.preview import rag                      # espacio de nombres preview = soporte para serverless
from vertexai.preview.rag.utils import resources as rr
import vertexai

PROJECT_ID = "tu-project-id"
LOCATION   = "us-central1"                             # serverless solo está disponible en us-central1
GCS_PATH   = "gs://tu-bucket/rag/"                     # un archivo o un prefijo

# Instrucción personalizada para el parser con LLM — extrae lo relevante, elimina el ruido.
PARSING_PROMPT = (
    "Extract the individual useful facts and recipes described in this text. "
    "Ignore and omit all metadata, boilerplate, and image captions. "
    "Output clean, self-contained prose."
)

vertexai.init(project=PROJECT_ID, location=LOCATION)

# 1. Cambia la base de datos administrada de RAG de la región a modo serverless (a nivel de proyecto, una vez).
cfg = f"projects/{PROJECT_ID}/locations/{LOCATION}/ragEngineConfig"
rag.update_rag_engine_config(rag_engine_config=rag.RagEngineConfig(
    name=cfg,
    rag_managed_db_config=rag.RagManagedDbConfig(mode=rr.Serverless()),
))

# 2. Crea el corpus. Serverless selecciona automáticamente el backend de Vector Search administrado;
#    tú solo eliges el modelo de embeddings.
corpus = rag.create_corpus(
    display_name="my-corpus",
    embedding_model_config=rag.EmbeddingModelConfig(
        publisher_model="publishers/google/models/text-embedding-005"),
)
print("corpus:", corpus.name)   # guarda esto — el agente lo necesita

# 3. Importar + procesar (parse) + fragmentar (chunk) + generar embeddings. El parser con LLM aplica PARSING_PROMPT por archivo.
resp = rag.import_files(
    corpus_name=corpus.name,
    paths=[GCS_PATH],
    transformation_config=rag.TransformationConfig(
        chunking_config=rag.ChunkingConfig(chunk_size=512, chunk_overlap=100)),
    llm_parser=rag.LlmParserConfig(
        model_name="gemini-2.5-flash",
        custom_parsing_prompt=PARSING_PROMPT),
)
print("imported:", resp.imported_rag_files_count)
```

**Elección del parser:** parser predeterminado (gratuito, texto limpio) < **LLM parser** (extracción semántica mediante un prompt — ideal para eliminar texto repetitivo o irrelevante) < layout parser (Document AI, ideal para tablas/gráficos). La fragmentación (chunking) se ejecuta después del parseo en todos los casos.

## Probar SIN tocar el código del agente

Solo recuperación — confirma que el índice devuelva pasajes adecuados, sin invocar un LLM:

```python
from vertexai.preview import rag
import vertexai
vertexai.init(project="tu-project-id", location="us-central1")

resp = rag.retrieval_query(
    text="what is good for a cough?",
    rag_resources=[rag.RagResource(rag_corpus="projects/.../ragCorpora/NNN")],
    rag_retrieval_config=rag.RagRetrievalConfig(top_k=5),
)
for c in resp.contexts.contexts:
    print(c.score, c.text[:200])
```

También puedes probarlo en **Cloud Console**: Vertex AI → RAG Engine → tu corpus → panel Retrieve. Después de la importación, permite un breve tiempo de indexación (la recuperación puede devolver 404 brevemente incluso cuando el archivo figure como `ACTIVE`).

## Conectarlo al agente (ADK) — exponer la recuperación como una herramienta de función regular

El agente accede al corpus mediante una **herramienta de recuperación (retrieval tool)**. Envuelve `rag.retrieval_query` en una función estándar de Python y registra *esa* función como la herramienta del agente. **Haz esto en lugar de usar `VertexAiRagRetrieval` de ADK** — consulta la advertencia de compatibilidad a continuación para entender por qué esto es crucial.

```python
from google.adk.agents import Agent

CORPUS_NAME = "projects/.../ragCorpora/NNN"   # obtenido de create_rag_corpus.py

def consult_docs(query: str) -> str:
    """Search the herbal corpus and return matched passages.

    Args:
        query: What to look up (a plant, ailment, or recipe).
    Returns:
        The matched passages, or a note that none was found.
    """
    from vertexai.preview import rag
    try:
        resp = rag.retrieval_query(
            text=query,
            rag_resources=[rag.RagResource(rag_corpus=CORPUS_NAME)],
            rag_retrieval_config=rag.RagRetrievalConfig(top_k=5),
        )
    except Exception as e:
        return f"Retrieval failed: {e}"
    contexts = getattr(resp.contexts, "contexts", [])
    passages = [c.text.strip() for c in contexts if getattr(c, "text", "").strip()]
    return "\n\n---\n\n".join(passages) or "No relevant passage found."

agent = Agent(
    model="gemini-2.5-flash",
    name="apothecary",
    instruction="Answer using the herbal corpus. Call consult_docs before answering.",
    tools=[consult_docs],   # junto con cualquier otra herramienta, ej. el conjunto de herramientas A2UI
)
```

En tiempo de ejecución, el modelo decide cuándo llamar a `consult_docs`; la función ejecuta `retrieval_query` contra el corpus y devuelve los mejores fragmentos (top-k) como resultado, los cuales el modelo lee para redactar una respuesta fundamentada. La documentación de la función (docstring) es fundamental — representa la declaración de la función para la herramienta, de modo que el modelo sabe cuándo utilizarla.

### ⚠️ Compatibilidad: NO uses `VertexAiRagRetrieval` junto con otras herramientas de función

`VertexAiRagRetrieval` de ADK (de `google.adk.tools.retrieval`) **no** se registra como una herramienta de función normal — se registra como la **herramienta nativa de fundamentación por recuperación (built-in retrieval grounding tool)** de Gemini. Los modelos Gemini 2.5 (`gemini-2.5-flash` y `gemini-2.5-pro`) **rechazan cualquier solicitud que combine la herramienta de recuperación nativa junto con declaraciones de funciones normales en el turno que transporta un `functionResponse`**. Por lo tanto, en el instante en que tu agente tiene *cualquier* otra herramienta de función — por ejemplo, el conjunto de herramientas A2UI (`SendA2uiToClientToolset` / `send_a2ui_to_client`) — la combinación resulta inválida.

- **Síntoma:** la primera solicitud al modelo tiene éxito; la falla se manifiesta en el turno de seguimiento justo después de que la primera llamada a la herramienta retorna — un error directo `400 Bad Request ... INVALID_ARGUMENT` sin mayor detalle de campos. Parece intermitente o misterioso y consume mucho tiempo de depuración.
- **La solución es la herramienta de función regular mostrada arriba** — la recuperación se convierte simplemente en otra declaración de función, no se incluye ninguna herramienta de fundamentación nativa en la solicitud y la combinación es totalmente válida en cualquier modelo o versión.
- **No** intentes "solucionarlo" cambiando a un alias variable como `gemini-flash-latest`; hoy en día puede tolerar la combinación de manera casual, pero es poco confiable. Desarrolla siempre contra los modelos 2.5 fijados.
- Esta es una regla general, no una peculiaridad de RAG: **ninguna herramienta integrada (built-in)** (retrieval, fundamentación con Google Search, ejecución de código) puede compartir una solicitud con declaraciones de funciones regulares. Expón la capacidad como una función regular siempre que también requieras llamadas a funciones (function calling).

**Consideración de región (común en producción):** un corpus serverless reside únicamente en `us-central1`, pero el modelo de tu agente a menudo se ejecuta en otra región (por ejemplo, `GOOGLE_CLOUD_LOCATION=global`). `rag.retrieval_query` se ejecuta contra la región en la que se inicializó el **SDK de aiplatform** — si no coincide con la región del corpus, recibirás un error `MethodNotImplemented / 404`. El cliente del modelo genai (basado en variables de entorno) y los servicios de sesión/memoria de ADK (con `location=` explícito) NO usan el inicializador del SDK de aiplatform, por lo que puedes inicializar con seguridad únicamente el cliente de RAG en la región del corpus:

```python
import vertexai
# región extraída de projects/<p>/locations/<region>/ragCorpora/<id>
vertexai.init(project="...", location="us-central1")  # antes de la primera llamada a retrieval_query
```

Para guiar al modelo sobre *cómo* responder, inclúyelo en la instrucción del agente, por ejemplo:
"When you rely on the Herbal's words, quote the passage verbatim in quotation marks and name it as Culpeper's Complete Herbal; otherwise paraphrase."

## Solución de problemas

- `INVALID_ARGUMENT ... Spanner mode ... restricted to allowlisted projects` → estás usando el espacio de nombres GA o el modo Spanner. Utiliza `vertexai.preview.rag` + `mode=rr.Serverless()`.
- `PERMISSION_DENIED ... Vector Search API has not been used` → habilita `vectorsearch.googleapis.com`, espera ~1 min y vuelve a intentar.
- `NOT_FOUND No vertex rag corpus found` justo después de importar → retraso de indexación; reintenta tras unos segundos.
- Error directo `400 ... INVALID_ARGUMENT` (sin detalle de campos) en el turno inmediatamente posterior al retorno de la primera llamada a una herramienta → estás combinando `VertexAiRagRetrieval` (herramienta de fundamentación integrada) con herramientas de funciones. Expón la recuperación como una herramienta de función regular (ver la advertencia de compatibilidad arriba).
