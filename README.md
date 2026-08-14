# MEDUSA — Segundo Cerebro Autónomo con IA

> Sistema **real en producción** que uso a diario: una base de conocimiento personal aumentada con RAG, agentes multi-modelo, interfaz de voz y orquestación de automatizaciones. Este repo es la vitrina pública del sistema — la implementación completa corre en mi entorno local con datos privados (ver `.gitignore`).

Es la pieza de portafolio que demuestra **no un demo, sino un sistema operando**: consulto mi conocimiento, aprendo cosas nuevas, ejecuto tareas y recibo respuestas por voz todos los días.

## Qué hace

| Capacidad | Detalle técnico |
|---|---|
| **Segundo cerebro RAG** | 234+ documentos procesados (134 artículos WIKI + 100+ fuentes RAW), chunked e indexados en SQLite + embeddings locales |
| **Búsqueda semántica** | `all-MiniLM-L6-v2` (384 dims), 100% local (CPU), sin API externa — embeddings de bajo coste y privacidad total |
| **Agentes multi-modelo** | 6 agentes (Hermes, OpenCode, Codex, Blackbox, Claude, Gemini) repartidos en distintos providers para evitar rate limits y validar calidad entre modelos |
| **Interfaz de voz** | Entrada con Whisper + salida TTS local (pyttsx3); memoria de voz persistente |
| **Automatización** | Orquestación con n8n para flujos de contenido y tareas repetitivas |
| **Herramientas MCP** | 17 herramientas (búsqueda, guardado de conocimiento, ejecución, web, notificaciones, screenshots…) expuestas al agente principal |

## Arquitectura (resumen)

```
                    ┌─────────────────────────────────────────────┐
 voice_in ──────▶ │ voice_system.py ──▶ whisper (transcripción)  │
 chat ──────────▶ │                                              │
                    │        AGENTE PRINCIPAL (opencode)          │
                    │          + 5 agentes satélite              │
                    └───────┬───────────────────┬─────────────────┘
                            │                   │
                 ┌──────────▼───────┐   ┌───────▼────────┐
                 │  MCP SERVER      │   │  n8n           │
                 │  medusa_search   │   │  automatización │
                 │  medusa_add_know │   │  de flujos      │
                 └──────────┬───────┘   └─────────────────┘
                            │
                 ┌──────────▼──────────────┐
                 │  KNOWLEDGE DB (SQLite)  │
                 │  + embeddings locales   │
                 └─────────────────────────┘
```

## Detalles de implementación que importan

- **RAG de bajo coste**: el modelo de embeddings corre en CPU local; indexar 134+ artículos toma segundos. Los embeddings se guardan en la misma base SQLite que los chunks → sin infraestructura extra.
- **Knowledge pipeline**: `RAW/` (fuentes) → `WIKI/` (artículos procesados con wikilinks `[[...]]`) → indexación automática → búsqueda semántica.
- **Validación entre modelos**: la suite de 6 agentes con modelos distintos permite contrastar respuestas y no depender de un solo provider.
- **Voz bidireccional**: transcripción local + síntesis local, sin depender de la nube.

## Lo que NO está en este repo (por diseño)

- Claves de API, tokens, credenciales y secretos (ver `.gitignore`).
- Contenido privado del usuario y la base de conocimiento real.
- La orquestación completa de n8n con sus claves de cifrado.

Esto es **intencional**: un portafolio se comparte; los secretos no. Si te interesa el pipeline de RAG con evals medibles, ver mi repo hermano **`hybrid-rag-evals`** que implementa y mide el retrieval desde cero.

## Estado

- **Operativo**: consultado diariamente.
- **Datos**: 234+ docs, 134 WIKI, embeddings 384-dims locales.
- **Pendiente público**: migrar este README a una demo reproducible con datos de ejemplo (en curso).

## Stack

Python 3.12 · SQLite · sentence-transformers (all-MiniLM-L6-v2) · OpenAI Whisper · pyttsx3 · n8n · opencode (agente principal) · MCP (Model Context Protocol)
