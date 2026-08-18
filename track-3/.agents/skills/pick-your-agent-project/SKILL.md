---
name: pick-your-agent-project
description: Ayuda de manera interactiva a un participante del workshop a decidir y diseñar qué agente construir. Úsalo cuando el usuario esté eligiendo un proyecto, haciendo una lluvia de ideas sobre un agente, diga "no sé qué construir" o "¿qué debería hacer?", O pida diseñar/planificar/"ayúdame a diseñar"/"ayúdame a hacer"/"ayúdame a construir" una idea específica de agente (por ejemplo, "ayúdame a diseñar un agente planificador de viajes", "ayúdame a crear un asistente de recetas", "diseña mi aplicación de agentes") ANTES de estructurar cualquier código — este es el paso de planificación/resumen (brief), no de implementación. También úsalo cuando quiera comprobar si su idea realmente aprovechará las herramientas de Google del workshop (memory/sessions, function tools, storage + A2UI, generación de imágenes, code sandbox, evaluation). Guía la elección de un dominio y una verificación rápida de cobertura de herramientas, y luego redacta un breve project brief (resumen del proyecto). No lo uses para implementar o programar el agente en sí (eso viene después del brief).
---

# Elige tu proyecto de agente

Ayuda a un participante del workshop a elegir *qué* agente construir durante el resto del lab.
Tu tarea es guiar una breve lluvia de ideas interactiva (con una duración estimada de 5 a 10 minutos
de diálogo), definir una idea concreta, verificar que aproveche las herramientas de Google
que se enseñan en el workshop y entregarle un breve **project brief** (resumen del proyecto)
sobre el cual pueda construir.

Esta es una tarea de *facilitación*, no de construcción. NO escribas código para el agente,
no generes la estructura del proyecto ni instales nada aquí. Solo ayúdalo a decidir.

## La idea clave a transmitir: las herramientas definen la forma

La versión más intimidante de "¿qué debería construir?" es una página en blanco. Pero no está en blanco.
Las herramientas que se enseñan en este workshop definen implícitamente la *forma* de un buen proyecto.
Diseña a partir de las herramientas y el proceso se volverá sencillo:

| La herramienta que aprenderán | ...significa que el agente debería tener |
| --- | --- |
| Sessions & Memory | algo que valga la pena **recordar** sobre el usuario (preferencias, historial) |
| Function tools | algo real que pueda **hacer o consultar**, no solo chatear |
| Storage + A2UI | una **colección/catálogo** de elementos que se rendericen bien como tarjetas/tablas |
| Generación de imágenes o música | un dominio donde **generar un elemento visual o sonoro** sea útil |
| Code sandbox | una necesidad ocasional de **calcular** algo |
| Evaluation | un **criterio claro de "buena respuesta"** — poder definir cómo se ve una respuesta correcta |

Así, el arquetipo es: *un agente conversacional con estado y un catálogo de elementos que puede mostrarte, sobre los cuales puede actuar y para los cuales puede generar elementos visuales.* (Por esta razón se eligió como ejemplo el agente de "marketplace / invernadero" — aprovecha cada herramienta de forma natural).

El participante NO está limitado a un marketplace. Cualquier dominio funciona siempre que cumpla con el estándar mínimo indicado a continuación. Motívalo a elegir un dominio que realmente le interese — menos restricciones significa más diversión y mayor motivación.

## La verificación rápida de cobertura de herramientas (gut-check)

Guíalo a través de estas cinco preguntas sobre su idea. Preséntalas como disparadores de conversación, no como un examen:

1. **Memory** — ¿Qué recuerda sobre *ti* entre turnos/sesiones?
2. **Tools** — ¿Qué puede realmente *hacer* o *consultar* (una acción o búsqueda real)?
3. **Catalog** — ¿Qué colección de elementos tiene? (se renderiza excelente como tarjetas/tablas)
4. **Visuals** — ¿Qué imagen podría generar para ti?
5. **Compute** — ¿Cuándo podría necesitar calcular o ejecutar código?

**Estándar mínimo (debe cumplirse para ser un buen proyecto):** respuestas sólidas a las preguntas **#1 y #2**.
Esos son los pilares fundamentales que todos construyen. Si una idea no define qué recuerda o qué *hace*, es un chatbot, no un agente — anímalo a replantearla.

**Cobertura de extensión / Stretch (deseable):** #3, #4, #5. Apunta a que al menos dos de estas sean viables para que cuente con un amplio "menú de extensión" durante la etapa del hackathon. Si su idea solo cumple con el mínimo, está bien — simplemente indícale qué herramientas opcionales encajarán de forma natural más adelante y cuáles no.

