<div align="center">

<img src="es/track-3/assets/build-with-gemini-banner.png" alt="Build with Gemini" width="100%" />

# 🚀 Build with Gemini · World Tour

### Starter kits, labs and workshop skills for the Build with Gemini World Tour — available in multiple languages.

Pick your language, pick your track, open [Antigravity](https://antigravity.google) and start building.

<br/>

![Build with Gemini](https://img.shields.io/badge/Build%20with%20Gemini-World%20Tour-4285F4?logo=google&logoColor=white)
![Languages](https://img.shields.io/badge/Languages-ES%20%C2%B7%20PT--BR-EA4335)
![Google Cloud](https://img.shields.io/badge/Google%20Cloud-Agent%20Platform-4285F4?logo=googlecloud&logoColor=white)
![Built with ADK](https://img.shields.io/badge/Built%20with-ADK%20%2B%20agents--cli-34A853)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)

<sub>️ <a href="https://google.github.io/agents-cli/guide/getting-started/">agents-cli</a> · 🤖 <a href="https://google.github.io/adk-docs/">ADK</a></sub>

</div>

---

## 🌎 Choose your language

| Language | Available tracks | Start here |
| --- | --- | --- |
| 🇪🇸 **Español** | Track 3 | [`es/track-3`](es/track-3/README.md) |
| 🇧🇷 **Português (Brasil)** | Track 3 | [`pt-br/track-3`](pt-br/track-3/README.md) |

> [!NOTE]
> Every language folder is self-contained: the README, the `.agents/` skills and all templates are fully localized. Clone the repo once and work entirely inside the folder for your language.

---

## 📚 Table of contents

- [🌎 Choose your language](#-choose-your-language)
- [🎯 What is Build with Gemini?](#-what-is-build-with-gemini)
- [🛤️ Tracks](#️-tracks)
- [🗂️ Repository structure](#️-repository-structure)
- [🚀 Quick start](#-quick-start)
- [🧠 What's inside a track](#-whats-inside-a-track)
- [📚 Resources](#-resources)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)

---

## 🎯 What is Build with Gemini?

**Build with Gemini World Tour** is a hands-on workshop series where participants build real applications on Google Cloud with Gemini, [Antigravity](https://antigravity.google) and the [Agent Development Kit (ADK)](https://google.github.io/adk-docs/).

This repository hosts the **starter kits** used during the labs — the agent skills, MCP tool configuration and templates that turn Antigravity into a guided workshop instructor — plus a gallery of what participants built with them.

---

## 🛤️ Tracks

| Track | Focus | Status |
| --- | --- | --- |
| **Track 1** | Coming soon | 🚧 Not in this repo yet |
| **Track 2** | Coming soon | 🚧 Not in this repo yet |
| **Track 3** | Agent-first apps on Google Cloud: ADK + `agents-cli`, Memory Bank, Firestore, Cloud Storage, RAG, A2UI and Cloud Run | ✅ Available in `es` and `pt-br` |

---

## 🗂️ Repository structure

```text
build-with-gemini/
├── README.md              # you are here — language hub
├── es/                    # 🇪🇸 Español
│   └── track-3/
│       ├── README.md      # track guide + project gallery
│       ├── assets/
│       └── .agents/       # skills + MCP config loaded by Antigravity
└── pt-br/                 # 🇧🇷 Português (Brasil)
    └── track-3/
        ├── README.md
        ├── assets/
        └── .agents/
```

Each new language gets its own top-level folder (`<lang>/`), and each track lives inside it as `<lang>/track-N/`.

---

## 🚀 Quick start

```bash
git clone https://github.com/miohana/build-with-gemini
cd build-with-gemini

# pick the folder for your language + track
cd pt-br/track-3      # or: cd es/track-3

agy
```

On startup, Antigravity scans the `.agents/` folder in that track and automatically loads its skills and tools. At the AGY prompt:

```text
/skills            # see the installed skills
/mcp               # confirm the firebase + google-developer-knowledge tools are connected
```

**Prerequisites** (the lab workstation comes with all of this preinstalled):

- A **Google Cloud project** with billing enabled
- **[Antigravity](https://antigravity.google)** (`agy`), the coding agent that loads the skills
- **[agents-cli](https://google.github.io/agents-cli/guide/getting-started/)**, built on top of the [ADK](https://google.github.io/adk-docs/)
- Authenticated gcloud: `gcloud auth login` and `gcloud auth application-default login`
- A **personal GitHub account** for the final publish & submit step

---

## 🧠 What's inside a track

Each track folder ships a `.agents/` directory that teaches Antigravity how to run the lab:

- **Skills** — self-loading instruction sets that guide the agent through each workshop step (picking a project, setting up Memory Bank, building RAG, enabling A2UI, deploying a frontend, recording a demo, publishing to GitHub).
- **MCP tools** — [`mcp_config.json`](es/track-3/.agents/mcp_config.json) wires up the **Firebase** and **Google Developer Knowledge** [MCP](https://modelcontextprotocol.io/) servers, authenticated with your gcloud credentials, so the agent looks things up instead of guessing.

See the per-track README for the full skill list and a breakdown of the architecture.

---

## 📚 Resources

- [Antigravity](https://antigravity.google)
- [agents-cli](https://google.github.io/agents-cli/guide/getting-started/)
- [Agent Development Kit (ADK)](https://google.github.io/adk-docs/)
- [Gemini Enterprise Agent Platform](https://docs.cloud.google.com/gemini-enterprise-agent-platform)
- [A2UI](https://adk.dev/integrations/a2ui/)

---

## 🤝 Contributing

**Built something?** Publish it with the `publish-to-github` skill and submit it through the form it gives you. Submissions can earn you swag, and featured projects are added to the gallery in the track README for your language.

**Found a bug?** If something is broken in a skill or in the lab, please [open an issue](https://github.com/miohana/build-with-gemini/issues).

**Adding a language?** Create a new top-level folder using the language code (e.g. `en/`, `fr/`), copy the track you want to localize into it, translate the README and the `.agents/` skills, and add a row to the [language table](#-choose-your-language) above. Keep the `name:` field in each skill's YAML frontmatter untranslated — it is the skill ID.

---

## 📄 License

This is not an officially supported Google product and is provided solely for demonstration purposes for the Build with Gemini workshop.
