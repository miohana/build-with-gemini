---
name: record-demo
description: Graba un video de demostración con captura de pantalla (.webm) del agente del usuario controlando su interfaz de chat con Playwright — abre la página, escribe consultas, espera las respuestas y guarda el video. Funciona con ambas interfaces del lab: el frontend de chat personalizado en FastAPI (localhost:8080, el predeterminado) y el ADK Playground / dev UI (puerto 8000, mediante el nombre de la app). También puede generar música de fondo instrumental con el modelo Lyria de Google (lyria-002 en Vertex AI) y mezclarla con el video ajustada a su duración — úsalo cuando el usuario pida agregar música de fondo/banda sonora/ambiente a la demo (por ejemplo, "agrega música lo-fi relajante"). Úsalo cuando el usuario quiera grabar una demo, capturar un video/grabación de pantalla de su agente, hacer un clip de demostración de su app o mostrar su agente en acción. Elige los prompts de demostración a partir de la solicitud del usuario si se proporcionan; de lo contrario, genera prompts inteligentes a partir de project_brief.md. Incluye ./record-agent.js. No es para tomar capturas de pantalla estáticas ni para construir la UI en sí.
---

# Grabar una demo de tu agente

Genera una breve **grabación de pantalla en formato `.webm`** del agente del participante controlando su interfaz de chat en un navegador headless: el script abre la página, escribe cada consulta, espera la respuesta y guarda el video. Se incluye como **`./record-agent.js`** y funciona con **cualquiera** de las interfaces del lab — el frontend personalizado en FastAPI (predeterminado) o la dev UI de ADK — adaptándose a la interfaz que el participante tenga en ejecución. Cada grabación incluye un marco distintivo de **"Gemini World Tour"** (un borde degradado + etiqueta de título) superpuesto, puede acelerarse para mantener el clip corto y — cuando se solicite — puede musicalizar el video con **música de fondo generada por IA** (modelo Lyria de Google) sincronizada con la duración exacta del video.

> **La regla principal:** no reescribas el grabador. Invoca `./record-agent.js` con los flags correspondientes de `--url`/`--app` y `--query`. Ya gestiona la configuración de Playwright, la apertura del navegador, la escritura interactiva, el marco de marca y el guardado del video.

## Qué hace que una demo sea buena (valores recomendados)

Mantenla **corta y dinámica** — un excelente clip dura aproximadamente **entre 20 y 40 segundos**. Los valores predeterminados están ajustados para ello; los parámetros principales son cuántos turnos grabas y cuánto tarda cada respuesta:

- **2 o 3 turnos.** Suficiente para contar una historia, lo bastante breve para mantenerse entretenido. No grabes más de 3.
- **Elige prompts que destaquen las mejores capacidades** (una llamada a herramientas, una búsqueda en Firestore, una imagen generada, una tarjeta A2UI) en lugar de una simple conversación informal.
- **Ajusta `--wait` al turno más lento.** Las respuestas de texto llegan en pocos segundos (el valor por defecto `--wait 12000` es suficiente); **la generación de imágenes o video requiere `--wait 30000` o más**, o el video se cortará a mitad de la respuesta.
- **Aceléralo para recortar tiempos muertos.** Si un turno tiene una espera prolongada, agrega **`--speed 1.5`** (o `2`) para que el clip final sea ágil. Esto requiere `ffmpeg`; si no está instalado, el script guardará el video a velocidad normal.
- El marco decorativo, una breve pausa inicial en la UI vacía y una retención final en la última respuesta son automáticos — no necesitas configurarlos manualmente.

## Paso 1 — Definir qué mostrar (los prompts / queries)

Elige de 2 a 3 prompts de demostración, según este orden de prioridad:

1. **Usa lo que indicó el usuario.** Si la solicitud del usuario especifica temas concretos que preguntar/mostrar (por ejemplo, "grábalo respondiendo qué hay en stock y luego generando una imagen"), conviértelos directamente en argumentos `--query`, en orden.
2. **De lo contrario, lee `project_brief.md`** en el proyecto e inventa de 2 a 3 prompts ingeniosos que luzcan al agente — idealmente aquellos que ejerciten sus mejores funciones (llamadas a herramientas, búsquedas en Firestore, imágenes generadas, tarjetas A2UI). Prefiere prompts que garanticen una respuesta visualmente rica para *este* agente.
3. **De lo contrario** (sin solicitud y sin brief), permite que el script utilice sus prompts integrados por defecto.

Mantén los prompts breves y concretos. Para dar sensación de continuidad, haz que un prompt posterior construya sobre el anterior (por ejemplo, pregunta por un artículo y luego pide ver una imagen de él).

## Paso 2 — Apuntar a la UI correcta

Selecciona el destino según lo que el participante tenga en funcionamiento:

