---
name: enable-a2ui
description: Haz que un agente de ADK emita A2UI para que sus respuestas se rendericen como una interfaz visual enriquecida (tarjetas, tablas construidas a partir de filas/columnas e imágenes desde una URL pública) en la interfaz de desarrollo de ADK (adk web) en lugar de texto sin formato. Úsalo cuando el usuario quiera agregar A2UI a su agente, renderizar tarjetas o tablas en adk web, mostrar una imagen dentro de una tarjeta, o cuando A2UI aparezca como JSON sin procesar o una tarjeta en blanco. Incluye un a2ui_utils.py listo para usar (el after_model_callback que necesita el renderizador de adk web) en ./template. Solo para visualización en adk web (los botones, acciones y modales no funcionan); enviar A2UI a un frontend personalizado de producción es un proceso independiente.
---

# Habilitar A2UI (renderizado en adk web)

Haz que un agente de ADK devuelva **A2UI** para que `adk web` renderice tarjetas en lugar de texto plano.
Necesitas **ambos** elementos o solo verás JSON sin procesar:

1. Un **system prompt** que enseñe al modelo a emitir JSON de A2UI (`A2uiSchemaManager`).
2. Un **`after_model_callback`** que reempaquete ese JSON en el formato que detecta el renderizador de adk web — se incluye listo para usar como **`./template/a2ui_utils.py`**. Cópialo, no lo reescribas.

> **Usa la versión `0.8` en todas partes.** El callback reacciona a mensajes de la v0.8 (`beginRendering`, `surfaceUpdate`). Las salidas en v0.9 no se renderizarán.

## Paso 0 — Instalar el SDK

`A2uiSchemaManager` / `BasicCatalog` (Paso 2) provienen del paquete **`a2ui-agent-sdk`**:

```bash
uv add "a2ui-agent-sdk>=0.4.0,<0.5.0"
```

- **Trampa con el nombre:** el paquete se llama `a2ui-agent-sdk`, pero lo **importas** como `a2ui` (`from a2ui.schema.manager import ...`). No ejecutes `pip install a2ui` ni agregues una dependencia directa `a2ui`; no existe y `uv sync` fallará con *"a2ui was not found"*.
- **Usa `uv add`, no `uv pip install`.** `uv add` registra la dependencia en `pyproject.toml`/`uv.lock`, lo cual requiere `agents-cli deploy`. `uv pip install` solo afecta al venv: funcionará en el playground, pero luego el agente desplegado fallará con `ModuleNotFoundError: No module named 'a2ui'`.
- No es necesario anular el índice, listar manualmente `a2ui-core` ni clonar el repositorio con `git clone`. Fija la versión `0.4.x`; versiones mayores posteriores cambian las rutas de importación utilizadas a continuación.

## Paso 1 — Copiar el callback

Copia `./template/a2ui_utils.py` junto a tu archivo `agent.py`. Es autocontenido (solo requiere `google-adk` / `google-genai`).

## Paso 2 — Construir el system prompt

```python
from a2ui.schema.manager import A2uiSchemaManager
from a2ui.basic_catalog.provider import BasicCatalog

schema_manager = A2uiSchemaManager(
    version="0.8",
    catalogs=[BasicCatalog.get_config("0.8")],
)

instruction = schema_manager.generate_system_prompt(
    role_description="<qué es / qué hace tu agente>",
    workflow_description="Analyze the request and return structured UI when appropriate.",
    ui_description=(
        "Keep every surface tiny and flat: ONE Card > ONE Column > a few Text rows. "
        "Never nest a Card inside a Card. "
        "Use ONLY these components: Card, Column, Row, Text, and Image. Do not use "
        "Table or Heading (unsupported), or Buttons, actions, or forms (they do "
        "nothing in adk web). "
        "You may include one Image component, but only when you have a public https "
        "URL for the image (for example the URL an image tool returns after uploading "
        "to a public bucket). Set the Image url to that exact https link, for example "
        "{\"Image\": {\"url\": {\"literalString\": \"https://...\"}}}. Never point an "
        "Image at a bare filename, an artifact name, or a non-http(s) path. If you do "
        "not have a public URL, add a short Text line noting the image instead. "
        "No markdown in text; use the usageHint property ('h1', 'h2', 'body') for "
        "headings and emphasis. "
        "Output ONLY the raw A2UI JSON array — no prose, and never wrap it in "
        "<a2a_datapart_json> tags or 'kind'/'data'/'metadata' objects."
    ),
    include_schema=True,
    include_examples=True,
)
```