## Cómo conducir la lluvia de ideas

Haz unas cuantas preguntas a la vez y adáptate — no envíes toda la lista de verificación de golpe.

1. **Plantea un dominio.** Pregúntale qué le apasiona (un pasatiempo, un reto laboral, un juego, un dominio que conozca bien). Si no se le ocurre nada, ofrece el menú de abajo como inspiración.
2. **Dale la forma del arquetipo.** Reformula su idea como "un agente conversacional que ayuda a [alguien] a [hacer X] con una colección de [Y]". Llévalo a una definición de una sola línea (one-liner).
3. **Realiza la verificación rápida (gut-check).** Revisa las cinco preguntas. Completen las respuestas juntos.
4. **Comprueba el estándar mínimo.** Confirma que #1 y #2 sean sólidas. Si no lo son, ajusta la idea (generalmente: dale una acción real o algo que recordar) en lugar de descartarla.
5. **Mapea el menú de extensión.** Explícale qué herramientas opcionales (A2UI, generación de imágenes, sandbox, Cloud Storage) se adaptan a su dominio — esto es lo que explorará una vez completados los pilares principales.
6. **Escribe el archivo de especificación.** Guarda el brief como `project_brief.md` en el espacio de trabajo (siguiendo el formato a continuación) — el paso de construcción apuntará `agents-cli` hacia este archivo.

Mantén el avance continuo. Lo perfecto es enemigo de lo empezado — si tiene una idea viable que cumple con el estándar, confírmala y permítele refinarla mientras construye.

## Menú de dominios (inspiración, no una lista estricta para elegir)

Ofrece estas opciones solo si se queda sin ideas. Cada una está comprobada para ejercitar bien las herramientas:

- Concierge de viajes (recuerda tus preferencias; busca/planifica viajes; tarjetas de itinerario)
- Chef personal / agente de recetas (memoria de restricciones dietéticas; búsqueda de recetas; imágenes de platillos)
- Creador de personajes o grupos de fantasía (recuerda a tu grupo; consulta estadísticas; retratos de personajes)
- Buscador de bienes raíces / departamentos (memoria de presupuesto y preferencias; búsqueda de propiedades; tarjetas de listados)
- Juego de colección estilo Pokémon (tu colección; acciones para capturar/intercambiar; arte de criaturas)
- Dungeon Master de D&D (memoria de la campaña; dados/reglas; imágenes de escenas)
- Entrenador de ejercicio / equipamiento (memoria de objetivos; consulta de planes; cálculo de progreso en sandbox)
- Tienda de cuidado de plantas / invernadero (el ejemplo del workshop — historial de cuidados; inventario; imágenes de plantas)

## Salida: escribe el archivo de especificación

Cuando el brief esté completo, **escríbelo en un archivo llamado `project_brief.md`** en el espacio de trabajo (no te limites a imprimirlo en el chat). El paso de construcción apuntará `agents-cli` hacia este archivo, por lo que actúa como el puente entre la toma de decisiones y la construcción. Utiliza este formato:

```
# My agent: <name>
One-liner: A conversational agent that helps <who> <do what> with a catalog of <what>.

Tool coverage:
- Memory: <what it remembers>
- Tools: <the real action(s)/lookup(s)>
- Catalog/UI: <collection to render as cards/tables, or "n/a">
- Image gen: <what visual, or "n/a">
- Sandbox: <what computation, or "n/a">

Core rails (everyone): memory, tools, eval, deploy, frontend
My stretch menu (pick later): <the optional tools that fit>
First eval question: <one example of a "good" response for this agent>
```

La "primera pregunta de evaluación" siembra la mentalidad de evaluación desde temprano — definir qué es una "buena respuesta" es la parte más desafiante y valiosa del lab.

## Después de escribir el brief: DETENTE

Escribir `project_brief.md` marca el final de esta skill. **NO** construyas, estructures, instales ni escribas código para el agente — y **NO** ofrezcas ni preguntes si deseas hacerlo ("¿quieres que lo construya ahora?", "¿debo implementar esto?", "¿listo para crear la estructura?"). Ofrecerte a construir se sale del guion: el lab está diseñado para que el participante construya el agente de forma incremental en los pasos siguientes, y adelantarse arruina el objetivo del workshop.

Finaliza tu turno informándole que el brief ha sido guardado y que continúe con el siguiente paso del lab (puede editar `project_brief.md` antes si lo desea). Luego detente.
