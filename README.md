# MEDUSA

Mi segundo cerebro con IA. Lo uso todos los días: guardo conocimiento, lo consulto con búsqueda semántica, hablo con él por voz y automatizo tareas con n8n.

Este repo es la vitrina pública del sistema. La implementación completa corre en mi máquina con mis datos privados. El `.gitignore` separa lo compartible de lo que no.

## Qué hace

| Capacidad | Detalle |
|---|---|
| Segundo cerebro RAG | 234+ documentos procesados, indexados en SQLite con embeddings locales |
| Búsqueda semántica | `all-MiniLM-L6-v2` (384 dims), 100% local, sin API externa |
| Agentes multi-modelo | 6 agentes con modelos de providers distintos para no depender de uno solo |
| Voz bidireccional | Whisper para entrada, pyttsx3 para salida, sin nube |
| Automatización | Orquestación con n8n para flujos de contenido y tareas repetitivas |
| Herramientas MCP | 17 herramientas expuestas al agente principal |

## Implementación

- Los embeddings corren en CPU local. Indexar 134 artículos toma segundos. Los vectores viven en la misma SQLite que los chunks, sin infraestructura extra.
- Pipeline de conocimiento: `RAW/` (fuentes) pasa a `WIKI/` (artículos con wikilinks `[[...]]`) y de ahí a la indexación y búsqueda.
- Validación entre modelos: el mismo problema pasa por varios providers y contrasto respuestas.
- La voz es local de punta a punta: transcripción y síntesis sin depender de la nube.

## Qué no está aquí

Claves de API, tokens, credenciales, la base de conocimiento real y la orquestación completa de n8n. Los secretos no van en un portafolio.

El retrieval con evals medibles está en el repo hermano [hybrid-rag-evals](https://github.com/MaikolProx/hybrid-rag-evals).

## Estado

- Operativo, en uso diario.
- 234+ docs, 134 artículos WIKI, embeddings 384-dims locales.
- Pendiente: una demo reproducible con datos de ejemplo.

## Stack

Python 3.12 · SQLite · sentence-transformers (all-MiniLM-L6-v2) · Whisper · pyttsx3 · n8n · opencode · MCP