- **Frontend personalizado (predeterminado, recomendado para una demo pulida):** escucha en `http://localhost:8080`. Simplemente omite `--url` (o pasa `--url http://localhost:8080`). Asegúrate de que lo haya iniciado (`python main.py` desde `frontend/`, ver la skill `build-agent-frontend`).
- **ADK Playground / dev UI:** pasa `--app <nombre>`, donde `<nombre>` es el directorio `app` o nombre del agente (de `agents-cli-manifest.yaml`). El script genera `http://127.0.0.1:8000/dev-ui/?app=<nombre>`. Asegúrate de que el playground esté ejecutándose. También puedes pasar una `--url` completa si el puerto varía.

En cualquier caso, confirma primero que la UI esté realmente activa y accesible — el script finalizará con un error claro si la página o el campo de entrada del chat no se encuentran.

## Paso 3 — Ejecutar el grabador

Desde la raíz del proyecto:

```bash
# Frontend personalizado (URL predeterminada), prompts adaptados a este agente:
node .agents/skills_es/record-demo/record-agent.js \
  -q "Primer prompt de demostración" \
  -q "Segundo prompt de demostración" \
  -o agent_demo.webm
```

```bash
# Dev UI de ADK en su lugar, por nombre de app:
node .agents/skills_es/record-demo/record-agent.js \
  --app mi_agente \
  -q "Primer prompt de demostración" \
  -q "Segundo prompt de demostración"
```

```bash
# Una demo con generación de imágenes: espera más tiempo y acelera el clip:
node .agents/skills_es/record-demo/record-agent.js \
  -q "¿Qué hay en inventario?" \
  -q "Genera una imagen del primer producto" \
  --wait 30000 --speed 1.5
```

Al completarse exitosamente, imprimirá `SUCCESS: Recording saved to <path>`.

## Dependencias (Playwright + Chromium, instalados una sola vez, de forma global)

El grabador requiere el paquete de npm `playwright` y su navegador Chromium. Ambos están pensados para instalarse **una sola vez y de manera global**, para que nunca se escriban en el repositorio del participante ni se descarguen en cada ejecución:

```bash
npm install -g playwright        # el paquete (global)
npx playwright install chromium  # el navegador (caché compartida por usuario)
```

En la imagen del lab, estos componentes deberían estar **preinstalados**. Si faltan, el script los instalará por ti la primera vez (paquete global + caché de navegador compartida) — esa primera ejecución tomará un minuto; las posteriores serán instantáneas. El script resuelve el paquete global mediante `npm root -g`, por lo que funciona sin una carpeta `node_modules` local. (Si la instalación global no está permitida, recurre a una local). `ffmpeg` es necesario para `--speed` y `--music`; fuera de eso es opcional. `--music` requiere adicionalmente una sesión **autenticada en gcloud** (con la que el lab ya cuenta) — consulta la sección de música a continuación.

## Referencia de opciones

| Flag | Significado | Valor predeterminado |
| --- | --- | --- |
| `-q, --query <texto>` | Mensaje a enviar; repite para múltiples turnos | dos prompts neutrales por defecto |
| `-u, --url <url>` | URL completa de la UI de chat | `http://localhost:8080/` |
| `-a, --app <nombre>` | Nombre de la app en la dev UI de ADK (construye la URL de dev-ui cuando no se especifica `--url`) | — |
| `-o, --output <archivo>` | Ruta del archivo `.webm` resultante | `./<app>_demo.webm` o `./agent_demo.webm` |
| `--delay <ms>` | Retardo de escritura por tecla en milisegundos | `40` |
| `--wait <ms>` | Tiempo de espera por respuesta (subir a 30000+ para generación multimedia) | `12000` |
| `--speed <factor>` | Acelerar el video final (requiere `ffmpeg`; ignorado si falta) | `1.0` |
| `--music <prompt>` | Generar y mezclar música de fondo con Lyria (requiere `ffmpeg` + gcloud) | desactivado |
| `--music-negative <texto>` | Prompt negativo para Lyria (elementos a excluir) | — |
| `--music-volume <0..1>` | Nivel de mezcla de la música | `0.6` |
| `--title <texto>` | Texto del título en el marco | `Gemini World Tour` |
| `--no-frame` | Desactivar el marco de marca | marco activado |
| `--headed` | Mostrar la ventana gráfica del navegador | headless (oculto) |

Ejecuta `node .agents/skills_es/record-demo/record-agent.js --help` para ver la misma lista.

## Música de fondo (Lyria) — solo cuando el usuario la solicite

La música es **opcional**. Agrégala solo cuando la solicitud del usuario lo amerite — por ejemplo: "agrega música lo-fi relajante de fondo", "ponle banda sonora", "dale un estilo animado". Cuando lo hagan, traduce su deseo en un **prompt para Lyria** y pasa `--music`:

```bash
node .agents/skills_es/record-demo/record-agent.js \
  -q "¿Qué hay en inventario?" \
  -q "Genera una imagen del primer producto" \
  --wait 30000 --speed 1.5 \
  --music "lo-fi chill hip-hop, mellow Rhodes piano, soft vinyl crackle, relaxed downtempo beat"
```

