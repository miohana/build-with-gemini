<div align="center">

<img src="assets/build-with-gemini-banner.png" alt="Build with Gemini" width="100%" />

# 🚀 Build with Gemini · Track 3

### El kit de inicio para el Track 3 del Build with Gemini World Tour, y una muestra de lo que los participantes construyeron con él.

Clona este repositorio, abre [Antigravity](https://antigravity.google) y crea tu propia app agent-first en Google Cloud. Cada proyecto en la [galería a continuación](#-proyectos-destacados) fue construido de la misma manera: prototipado con Antigravity y `agents-cli`, equipado con Memory, herramientas, almacenamiento y RAG, desplegado en Agent Platform y con una interfaz web en Cloud Run.

<br/>

![Build with Gemini](https://img.shields.io/badge/Build%20with%20Gemini-World%20Tour-4285F4?logo=google&logoColor=white)
![Track 3](https://img.shields.io/badge/Track%203-Agent--First%20Apps-EA4335)
![Google Cloud](https://img.shields.io/badge/Google%20Cloud-Agent%20Platform-4285F4?logo=googlecloud&logoColor=white)
![Built with ADK](https://img.shields.io/badge/Built%20with-ADK%20%2B%20agents--cli-34A853)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)
![Projects](https://img.shields.io/badge/Projects-0-blue)

<sub>️ <a href="https://google.github.io/agents-cli/guide/getting-started/">agents-cli</a> · 🤖 <a href="https://google.github.io/adk-docs/">ADK</a></sub>

</div>

---

## 📚 Tabla de contenidos

- [🧩 Anatomía de un proyecto de Track 3](#-anatomía-de-un-proyecto-de-track-3)
- [🏷️ Leyenda de capacidades](#️-leyenda-de-capacidades)
- [📂 Proyectos destacados](#-proyectos-destacados)
  - [🛍️ Agentes de comercio y marketplace](#️-agentes-de-comercio-y-marketplace)
  - [🍳 Agentes de comida y recetas](#-agentes-de-comida-y-recetas)
  - [✈️ Agentes de viajes y locales](#️-agentes-de-viajes-y-locales)
  - [💪 Agentes de salud, fitness y bienestar](#-agentes-de-salud-fitness-y-bienestar)
  - [📚 Agentes de aprendizaje y conocimiento](#-agentes-de-aprendizaje-y-conocimiento)
  - [🎨 Agentes creativos y de medios](#-agentes-creativos-y-de-medios)
  - [🏢 Agentes de productividad y empresariales](#-agentes-de-productividad-y-empresariales)
  - [🧪 Experimentales y otros](#-experimentales-y-otros)
- [🧠 Qué hay en este repositorio](#-qué-hay-en-este-repositorio)
- [🧰 Construye el tuyo](#-construye-el-tuyo)
- [📚 Recursos](#-recursos)
- [🤝 Contribuir](#-contribuir)
- [📄 Licencia](#-licencia)

---

## 🧩 Anatomía de un proyecto de Track 3

Cada aplicación en esta colección está construida a partir del mismo conjunto de bloques de construcción de Google Cloud presentados en el lab. Una vez que entiendas esta estructura, podrás interpretar cualquier proyecto aquí de un vistazo:

| Capa | Qué hace | Impulsado por |
|---|---|---|
| 🤖 **El Agente** | El bucle central de razonamiento | [ADK](https://google.github.io/adk-docs/) + [`agents-cli`](https://google.github.io/agents-cli/guide/getting-started/), estructurado con [Antigravity](https://antigravity.google) |
| 🧠 **Memory** | Recuerda información entre sesiones | [Agent Platform Memory Bank](https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/memory-bank) |
| 🗄️ **Datos estructurados** | Inventario, registros, listas | [Firestore](https://console.cloud.google.com/firestore) |
| 🖼️ **Archivos y blobs** | Imágenes, contenido multimedia, recursos | [Cloud Storage](https://console.cloud.google.com/storage) |
| 🔧 **Herramientas** | Ejecutan acciones reales y obtienen datos reales | ADK function tools |
| 📖 **RAG** | Respuestas fundamentadas en tus documentos | [Vertex AI RAG Engine](https://console.cloud.google.com/agent-platform/rag) |
| 🎨 **Generación de medios** | Crea imágenes (y video) bajo demanda | `gemini-3.1-flash-lite-image` (Nano Banana 2 Lite) · Omni (video) |
| 🧪 **Code sandbox** | Ejecuta código generado de forma segura | Ejecución de código en Agent Platform |
| 🪟 **Agent-first UI** | Tarjetas y tablas en lugar de texto sin formato | [A2UI](https://adk.dev/integrations/a2ui/) |
| 🌐 **Frontend** | Una interfaz web para compartir | Proxy FastAPI en [Cloud Run](https://cloud.google.com/run) |

---

## 🏷️ Leyenda de capacidades

Cada proyecto a continuación está etiquetado con los bloques de construcción que utiliza, para que puedas encontrar exactamente el patrón que deseas aprender:

`🧠 Memory` · `🗄️ Firestore` · `🖼️ Storage` · `🔧 Tools` · `📖 RAG` · `🎨 Image Gen` · `🎬 Video` · `🧪 Sandbox` · `🪟 A2UI` · `🌐 Cloud Run`

---

## 📂 Proyectos destacados

Una muestra de lo que los participantes del workshop construyeron con este lab. Las entradas se agregan aquí desde el formulario de envío para swag y galería después de cada evento, por lo que las categorías a continuación comienzan vacías y se van llenando con el tiempo. Explóralas en busca de inspiración, o [envía el tuyo](#-contribuir) una vez que hayas publicado tu proyecto con la skill `publish-to-github`.

<!--
Agrega una entrada por proyecto, en este formato:
- 🌿 **[Nombre del Proyecto](https://github.com/su-usuario/su-repo)**: descripción de una línea de lo que hace. <br/> <sub>`🗄️ Firestore` · `🎨 Image Gen` · `🪟 A2UI`, por [@usuario](https://github.com/usuario)</sub>

Elige etiquetas de la Leyenda de capacidades de arriba. Incrementa el contador del badge "Projects" en la parte superior cuando agregues uno.
-->

### 🛍️ Agentes de comercio y marketplace

### 🍳 Agentes de comida y recetas

### ✈️ Agentes de viajes y locales

### 💪 Agentes de salud, fitness y bienestar

### 📚 Agentes de aprendizaje y conocimiento

### 🎨 Agentes creativos y de medios

### 🏢 Agentes de productividad y empresariales

### 🧪 Experimentales y otros

---

## 🧠 Qué hay en este repositorio

La carpeta `.agents/` le enseña a Antigravity cómo construir agentes en Google Cloud.

### Skills

Una **skill** es un conjunto de instrucciones que se carga automáticamente cuando es relevante, permitiendo que el agente ejecute el flujo de trabajo correctamente en menos pasos en lugar de tener que redescubrirlo cada vez.

| Skill | Qué hace |
| --- | --- |
| [`pick-your-agent-project`](.agents/skills/pick-your-agent-project/SKILL.md) | Haz una lluvia de ideas para tu aplicación y redacta un brief del proyecto |
| [`troubleshoot-lab-setup`](.agents/skills/troubleshoot-lab-setup/SKILL.md) | Verifica tu entorno y soluciona errores comunes de configuración |
| [`memory-bank-setup`](.agents/skills/setup-memory-bank/SKILL.md) | Agrega memoria entre sesiones a tu agente con Vertex AI Memory Bank |
| [`rag-engine-setup`](.agents/skills/build-rag/SKILL.md) | Fundamenta tu agente en documentos con un corpus serverless de Vertex AI RAG Engine |
| [`enable-a2ui`](.agents/skills/enable-a2ui/SKILL.md) | Haz que tu agente responda con tarjetas visuales enriquecidas (A2UI) en la UI de desarrollo de ADK |
| [`build-agent-frontend`](.agents/skills/build-agent-frontend/SKILL.md) | Genera un frontend de chat con FastAPI y despliégalo en Cloud Run |
| [`record-demo`](.agents/skills/record-demo/SKILL.md) | Graba un video de demostración con marca para tu agente, con banda sonora opcional generada por IA |
| [`publish-to-github`](.agents/skills/publish-to-github/SKILL.md) | Publica tu proyecto finalizado en tu propio GitHub y envíalo para obtener swag |

### Herramientas preconfiguradas (MCP)

[`.agents/mcp_config.json`](.agents/mcp_config.json) conecta dos servidores de [Model Context Protocol](https://modelcontextprotocol.io/) que se autentican con tus credenciales de gcloud, para que el agente pueda consultar información en lugar de adivinar:

- **Firebase**: trabaja directamente con Firestore y otros servicios de Firebase
- **Google Developer Knowledge**: acceso fundamentado a la documentación oficial de Google (Cloud, Firebase, ADK, Agent Platform)

### Estructura

```text
.agents/
├── mcp_config.json    # Servidores MCP de Firebase + Developer Knowledge
└── skills/            # las skills del workshop listadas arriba
```

---

## 🧰 Construye el tuyo

**Requisitos previos** (la estación de trabajo del lab viene con todo esto preinstalado; lo necesitarás si lo ejecutas en tu propia máquina):

- Un **proyecto de Google Cloud** con facturación habilitada
- **[Antigravity](https://antigravity.google)** (`agy`), el agente de codificación que carga las skills mencionadas arriba
- **[agents-cli](https://google.github.io/agents-cli/guide/getting-started/)**, construido sobre el [Agent Development Kit (ADK)](https://google.github.io/adk-docs/)
- gcloud autenticado: `gcloud auth login` y `gcloud auth application-default login`
- Una **cuenta personal de GitHub** para el paso final de publicación y envío

**Inicio rápido:**

```bash
git clone https://github.com/miohana/build-with-gemini
cd build-with-gemini/track-3
agy
```

Al iniciar, Antigravity escanea la carpeta `.agents/` y carga automáticamente las skills y herramientas anteriores. En el prompt de AGY:

```text
/skills            # ver las skills instaladas
/mcp               # confirmar que las herramientas de firebase + google-developer-knowledge estén conectadas
```

```text
Verify my setup.   # ejecuta la skill troubleshoot-lab-setup para verificar tu entorno
```

---

## 📚 Recursos

- [Antigravity](https://antigravity.google)
- [agents-cli](https://google.github.io/agents-cli/guide/getting-started/)
- [Agent Development Kit (ADK)](https://google.github.io/adk-docs/)
- [Gemini Enterprise Agent Platform](https://docs.cloud.google.com/gemini-enterprise-agent-platform)

---

## 🤝 Contribuir

**¿Construiste algo?** Publícalo con la skill `publish-to-github` y envíalo mediante el formulario que te proporciona. Con tus envíos puedes obtener swag, y los proyectos destacados se agregarán a la galería de [Proyectos destacados](#-proyectos-destacados) de arriba.

**¿Encontraste un error?** Si encuentras algún problema en una skill o en el lab, por favor [abre un issue](https://github.com/miohana/build-with-gemini/issues).

---

## 📄 Licencia

Este no es un producto de Google con soporte oficial y se proporciona exclusivamente con fines de demostración para el workshop Build with Gemini.
