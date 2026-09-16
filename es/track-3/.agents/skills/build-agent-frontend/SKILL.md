---
name: build-agent-frontend
description: Construye un frontend web de chat para un agente de ADK / Agent Engine desplegado y conéctalo (navegador -> proxy FastAPI -> agente a través del protocolo A2A), y luego despliégalo en Cloud Run. Diseñado para despliegues de Agent Runtime con agents-cli 1.1.0 (GA), donde el proxy se comunica mediante A2A (la ruta anterior de stream_query / operation_schemas ya no está disponible). La interfaz de chat muestra respuestas en texto sin formato Y TAMBIÉN renderiza de forma nativa las tarjetas A2UI del agente mediante un pequeño renderizador integrado, por lo que funciona tanto si el agente tiene A2UI habilitado como si no. Úsalo cuando el usuario quiera construir un frontend, una interfaz de chat o una interfaz web para su agente desplegado, conectar una interfaz de usuario a su agente, configurar el proxy FastAPI, resolver la autenticación navegador-proxy-agente, o solucionar problemas en un frontend que no puede comunicarse con un despliegue 1.1.0. A2UI es una característica del lado del agente (ver la skill enable-a2ui); esta skill lo renderiza en el frontend personalizado. Incluye una plantilla de trabajo completa (proxy FastAPI + UI de chat + renderizador A2UI) en ./template. No lo uses para la lógica del lado del agente no relacionada con la UI.
---

# Construir un frontend de chat

Dale a un agente desplegado una **interfaz de chat web** sencilla y conéctala al agente. La
estructura utiliza un **proxy FastAPI** ligero: el navegador se comunica exclusivamente con el proxy,
y el proxy se autentica ante el agente desplegado. La interfaz muestra **respuestas en texto sin formato**
y también **renderiza las tarjetas A2UI del agente** de forma nativa (mediante un pequeño renderizador integrado),
por lo que funciona tanto si el agente tiene `enable-a2ui` como si no. Se incluye un frontend completo
y funcional en **`./template`** — cópialo, no lo reconstruyas desde cero.

> **La regla principal:** copia `./template` dentro del proyecto como `frontend/` y realiza
> la menor cantidad de cambios posible. **NO** construyas una app en React, no configures un nuevo
> framework de UI ni copies un proyecto de muestra grande. La plantilla es suficiente.

## Cómo funciona

```
Navegador (chat UI)  ->  Proxy FastAPI (main.py)  ->  Agente desplegado
```

- El navegador solo se comunica con el proxy (mismo origen, sin CORS, sin credenciales de nube en el cliente).
- El proxy se autentica mediante Application Default Credentials y se comunica con el
  agente desplegado a través del protocolo A2A. agents-cli 1.1.0 (GA) despliega agentes de ADK
  en Agent Runtime como agentes A2A y ya no registra el esquema de operaciones de reasoning-engine,
  por lo que la ruta anterior `agent_engines.get(...).stream_query()` quedó obsoleta
  (su método `operation_schemas()` está vacío). El proxy obtiene la tarjeta A2A del agente
  una vez y luego envía cada mensaje utilizando el cliente de a2a-sdk (la misma ruta que
  utiliza `agents-cli run --mode a2a`). Esto funciona tanto para despliegues A2A como de ADK 1.1.0
  estándar, ya que el contenedor sirve A2A en ambos casos.
- Reutiliza un contexto A2A por usuario para que el agente recuerde la conversación.
- El proxy devuelve partes estructuradas (`text` o `a2ui`); la interfaz muestra respuestas de texto
  y renderiza las tarjetas A2UI.

## Paso 1 — Copiar la plantilla

Copia `./template` dentro del proyecto como `frontend/`. No lo reescribas — `main.py`
(el proxy) y `static/index.html` (la UI de chat) ya están diseñados para trabajar juntos.

## Paso 2 — Ejecutar localmente

Desde la carpeta `frontend/`:

```bash
pip install -r requirements.txt
export AGENT_ENGINE_RESOURCE_NAME="<pegar de deployment_metadata.json>"
export AGENT_DIRECTORY="app"   # tu agent_directory de agents-cli-manifest.yaml
python main.py     # -> http://localhost:8080
```

Envía un mensaje y luego pregunta *"¿Qué acabo de preguntar?"* para confirmar que la sesión funcione.

## Paso 3 — Desplegar en Cloud Run