Cómo funciona: el script genera una pista instrumental con el modelo **`lyria-002`** de Google en Vertex AI y luego la mezcla debajo del video finalizado usando ffmpeg. El audio se **ajusta a la duración del video** — se recorta (o se reproduce en bucle, ya que los clips de Lyria duran ~30s) a la duración exacta y recibe un fundido de entrada (fade-in) de 1s y un fundido de salida (fade-out) de 2s que finaliza en el último cuadro, asegurando que la música comience y termine junto con el video. La música se aplica **después** de `--speed`, por lo que siempre coincide con la duración acelerada final.

Cómo escribir un buen prompt para Lyria (convierte el ambiente deseado por el usuario en esto):
- **Solo instrumental** — Lyria no genera voces. Describe género, estado de ánimo, instrumentos y tempo. Ejemplo: "upbeat corporate synth-pop, bright plucks, driving four-on-the-floor beat, optimistic" o "ambient cinematic pad, warm strings, slow and hopeful".
- Alinea el ambiente con la demo (calma/productividad → lo-fi o ambiental; lanzamiento de producto emocionante → electrónica animada).
- Usa `--music-negative` para excluir elementos ("vocals, harsh distortion") y `--music-volume` para hacerla más sutil (ej. `0.4`) o presente (`0.8`).

Requisitos: `ffmpeg` y una sesión **autenticada en gcloud** con un proyecto configurado — el entorno del lab ya cuenta con ambos (`gcloud auth login` / ADC y proyecto). `lyria-002` se ejecuta en `us-central1`; el script apunta a esa región automáticamente y obtiene el proyecto desde `GOOGLE_CLOUD_PROJECT` o `gcloud config`. Si la generación de música falla por cualquier motivo, el script **igualmente guardará el video** (sin música) e imprimirá la causa.

## El marco decorativo (branded frame)

El grabador inyecta un marco de **"Gemini World Tour"** en la página antes de grabar: un borde degradado alrededor del viewport y una etiqueta de título en la parte superior. Se dibuja como una capa con `pointer-events: none`, por lo que nunca bloquea la interfaz del agente, y se vuelve a aplicar si la aplicación se vuelve a renderizar. Los estilos residen en `./assets/overlay.css` y el logotipo en `./assets/logo.svg` — edítalos para cambiar el diseño. Modifica el texto con `--title "…"` o elimina el marco completamente con `--no-frame`.

## Ajuste de resultados

- **Las respuestas se cortan / el video finaliza a mitad de la respuesta:** incrementa `--wait` (la generación de imágenes o video puede demorar mucho más que el texto — prueba con `--wait 30000` o más).
- **El clip se siente lento o demasiado largo:** agrega `--speed 1.5` (o `2`) para reducir tiempos muertos y/o graba menos turnos. Apunta a un total de ~20 a 40 segundos.
- **La música está muy alta o muy baja:** ajusta `--music-volume` (ej. `0.4` más suave, `0.8` más fuerte). Cambia la vibra reformulando el prompt de `--music`.
- **La escritura parece muy rápida o muy lenta:** ajusta `--delay`.
- **No se encuentra nada / no se puede conectar a la UI:** el frontend o el playground no se están ejecutando, o están en un puerto diferente — inícialos o especifica la `--url` correcta.
- **Observar el proceso en vivo:** agrega `--headed` para ver cómo el navegador interactúa con la interfaz en tiempo real.

## Solución de problemas

- **"Could not load <url>":** la interfaz no se está ejecutando o el puerto es incorrecto. Inicia el frontend (`python main.py`) o el playground, o corrige `--url`/`--app`.
- **"Could not find a chat input":** la página cargó pero no es la UI de chat (URL equivocada), o su campo de entrada difiere de los compatibles (`#input`, el textarea de la dev UI o un input genérico / contenteditable). Apunta a la página adecuada.
- **Falla la instalación de Playwright/Chromium:** el script autoinstala ambos (globalmente); si el entorno lo bloquea, instala manualmente con `npm install -g playwright && npx playwright install chromium` y vuelve a ejecutar.
- **`--speed` no tuvo efecto:** `ffmpeg` no está instalado, por lo que el video se guardó a velocidad normal. Instala ffmpeg y vuelve a ejecutar, o consérvalo tal cual.
- **No aparece el marco en el video:** faltan los archivos de `./assets` o pasaste `--no-frame`. La grabación seguirá funcionando — únicamente sin el marco decorativo.
- **No se agregó música:** el script imprime la razón y de todos modos guarda el video. Causas comunes: falta `ffmpeg`; gcloud no está autenticado o no hay proyecto configurado (`gcloud auth login` + `gcloud config set project …`); o la cuenta no tiene acceso a `lyria-002` (Vertex AI / Agent Platform). Un error `403`/`401` de la API de Lyria indica problemas de autenticación o permisos.

## Referencia

- Script de grabación: `./record-agent.js`
- Recursos del marco: `./assets/overlay.css`, `./assets/logo.svg`
- Modelo de música: Lyria `lyria-002` en Vertex AI (`us-central1`), instrumental, clips de ~30s
- Frontend al que apunta por defecto: la skill `build-agent-frontend` (`localhost:8080`)
