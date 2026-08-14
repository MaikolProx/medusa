# Arquitectura de MEDUSA (documentación pública)

Diagrama y decisiones de diseño del sistema MEDUSA, nivel arquitectura. Los detalles de implementación privados (secretos, rutas reales, credenciales) no se documentan aquí.

## Visión general

MEDUSA es un "segundo cerebro": un vault de conocimiento personal que un agente de IA puede consultar, actualizar y usar para ejecutar tareas. Los tres pilares:

1. **Conocimiento** — documentos fuente (RAW) procesados en artículos (WIKI) con enlaces internos, indexados para búsqueda semántica.
2. **Agente(s)** — un agente principal con herramientas MCP + agentes satélite con modelos distintos.
3. **Interfaces** — chat de escritorio y voz bidireccional (Whisper + TTS local).

## Capas

### 1. Capa de conocimiento
- **RAW/**: fuentes originales (artículos, videos, notas). Sin procesar.
- **WIKI/**: conocimiento procesado, en español, con estructura consistente (secciones numeradas, tablas, wikilinks `[[...]]`).
- **Índice**: `index.md` (mapa de categorías) + `changelog.md` (historial de cambios).

### 2. Capa RAG
- **Chunking**: los artículos WIKI se dividen en fragmentos (chunks) en la fase de indexación.
- **Embeddings**: `all-MiniLM-L6-v2` (sentence-transformers), 384 dimensiones, ejecución local en CPU. Los vectores se normalizan; la similitud se computa por producto escalar.
- **Almacenamiento**: SQLite — tabla de chunks con su texto, origen y vector embebido. Un solo archivo, sin infraestructura.
- **Búsqueda**: dado un query, se genera su embedding y se comparan cosenos contra todos los chunks (fuerza bruta por numpy; corpus pequeño → suficiente y simple).

### 3. Capa de agentes
- **Agente principal**: opencode, con acceso a las herramientas MCP de MEDUSA.
- **Agentes satélite**: 5 agentes más (diferentes providers/modelos) para tareas específicas o contraste de respuestas. Distribución de providers para evitar rate limits.
- **Contexto compartido**: todos leen `MEMORY.md` (estado del sistema) + `AGENTS.md` (configuración) + `SHARED_CONTEXT.md` (estado compartido entre agentes).

### 4. Capa MCP (Model Context Protocol)
El servidor MCP expone herramientas al agente:

- **Búsqueda/guardado**: `medusa_search`, `medusa_add_knowledge`, `medusa_stats`, `medusa_memory`
- **Ejecución**: `medusa_exec`, `medusa_open_app`, `medusa_status`
- **Archivos**: `medusa_read_file`, `medusa_write_file`, `medusa_list_files`, `medusa_create_doc`
- **Sistema**: `medusa_notify`, `medusa_screenshot`, `medusa_web_search`, `medusa_learn`

### 5. Capa de voz
- **Entrada**: microfono → Whisper (transcripción) → archivo de texto intermedio → el agente lo procesa como mensaje.
- **Salida**: el agente escribe su resumen → TTS local (pyttsx3) lo lee por el altavoz.

### 6. Capa de automatización
- n8n orquesta flujos (ingestión de contenido, generación de resúmenes, tareas programadas).

## Decisiones de diseño clave

| Decisión | Por qué |
|---|---|
| Embeddings locales, no API | Coste cero, privacidad total, funciona offline. Corpus personal (cientos de docs) no necesita un vector DB distribuido. |
| SQLite con vectores embebidos | Simple, portable, sin servicios extra. Suficiente para búsqueda por fuerza bruta a esta escala. |
| Wikilinks entre artículos | Convierte notas sueltas en un grafo de conocimiento navegable; el índice por categorías mantiene el mapa. |
| Varios agentes/providers | Resiliencia ante rate limits y fallos de un solo provider; permite validación cruzada de respuestas. |
| Voz local (Whisper + pyttsx3) | Interacción manos libres sin coste de API por transcripción. |

## Límites conocidos y evolución

- Búsqueda vectorial **solo densa** a nivel de vault; el repo hermano `hybrid-rag-evals` implementa y mide la versión híbrida (BM25 + dense + RRF + rerank).
- Fuerza bruta de similitud: a partir de ~100k chunks habría que pasar a índice HNSW (pgvector o similar).
- Sin multi-usuario: es un sistema personal por diseño.

## Estructura de carpetas (resumen)

```
Cerebro/
├── RAW/              # fuentes originales
├── WIKI/             # conocimiento procesado (index.md, changelog.md)
├── scripts/          # rag_system.py, voice_system.py, MCP server
├── MEMORY.md         # estado del sistema
├── AGENTS.md         # configuración de agentes
└── SHARED_CONTEXT.md # estado compartido entre agentes
```