El agente permanece en Agent Engine; solo el frontend se despliega en Cloud Run. Su
service account se ejecuta bajo una identidad diferente a la de tu usuario local, por lo que necesita
**`roles/aiplatform.user`** o `/chat` devolverá error 403:

```
Deploy the frontend to Cloud Run pointing at my AGENT_ENGINE_RESOURCE_NAME, and grant the Cloud Run service account roles/aiplatform.user.
```

## ¿Quieres más que un simple chat?

Despliega el chat básico primero. Para agregar controles más adelante (un panel de entrada, botones de
acción rápida, filtros), agrégalos dentro de `static/index.html` como un pequeño bloque autocontenido
que construya un mensaje y lo envíe a través del flujo de chat existente. De esa manera, la interfaz
crece sin requerir un nuevo framework ni un paso de compilación adicional.

## ¿Qué pasa con A2UI (tarjetas)?

A2UI es una funcionalidad **del lado del agente** (consulta la skill **`enable-a2ui`**). Este frontend
**lo renderiza**: a través de A2A, el agente devuelve su A2UI como partes de datos etiquetadas como
`application/json+a2ui`, el proxy convierte cada una en una parte `a2ui` y un pequeño renderizador
integrado en `static/index.html` las dibuja como tarjetas.

- Componentes admitidos: `Card, Column, Row, Text` (con `usageHint`), `Divider`,
  `List`, `Image`, `Icon`. Cualquier otro componente **muestra un texto alternativo sin fallar**, por
  lo que una respuesta nunca queda en blanco. (`Icon` utiliza la fuente web Material Symbols; si no
  puede cargar, el icono muestra su nombre como texto).
- `Image` se renderiza inline cuando su url es un enlace `http(s)` público. Si el agente sube una
  imagen generada a un bucket público y coloca esa URL en el `Image`, la imagen aparecerá en el chat.
  Un nombre de archivo de artefacto local simple no tiene una URL accesible y se mostrará como una imagen rota.
- Es **solo de visualización**, coincidiendo con `enable-a2ui`: los botones/acciones no están conectados.
- Al tener el control directo de este renderizador, es más confiable que `adk web` (sin anomalías de
  streaming ni bloqueos de pantalla en blanco).
- Mantén las salidas A2UI del agente pequeñas y planas (siguiendo la guía de `enable-a2ui`) para que el
  renderizador tenga menos margen de error.

## Si algo sale mal

- **Error 403 desde `/chat`:** falta ADC localmente o la service account de Cloud Run no tiene el rol
  `roles/aiplatform.user`. (Consulta la skill `troubleshoot-lab-setup`).
- **El agente olvida la conversación (falla "¿Qué acabo de preguntar?"):** el proxy debe reutilizar un
  contexto A2A por usuario, lo cual maneja el diccionario `_contexts` de la plantilla manteniendo
  `task.contextId` entre turnos.
- **Nada se renderiza / error de CORS:** el navegador está llamando al agente directamente; debe llamar
  exclusivamente al proxy (mismo origen).
- **`operation_schemas()` vacío, o el código anterior de `stream_query` no devuelve nada:** se trata de
  un despliegue de Agent Runtime GA 1.1.0 (A2A, sin esquema de reasoning-engine). Esta plantilla ya se
  comunica vía A2A, por lo que debes usarla en lugar de la ruta antigua del SDK.
- **Las respuestas no llegan:** confirma primero que el agente responda a través de A2A con
  `agents-cli run --url <resource-url> --mode a2a "hi"`. La estructura de las partes de respuesta de A2A
  puede variar según la versión de a2a-sdk; ajusta `_extract_parts` en `main.py` si es necesario.
- **La tarjeta A2UI se muestra como texto sin formato:** el renderizador encontró un componente no
  admitido (o una superficie con errores) y recurrió al texto de respaldo — esto es un comportamiento
  previsto, no una falla crítica. Mantén las superficies del agente dentro del subconjunto admitido.
- **`(The agent didn't return a reply.)`:** el turno no generó texto ni UI — generalmente el agente solo
  ejecutó herramientas o una herramienta se detuvo en el servidor (un problema del agente/deploy, no del
  frontend). Revisa los registros (logs) del agente.

## Referencia

- Plantilla: `./template` (`main.py`, `requirements.txt`, `static/index.html`)
- A2UI (lado del agente): la skill `enable-a2ui` · https://adk.dev/integrations/a2ui/