## Paso 3 — Conectar el callback en el agente

```python
from google.adk.agents import Agent
from .a2ui_utils import a2ui_callback

root_agent = Agent(
    model="<tu modelo>",
    name="<tu agente>",
    instruction=instruction,
    tools=[...],
    after_model_callback=a2ui_callback,   # <- esto es lo que hace que adk web lo renderice
)
```

## Paso 4 — Probar en adk web

```bash
uv run adk web --port 8080 --allow_origins "*" --reload_agents
```

> Usa **`uv run`**, no un comando directo `adk web`. Un `adk` directo utiliza un Python global que carece de las dependencias de tu proyecto y fallará al inicio (`cannot import name 'firestore' from 'google.cloud'` / `ModuleNotFound`). `uv run` utiliza el `.venv` del proyecto.

Inicia una **Nueva Sesión (New Session)** y solicita algo visual — deberías ver una tarjeta, no JSON. **Desactiva el Token Streaming** primero (icono de engranaje en la UI de desarrollo): si está activo, adk web muestra el JSON transmitido sin procesar y nunca lo reemplaza por la tarjeta.

## Límites del renderizador de adk web (diseña teniendo esto en cuenta)

- **Solo visualización.** Los botones, acciones y formularios se renderizan pero no hacen nada. Para una interactividad real utiliza un frontend personalizado (`build-agent-frontend`).
- **Componentes admitidos:** `Card, Column, Row, Text, Divider, List, Icon, Image`. Sin soporte para `Table` o `Heading`: construye tablas a partir de filas/columnas de `Text`, y usa `Text` + `usageHint` para los encabezados.
- **Las imágenes requieren una URL pública `http(s)`.** Un componente `Image` se renderiza inline solo si su url es un enlace `https` accesible. Si tu agente sube una imagen generada a un bucket público y pasa esa URL al `Image`, se renderizará inline dentro de la tarjeta. Una imagen guardada únicamente como un artefacto local no tiene una URL accesible, por lo que un `Image` apuntado a ese nombre de archivo mostrará un icono roto; el callback reemplaza las imágenes sin `http(s)` por una breve nota de texto y la imagen aún aparecerá en el panel de Artifacts. Consulta `build-agent-frontend` para mostrar imágenes en un frontend personalizado.
- **Estructuras pequeñas y planas se renderizan mejor.** Anidaciones profundas y tarjetas grandes se renderizan en blanco. El callback descarta superficies corruptas (JSON inválido, raíz no definida, referencias sueltas) y muestra un mensaje alternativo breve.
- **El renderizador es inestable:** incluso una superficie válida a veces puede renderizarse en blanco. Se trata de un detalle del renderizador de adk-web, no de un fallo en tu agente.

## Solución de problemas

| Lo que ves | Solución más probable |
| --- | --- |
| `ModuleNotFoundError: No module named 'a2ui'` (localmente) | El SDK no está instalado — ejecuta `uv add "a2ui-agent-sdk>=0.4.0,<0.5.0"` (Paso 0). La importación es `a2ui`, el paquete es `a2ui-agent-sdk` |
| Funciona localmente pero el agente **desplegado** arroja `No module named 'a2ui'` | Usaste `uv pip install` (solo en venv). Ejecuta `uv add "a2ui-agent-sdk>=0.4.0,<0.5.0"` para registrarlo en `pyproject.toml` y vuelve a desplegar |
| `uv sync` falla: `a2ui was not found in the package registry` | Agregaste el nombre incorrecto. Elimina `a2ui` de las dependencias; usa `a2ui-agent-sdk` en su lugar (Paso 0) |
| JSON sin formato mientras escribe | Token Streaming está ACTIVADO — desactívalo (engranaje), recarga la página (hard-refresh) e inicia una Nueva Sesión |
| JSON sin formato (sin streaming) | El callback no está conectado (`after_model_callback=a2ui_callback`), o se está usando una versión incorrecta (usa `0.8`) |
| Tarjeta en blanco | La superficie es muy grande/compleja — solicita algo más simple; o el estado de la UI se trabó — recarga la página + Nueva Sesión |

## Referencia

- Plantilla: `./template/a2ui_utils.py` (el `after_model_callback`)
- Integración de A2UI en ADK: https://adk.dev/integrations/a2ui/
- Especificación de A2UI (opcional): https://a2ui.org/
