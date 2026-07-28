# surreal-skills

[![CI](https://github.com/24601/surreal-skills/actions/workflows/ci.yml/badge.svg)](https://github.com/24601/surreal-skills/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.7.1-blue.svg)](https://github.com/24601/surreal-skills/releases)
[![skills.sh](https://img.shields.io/badge/skills.sh-surrealdb-purple.svg)](https://skills.sh)

Habilidad experta de SurrealDB 3 para agentes de codificación de IA. Sigue **SurrealDB v3.1.4+**.
Cobertura completa de SurrealQL, modelado de datos multi-modelo, recorrido de grafos, búsqueda vectorial (HNSW + DiskANN), seguridad, despliegue, ajuste de rendimiento, integración de SDK, extensiones WASM, MCP integrado (v3.1+), SurrealMCP independiente, SurrealKit, Surrealist, n8n, CodeMirror y flujos de trabajo de habilidades para agentes.

## Características

- **Maestría en SurrealQL** -- Referencia completa del lenguaje con sentencias, funciones, operadores e modismos.
- **Modelado de datos multi-modelo** -- Patrones de documentos, grafos, vectores, relacionales, series temporales y geoespaciales en un solo esquema.
- **Consultas de grafos** -- Creación de aristas (edges) y recorridos de primera clase sin JOINs.
- **Búsqueda vectorial** -- Índices HNSW y DiskANN, funciones de similitud y patrones de pipeline RAG.
- **MCP integrado (v3.1+)** -- `surreal mcp` stdio y HTTP `/mcp` para hosts de agentes de IA; surrealmcp independiente para herramientas extendidas.
- **Seguridad** -- Permisos a nivel de fila, autenticación JWT, control de acceso a nivel de namespace/base de datos/registro.
- **Despliegue** -- Selección del motor de almacenamiento, Docker, Kubernetes, endurecimiento para producción.
- **Rendimiento** -- Estrategias de indexación, análisis EXPLAIN, operaciones por lotes, pooling de conexiones.
- **Integraciones con más de 12 SDKs** -- JavaScript/TypeScript, Python, Go, Rust, Java, Kotlin, .NET, C, PHP, Swift (iOS/macOS/visionOS), Ruby.
- **Extensiones WASM Surrealism** -- Funciones y analizadores personalizados compilados desde Rust (nuevo en v3).
- **Cobertura de SurrealML** -- Formato de artefactos `.surml` (vista previa/inestable), límites de lanzamiento PyPI-vs-GitHub y advertencia de descarga de librerías nativas.
- **SurrealMCP para agentes de IA** -- MCP integrado en SurrealDB 3.1+ más `surrealdb/surrealmcp` v0.4.0 independiente.
- **Herramientas de edición** -- `surrealql-language-server` v0.1.6 oficial, gramática tree-sitter, CodeMirror v1.0.6 y guías de descubrimiento para extensiones de VS Code / Cursor / Windsurf / VSCodium, JetBrains, Neovim, Helix, Sublime Text, Zed.
- **Integración con LangChain (solo Python)** -- `langchain-surrealdb` 0.2.1 (Python) -- uso de vector store; el paquete JS `@langchain/surrealdb` fue retirado en v1.4.1 (no existe en npm).
- **GitHub Action `surrealdb/setup-surreal@v2`** -- Action oficial para ejecutar SurrealDB dentro de flujos de trabajo de CI (la documentación v1.4.0 que describía setup-surreal como un bootstrap de CLI fue retirada en v1.4.2).
- **Ecosistema completo** -- IDE Surrealist, Surreal-Sync CDC, sistema de archivos para agentes SurrealFS, herramientas de esquema SurrealKit, nodo comunitario de n8n, repo oficial de Agent Skills y límite de la hoja de ruta de Spectron.
- **Chequeos de salud e introspección** -- Script doctor e introspección de esquema para cualquier instancia de SurrealDB.
- **Soporte universal para agentes** -- Funciona con más de 30 agentes de codificación de IA a través de skills.sh.

## Instalación

### Claude Code (recomendado)

**Opción 1 -- Instalar como habilidad de Claude Code (global)**

```bash
npx skills add 24601/surreal-skills -a claude-code -g -y
```

Esto instala la habilidad globalmente para que esté disponible en cada sesión de Claude Code. El flag `-g` instala globalmente, `-y` confirma automáticamente los prompts.

**Opción 2 -- Instalar por proyecto vía CLAUDE.md**

Clona el repo y referéncialo desde el `CLAUDE.md` de tu proyecto:

```bash
git clone https://github.com/24601/surreal-skills.git ~/.claude/skills/surrealdb
```

Luego añade al `CLAUDE.md` de tu proyecto (o `~/.claude/CLAUDE.md` para global):

```markdown
# SurrealDB Skill

@import ~/.claude/skills/surrealdb/AGENTS.md
```

O incluye la referencia en línea:

```markdown
# SurrealDB Skill

For SurrealDB work, read the rules at ~/.claude/skills/surrealdb/rules/ and
use the scripts at ~/.claude/skills/surrealdb/scripts/ for health checks
and schema introspection.
```

**Opción 3 -- Añadir como comando slash personalizado de Claude Code**

Crea `~/.claude/commands/surrealdb.md`:

```markdown
Load the SurrealDB 3 skill from ~/.claude/skills/surrealdb/AGENTS.md
and use its rules for all SurrealDB architecture, development, and operations tasks.
Available rules: surrealql, data-modeling, graph-queries, vector-search, security,
deployment, performance, sdks, surrealism, surrealist, surreal-sync, surrealfs,
surrealkit, surrealmcp, surrealml, editor-tooling, langchain, ecosystem-integrations, gotchas.
```

Luego invócalo con `/surrealdb` en cualquier sesión de Claude Code.

**Opción 4 -- Comandos slash con alcance de proyecto**

Añade comandos específicos de SurrealDB a tu proyecto:

```bash
mkdir -p .claude/commands
```

Crea `.claude/commands/surreal-doctor.md`:

```markdown
Run the SurrealDB health check: uv run ~/.claude/skills/surrealdb/scripts/doctor.py
Report any issues found and suggest fixes based on the deployment rules.
```

Crea `.claude/commands/surreal-schema.md`:

```markdown
Introspect the current SurrealDB schema: uv run ~/.claude/skills/surrealdb/scripts/schema.py introspect
Analyze the output using the data-modeling rules and suggest improvements.
```

### Otros Agentes de IA

```bash
# skills.sh (universal -- funciona con todos los agentes soportados)
npx skills add 24601/surreal-skills

# Amp
npx skills add 24601/surreal-skills -a amp -g -y

# Codex
npx skills add 24601/surreal-skills -a codex -g -y

# Gemini CLI
npx skills add 24601/surreal-skills -a gemini-cli -g -y

# OpenCode
npx skills add 24601/surreal-skills -a opencode -g -y

# Pi (badlogic/pi-mono)
npx skills add 24601/surreal-skills -a pi -g -y

# OpenClaw / Clawdbot
npx skills add 24601/surreal-skills -a openclaw -g -y
```

### GitHub Copilot (habilidades de agente nativas)

Copilot soporta el [estándar Agent Skills](https://agentskills.io/) nativamente en VS Code, Copilot CLI y el agente de codificación de Copilot. Esta habilidad incluye un archivo `.github/skills/surrealdb/SKILL.md` nativo de Copilot que se carga automáticamente cuando tu prompt está relacionado con SurrealDB.

**Opción 1 -- Nivel de proyecto (recomendado para equipos)**

Copia toda la habilidad en el directorio `.github/skills/` de tu proyecto:

```bash
# Desde el repo surreal-skills
cp -r .github/skills/surrealdb <tu-proyecto>/.github/skills/surrealdb
cp -r rules/ <tu-proyecto>/.github/skills/surrealdb/rules/
```

Copilot descubre esto automáticamente, no necesita configuración. Escribe `/surrealdb` en el chat o deja que Copilot lo cargue automáticamente cuando detecte contexto de SurrealQL.

**Opción 2 -- Personal (todos los proyectos)**

Clona en `~/.copilot/skills/`:

```bash
git clone https://github.com/24601/surreal-skills.git ~/.copilot/skills/surrealdb
```

O añade una ubicación de búsqueda personalizada en los ajustes de VS Code:

```json
{
  "chat.agentSkillsLocations": [
    "~/.copilot/skills"
  ]
}
```

**Opción 3 -- Usar el menú `/skills`**

Escribe `/skills` en el chat de Copilot para abrir el menú de Configuración de Habilidades, luego navega hasta el directorio clonado de `surrealdb`.

### Otras Integraciones de IDE

```bash
# Cursor -- añade la habilidad a .cursor/skills/ (mismo estándar Agent Skills)
cp -r .github/skills/surrealdb <tu-proyecto>/.cursor/skills/surrealdb

# Windsurf -- añade AGENTS.md al final de .windsurfrules
cat AGENTS.md >> .windsurfrules

# Cline / Continue -- referéncialo en tu configuración
# Añade la ruta de AGENTS.md a la configuración de tu system prompt
```

### Instalación Manual

```bash
# Clonar en cualquier ubicación
git clone https://github.com/24601/surreal-skills.git ~/.claude/skills/surrealdb

# Verificar instalación
uv run ~/.claude/skills/surrealdb/scripts/doctor.py --check
```

## Inicio Rápido

> **Advertencia de credenciales**: Los ejemplos a continuación usan `root/root` solo para **desarrollo local**. Nunca uses credenciales predeterminadas en instancias de producción o compartidas.

```bash
# Iniciar SurrealDB en memoria SOLO PARA DESARROLLO LOCAL
surreal start memory --user root --pass root --bind 127.0.0.1:8000

# Conectar vía CLI REPL (desarrollo local)
surreal sql --endpoint http://localhost:8000 --user root --pass root --ns test --db test

# Crear registros con SurrealQL
CREATE person:alice SET name = 'Alice', email = 'alice@example.com';
CREATE person:bob SET name = 'Bob', email = 'bob@example.com';

# Crear aristas de grafo
RELATE person:alice->follows->person:bob SET since = time::now();

# Recorrer el grafo
SELECT ->follows->person.name AS following FROM person:alice;

# Ejecutar el chequeo de salud
uv run scripts/doctor.py
```

## Arquitectura

```
surreal-skills/
  SKILL.md              # Manifiesto de la habilidad (frontmatter + cuerpo)
  AGENTS.md             # Briefing estructurado para el agente
  README.md             # Este archivo
  LICENSE               # Licencia MIT
  scripts/
    onboard.py          # Asistente de configuración / manifiesto de capacidades
    doctor.py           # Chequeo de salud (CLI, servidor, auth, almacenamiento)
    schema.py           # Introspección y exportación de esquema
  rules/
    surrealql.md        # Referencia del lenguaje SurrealQL
    data-modeling.md    # Patrones de diseño de esquema multi-modelo
    graph-queries.md    # Recorridos de grafo y patrones RELATE
    vector-search.md    # Índices vectoriales, búsqueda de similitud, RAG
    security.md         # Permisos, auth, control de acceso
    deployment.md       # Motores de almacenamiento, Docker, K8s, producción, GitHub Action setup-surreal
    performance.md      # Índices, EXPLAIN, optimización
    sdks.md             # Integración de SDKs oficiales (12+ lenguajes)
    surrealism.md       # Sistema de extensiones WASM (nuevo en v3)
    surrealml.md        # Alcance de SurrealML y límites de paquetes
    surrealmcp.md       # Servidor Model Context Protocol
    editor-tooling.md   # LSP, tree-sitter, extensiones de IDE
    langchain.md        # Integración con LangChain Python
    ecosystem-integrations.md # n8n, CodeMirror, Agent Skills, límite Spectron
    surrealist.md       # IDE/GUI Surrealist
    surreal-sync.md     # Herramienta de migración CDC
    surrealfs.md        # Sistema de archivos para agentes de IA
    surrealkit.md       # Sincronización de esquema de estado deseado y despliegues
  skills/
    surrealism/
    surreal-sync/
    surrealfs/
    surrealkit/
    surrealmcp/
```

## Reglas

| Regla | Descripción |
|------|-------------|
| `surrealql.md` | Referencia completa de SurrealQL: CREATE, SELECT, UPDATE, DELETE, RELATE, INSERT, UPSERT, LIVE SELECT, DEFINE, REMOVE, INFO, subconsultas, transacciones, futures, todas las funciones integradas, notas de migración v2-a-v3 |
| `data-modeling.md` | Patrones de diseño de esquema: IDs de registro (tipados, generados, compuestos), tipos de campo, schemafull vs schemaless, estrategias de normalización, diseño multi-modelo (documento + grafo + vector en un esquema), datos de series temporales y geoespaciales |
| `graph-queries.md` | Creación de aristas con RELATE, operadores de recorrido (-> saliente, <- entrante, <-> bidireccional), expresiones de ruta, consultas recursivas, filtrado y agregación en aristas, DEFINE TABLE TYPE RELATION específico de grafos |
| `vector-search.md` | Definiciones de campos vectoriales, creación de índices HNSW y brute-force, métricas de distancia (coseno, euclidiana, manhattan, minkowski), funciones vector::similarity, patrones de pipeline RAG, búsqueda híbrida combinando vector + filtrado de metadatos |
| `security.md` | Permisos a nivel de fila con predicados WHERE, DEFINE ACCESS para JWT y auth basada en registros, DEFINE USER para usuarios del sistema, alcance de permisos de namespace/base de datos/tabla, variables de entorno $auth y $session, patrones de flujo de autenticación |
| `deployment.md` | Métodos de instalación (gestor de paquetes, Docker, binario), selección de motor de almacenamiento (memory, RocksDB, SurrealKV con viaje en el tiempo, TiKV para distribuido), Docker Compose y Helm charts de Kubernetes, endurecimiento de producción, backup/restore, niveles de log, monitoreo, GitHub Action `surrealdb/setup-surreal@v2` para CI |
| `performance.md` | Estrategias de índices (únicos, analizadores de búsqueda de texto completo, vectores HNSW), sentencia EXPLAIN para análisis de consultas, operaciones por lotes, pooling de conexiones, compromisos del motor de almacenamiento según carga, consultas paralelas, límites de recursos, ratios computación-almacenamiento |
| `sdks.md` | Uso de SDKs oficiales para JavaScript/TypeScript (Node, Deno, Bun, navegador), Python, Go, Rust, Java, Kotlin, .NET, C, PHP, Swift (iOS / macOS / visionOS), Ruby: configuración de conexión (HTTP vs WebSocket), flujos de autenticación, operaciones CRUD, suscripciones a consultas en vivo, manejo de registros tipados, patrones de error |
| `surrealism.md` | Sistema de extensiones WASM Surrealism introducido en SurrealDB 3: SDK de Rust para autoría, registro de funciones personalizadas, creación de analizadores personalizados, compilación de módulos a wasm32-unknown-unknown, despliegue en instancias activas, versionado, pruebas |
| `surrealml.md` | Resumen del alcance de SurrealML (vista previa/inestable; límite GitHub v0.1.2 vs PyPI 0.0.4; advertencia de descarga de librerías nativas en el setup). Formato de artefactos `.surml`, extras de pip soportados, patrones estables que no dependen de la superficie inestable de ML |
| `surrealmcp.md` | Servidor Model Context Protocol SurrealMCP: instalación verificada (Cargo desde fuente / Docker; no está en crates.io o npm), formato CLI de `surrealmcp start`, convenciones de variables de entorno, catálogo de herramientas agrupadas por README original, guías de configuración de host para Claude Desktop / Cursor / Copilot / Zed / n8n |
| `editor-tooling.md` | `surrealql-language-server` v0.1.3 oficial, límite de la comunidad `surql-lsp`, `surrealql-tree-sitter`, paquetes de CodeMirror y guías de descubrimiento de extensiones de editor |
| `langchain.md` | Integración con LangChain: `langchain-surrealdb` 0.2.1 (Python; `langchain-core ~= 1.1.0`, `surrealdb ~= 1.0.8` SDK v1) para pipelines de LangChain 1.1+: SurrealDB como vector store vía constructor `SurrealDBVectorStore(embeddings, conn)`. La sección de JS fue retirada en v1.4.1 (`@langchain/surrealdb` no está en npm) |
| `ecosystem-integrations.md` | El nodo comunitario de n8n (`@surrealdb/n8n-nodes-surrealdb`), índice de documentación oficial de frameworks de IA, límite de la hoja de ruta de Spectron, paquetes de CodeMirror y repo `surrealdb/agent-skills` |
| `surrealist.md` | IDE y GUI Surrealist: diseñador de esquemas con edición visual de tablas, editor de consultas con resaltado de sintaxis y auto-completado, visualizador de grafos para relaciones, explorador de tablas, perfiles de conexión, importación/exportación, embebido en aplicaciones |
| `surreal-sync.md` | Herramienta de migración CDC Surreal-Sync: conectores de origen (PostgreSQL, MySQL, MongoDB, etc.), SurrealDB como destino, captura de datos incremental, reglas de traducción de esquema, orquestación de flujo de migración, resolución de conflictos, monitoreo |
| `surrealfs.md` | Sistema de archivos SurrealFS para agentes de IA: almacenamiento de archivos respaldado por SurrealDB, gestión de metadatos con consultas SurrealQL, estructuras de directorios, versionado de archivos, patrones de API aptos para agentes, integración con frameworks de agentes de IA |
| `surrealkit.md` | Gestión de esquema SurrealKit para aplicaciones SurrealDB: sincronización de estado deseado para desarrollo, migraciones basadas en rollouts para bases de datos compartidas/producción, semillas (seeds) y pruebas declarativas de esquema/permisos/API |
| `gotchas.md` | Errores comunes y riesgos verificados contra v3.1.4+: migración, permisos, grafos, vectores, SurrealQL, MCP, SDKs |

## Scripts

| Script | Uso | Descripción |
|--------|-------|-------------|
| `onboard.py` | `uv run scripts/onboard.py --check` | Verificar requisitos previos (CLI surreal, Python, uv, conectividad del servidor) |
| `onboard.py` | `uv run scripts/onboard.py --agent` | Salida del manifiesto de capacidades en JSON para integración de agentes |
| `doctor.py` | `uv run scripts/doctor.py` | Chequeo de salud completo: versión de CLI, alcanzabilidad del servidor, auth, namespace, base de datos, motor de almacenamiento |
| `doctor.py` | `uv run scripts/doctor.py --check` | Paso rápido aprobado/fallido (código de salida 0 = saludable, 1 = problemas) |
| `doctor.py` | `uv run scripts/doctor.py --endpoint URL` | Verificar un endpoint específico de SurrealDB |
| `schema.py` | `uv run scripts/schema.py introspect` | Volcado completo del esquema de todas las tablas, campos, índices, eventos, accesos |
| `schema.py` | `uv run scripts/schema.py tables` | Listar todas las tablas con recuentos de campos/índices |
| `schema.py` | `uv run scripts/schema.py table <name>` | Inspeccionar una sola tabla en detalle |
| `schema.py` | `uv run scripts/schema.py export --format surql` | Exportar esquema como sentencias DEFINE reproducibles |
| `schema.py` | `uv run scripts/schema.py export --format json` | Exportar esquema como JSON estructurado |
| `check_upstream.py` | `uv run scripts/check_upstream.py` | Comparar repositorios upstream contra la instantánea de la habilidad; muestra qué cambió |
| `check_upstream.py` | `uv run scripts/check_upstream.py --stale` | Mostrar solo repos con nuevos commits desde la instantánea |
| `check_upstream.py` | `uv run scripts/check_upstream.py --json` | Salida solo en JSON (sin tabla Rich) |

Todos los scripts siguen la convención de salida dual: stderr para salida humana formateada con Rich, stdout para JSON legible por máquinas.

## Sub-Habilidades

### Surrealism (Extensiones WASM)

Nuevo en SurrealDB 3. Extiende la base de datos con funciones y analizadores personalizados escritos en Rust, compilados a WebAssembly y desplegados en instancias activas. La regla `rules/surrealism.md` cubre todo el SDK de Surrealism, autoría de módulos, compilación, despliegue y flujo de pruebas.

### Surreal-Sync (Migración CDC)

Herramienta de Captura de Datos Cambiantes (CDC) para migrar datos desde bases de datos externas (PostgreSQL, MySQL, MongoDB, y otras) hacia SurrealDB. Soporta sincronización incremental, traducción de esquemas y resolución de conflictos. Ver `rules/surreal-sync.md`.

### SurrealFS (Sistema de Archivos para Agentes de IA)

Una abstracción de sistema de archivos construida sobre SurrealDB, diseñada para flujos de trabajo de agentes de IA. Almacena archivos con metadatos enriquecidos consultables vía SurrealQL, versiona archivos automáticamente e intégralos con frameworks de agentes. Ver `rules/surrealfs.md`.

### SurrealKit (Sincronización de Esquema y Rollouts)

Gestión de esquemas de SurrealDB para equipos de aplicación. Usa `sync` de estado deseado para entornos desechables, manifiestos de `rollout` para bases de datos compartidas y de producción, `seed` para datos de prueba y `test` para verificaciones declarativas de esquema, permisos y API. Ver `rules/surrealkit.md`.

### SurrealMCP (Servidor Model Context Protocol)

**Integrado (SurrealDB 3.1+):** `surreal mcp` stdio o HTTP `POST /mcp` — no requiere instalación separada.
**Independiente:** `surrealdb/surrealmcp` v0.4.0 oficial para herramientas extendidas y ayudantes de nube.
Instálalo de forma independiente desde la fuente (`cargo install --path .`) o Docker; no está en crates.io o npm.
Ver `rules/surrealmcp.md` y `skills/surrealmcp/SKILL.md`.

### SurrealML (Inferencia de ML en Base de Datos) -- vista previa / inestable

`surrealml` tiene un lanzamiento de GitHub `v0.1.2`, pero PyPI todavía expone `surrealml` 0.0.4 como la última versión a fecha de 2026-06-17. La guía local estable sigue siendo el formato de artefactos `.surml` y los extras `[sklearn]`, `[torch]`, `[tensorflow]`. La configuración actual de Python upstream puede descargar librerías nativas desde GitHub Releases hacia `~/surrealml_deps` a menos que `LOCAL_BUILD=TRUE`; fija y audita antes del uso en producción. Ver `rules/surrealml.md`.

### Herramientas de Edición

`surrealql-language-server` v0.1.6 oficial, límite de la comunidad `surql-lsp` v0.1.1, gramática tree-sitter, `@surrealdb/codemirror` / `@surrealdb/lezer` v1.0.6, y entradas de guía para extensiones de VS Code / Cursor / Windsurf / VSCodium, IDEs de JetBrains, Neovim, Helix, Sublime Text, Zed y Emacs. Ver `rules/editor-tooling.md`.

### Integración con LangChain (Solo Python)

`langchain-surrealdb` 0.2.1 (Python; `langchain-core ~= 1.1.0`, `surrealdb ~= 1.0.8` SDK v1) para pipelines de LangChain 1.1+: SurrealDB como un vector store vía el constructor `SurrealDBVectorStore(embeddings, conn)`. El paquete npm `@langchain/surrealdb`, `AsyncSurrealDBVectorStore`, `SurrealChatMessageHistory`, `SurrealHybridRetriever` y las factorías `from_endpoint`/`from_client` de v1.4.0 fueron retirados en v1.4.1. Ver `rules/langchain.md`.

### Integraciones del Ecosistema

`rules/ecosystem-integrations.md` rastrea nuevas superficies del ecosistema y guías: el nodo comunitario de n8n con scope oficial (`@surrealdb/n8n-nodes-surrealdb` v0.6.0), el índice de documentación oficial de frameworks de IA, Spectron / Agent Memory Context solo como hoja de ruta, paquetes de CodeMirror y `surrealdb/agent-skills`. Trata las páginas de frameworks de IA que no sean LangChain como simples guías hasta que su forma de paquete/API sea re-verificada.

## Casos de Uso

### Backend de API

Usa SurrealDB como el almacén de datos principal para APIs REST o GraphQL. Define tablas con validación schemafull, configura permisos a nivel de fila para seguridad multi-inquilino, conecta vía SDK de JavaScript o Python sobre WebSocket para consultas en vivo en tiempo real.

### Aplicación en Tiempo Real

Aprovecha LIVE SELECT para suscripciones de datos basadas en push. Los clientes reciben los cambios a medida que ocurren sin necesidad de hacer polling. Combínalo con conexiones SDK WebSocket para aplicaciones de chat, editores colaborativos, dashboards y sistemas de notificación.

### Analítica de Grafos

Modela relaciones complejas (redes sociales, jerarquías organizativas, árboles de dependencias, grafos de conocimiento) usando RELATE y tablas de aristas tipadas. Recorre rutas de profundidad arbitraria con operadores `->`. Filtra y agrega en cada salto sin escribir JOINs.

### Búsqueda Vectorial y RAG

Almacena embeddings de documentos junto al contenido. Crea índices vectoriales HNSW con métricas de distancia configurables. Consulta con `vector::similarity::cosine` para búsqueda semántica. Construye pipelines de generación aumentada por recuperación (RAG) que combinen la similitud vectorial con el filtrado de metadatos en una sola consulta SurrealQL.

### IoT y Series Temporales

Ingiere datos de sensores de alto volumen con tablas schemaless. Usa campos datetime y consultas de rango para análisis de series temporales. Agrega con funciones matemáticas y temporales integradas. El motor de almacenamiento SurrealKV permite consultas de viaje en el tiempo para acceder al estado histórico en cualquier momento.

### Aplicaciones Geoespaciales

Almacena tipos de geometría (puntos, polígonos, multipuntos) como valores nativos de SurrealDB. Usa funciones geo integradas (`geo::distance`, `geo::bearing`, `geo::area`, `geo::contains`) para consultas espaciales. Combínalo con otros modelos de datos: una sola consulta puede recorrer un grafo, filtrar por ubicación y clasificar por similitud vectorial.

### Migración de Datos

Migra desde PostgreSQL, MySQL, MongoDB u otras bases de datos usando Surreal-Sync CDC. Traduce esquemas automáticamente, sincroniza incrementalmente y valida con introspección de esquemas. Para actualizaciones de SurrealDB v2-a-v3, usa la exportación/importación de surreal con las notas de migración en `rules/surrealql.md`.

### Extensiones WASM

Extiende SurrealDB con lógica de negocio personalizada usando Surrealism. Escribe funciones y analizadores en Rust, compila a WASM y despliega sin reiniciar el servidor. Los casos de uso incluyen validación personalizada, puntuación específica del dominio, tokenizadores propietarios y funciones de agregación especializadas.

### Sistema de Archivos de Agente de IA

Usa SurrealFS como un sistema de archivos persistente y consultable para flujos de trabajo de agentes de IA. Los agentes pueden almacenar y recuperar archivos con metadatos enriquecidos, realizar consultas entre archivos con SurrealQL y aprovechar el sistema de permisos de SurrealDB para el control de acceso multi-agente.

### Búsqueda de Texto Completo

Define analizadores personalizados (tokenizadores, filtros, stemmers) y crea índices de búsqueda en campos de texto. Consulta con predicados de búsqueda de texto completo que se integran con el resto de SurrealQL: combina la búsqueda de texto con el recorrido de grafos, la similitud vectorial y los filtros relacionales en una sola sentencia.

## Configuración

Establece estas variables de entorno para configurar los scripts de la habilidad. Todas son opcionales con valores predeterminados sensatos.

| Variable | Descripción | Predeterminado |
|----------|-------------|---------|
| `SURREAL_ENDPOINT` | URL del servidor SurrealDB | `http://localhost:8000` |
| `SURREAL_USER` | Nombre de usuario de root o namespace | `root` |
| `SURREAL_PASS` | Contraseña de root o namespace | `root` |
| `SURREAL_NS` | Namespace predeterminado | `test` |
| `SURREAL_DB` | Base de datos predeterminada | `test` |

Estas variables también son reconocidas por la CLI de surreal y los SDKs oficiales de SurrealDB.

## Procedencia de la Fuente

Esta habilidad fue actualizada el **2026-05-03** desde estas fuentes upstream. Usa `check_upstream.py` para detectar qué ha cambiado desde esta instantánea para actualizaciones incrementales.

| Repositorio | Versión | SHA | Fecha Instantánea | Reglas Afectadas |
|------------|---------|-----|---------------|----------------|
| [surrealdb/surrealdb](https://github.com/surrealdb/surrealdb) | v3.0.5 (main hacia v3.1.0-alpha) | `a97d3af85d79` | 2026-04-29 | surrealql, data-modeling, security, performance, deployment, surrealism |
| [surrealdb/surrealist](https://github.com/surrealdb/surrealist) | surrealist-v3.8.5 | `3699b2d09b62` | 2026-05-01 | surrealist |
| [surrealdb/surrealdb.js](https://github.com/surrealdb/surrealdb.js) | v2.0.3 | `f0fa3cd7d8fb` | 2026-03-25 | sdks |
| [surrealdb/surrealdb.py](https://github.com/surrealdb/surrealdb.py) | v2.0.0 (GA) | `6e45a820d27c` | 2026-05-02 | sdks |
| [surrealdb/surrealdb.go](https://github.com/surrealdb/surrealdb.go) | v1.4.0 (main) | `aef39d3a439f` | 2026-04-30 | sdks |
| [surrealdb/surreal-sync](https://github.com/surrealdb/surreal-sync) | v0.3.4 | `59b3166910f0` | 2026-03-11 | surreal-sync |
| [surrealdb/surrealfs](https://github.com/surrealdb/surrealfs) | -- | `0008a3a94dbe` | 2026-01-29 | surrealfs |
| [surrealdb/surrealkit](https://github.com/surrealdb/surrealkit) | v0.6.0 (pre-lanzamiento) | `28f5a1c9d20c` | 2026-05-03 | surrealkit |

Documentación: [surrealdb.com/docs](https://surrealdb.com/docs) instantánea del 2026-05-03.

Procedencia legible por máquina: [`SOURCES.json`](SOURCES.json).

## Registros

Esta habilidad está publicada en múltiples registros de habilidades para agentes:

| Registro | Comando de Instalación |
|----------|----------------|
| [skills.sh](https://skills.sh) | `npx skills add 24601/surreal-skills` |
| [ClawHub](https://clawhub.ai) | `npx clawhub install surrealdb` |
| [OpenClaw / Clawdbot](https://github.com/openclaw) | `clawhub install surrealdb` |
| GitHub | `git clone https://github.com/24601/surreal-skills.git` |

## Contribución

Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para la configuración del entorno de desarrollo, el estilo de código y el proceso de PR.

## Seguridad

Para reportar una vulnerabilidad, utiliza [GitHub Security Advisories](https://github.com/24601/surreal-skills/security/advisories/new). Consulta [SECURITY.md](SECURITY.md) para más detalles.

Esta habilidad declara las siguientes propiedades de seguridad en el frontmatter de `SKILL.md`:

| Propiedad | Valor | Significado |
|----------|-------|---------|
| `no_network` | **false** | `doctor.py`/`schema.py` se conectan al endpoint de SurrealDB especificado por el usuario (WebSocket). `check_upstream.py` llama a la API de GitHub vía CLI `gh`. No hay otras llamadas a terceros. |
| `no_credentials` | **false** | Los scripts aceptan `SURREAL_USER`/`SURREAL_PASS` para auth de DB. No se almacenan credenciales en la habilidad misma. |
| `no_env_write` | true | Los scripts no modifican variables de entorno |
| `no_file_write` | **false** | `schema.py` puede escribir `schema.surql` cuando se usa `--output-dir`, y `onboard.py --interactive` puede escribir un archivo `.env` local solo tras confirmación explícita del usuario. |
| `no_shell_exec` | false | Los scripts invocan la CLI `surreal` y la CLI `gh` |
| `scripts_auditable` | true | Todos los scripts son Python legible sin ofuscación |
| `scripts_use_pep723` | true | Dependencias declaradas en línea vía PEP 723, sin requirements.txt |
| `no_obfuscated_code` | true | Sin código ofuscado, codificado o cifrado |
| `no_binary_blobs` | true | Sin binarios compilados o archivos WASM |
| `no_minified_scripts` | true | Sin JavaScript minificado o código comprimido |
| `no_curl_pipe_sh` | true | La habilidad documenta solo instalaciones vía gestor de paquetes y contenedores. No se incluyen comandos de instalación pipe-to-shell. |

### Variables de Entorno Requeridas

Declaradas en `SKILL.md` `requires.env_vars`:

| Variable | Sensible | Predeterminado | Propósito |
|----------|-----------|---------|---------|
| `SURREAL_ENDPOINT` | No | `http://localhost:8000` | URL del servidor SurrealDB |
| `SURREAL_USER` | **Sí** | `root` | Nombre de usuario de autenticación |
| `SURREAL_PASS` | **Sí** | `root` | Contraseña de autenticación |
| `SURREAL_NS` | No | `test` | Namespace predeterminado |
| `SURREAL_DB` | No | `test` | Base de datos predeterminada |

### Binarios Requeridos

Declarados en `SKILL.md` `requires.binaries`:

| Binario | Requerido | Instalación |
|--------|----------|---------|
| `surreal` | Sí | `brew install surrealdb/tap/surreal` |
| `python3` (>=3.10) | Sí | Gestor de paquetes del sistema |
| `uv` | Sí | `brew install uv` o `pip install uv` |
| `docker` | No | Opcional para instancias contenedorizadas |
| `gh` | No | Opcional -- solo usado por `check_upstream.py` para comparar SHAs de repos upstream vía API de GitHub |

### Seguridad de los Scripts

- Todos los nombres de tabla proporcionados por el usuario se validan contra `[a-zA-Z_][a-zA-Z0-9_]*` antes de la interpolación en consultas SurrealQL (previene inyección de SurrealQL).
- `doctor.py` y `schema.py` se conectan únicamente al endpoint de SurrealDB especificado por el usuario (vía variable de entorno o flag de CLI).
- `check_upstream.py` llama a la API de GitHub vía CLI `gh` para comparar SHAs de repos upstream (script de mantenimiento opcional, no necesario para uso normal).
- No se envían datos a servicios de terceros.
- Las etiquetas de advertencia de credenciales están presentes en todos los ejemplos de `root/root`.

## Licencia

[MIT](LICENSE)

## Créditos

Construido para la comunidad de [SurrealDB](https://surrealdb.com). SurrealDB es creado y mantenido por [SurrealDB Ltd](https://github.com/surrealdb/surrealdb).

Publicado en [skills.sh](https://skills.sh), [ClawHub](https://clawhub.ai) y [GitHub](https://github.com/24601/surreal-skills).
