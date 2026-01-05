# MCP Atlassian

![PyPI Version](https://img.shields.io/pypi/v/mcp-atlassian)
![PyPI - Downloads](https://img.shields.io/pypi/dm/mcp-atlassian)
![PePy - Total Downloads](https://static.pepy.tech/personalized-badge/mcp-atlassian?period=total&units=international_system&left_color=grey&right_color=blue&left_text=Total%20Downloads)
[![Run Tests](https://github.com/sooperset/mcp-atlassian/actions/workflows/tests.yml/badge.svg)](https://github.com/sooperset/mcp-atlassian/actions/workflows/tests.yml)
![License](https://img.shields.io/github/license/sooperset/mcp-atlassian)
[![Docs](https://img.shields.io/badge/docs-mintlify-blue)](https://personal-1d37018d.mintlify.app)

Servidor Model Context Protocol (MCP) para productos Atlassian (Confluence y Jira). Soporta despliegues tanto en Cloud como en Server/Data Center.

https://github.com/user-attachments/assets/35303504-14c6-4ae4-913b-7c25ea511c3e

<details>
<summary>Demo de Confluence</summary>

https://github.com/user-attachments/assets/7fe9c488-ad0c-4876-9b54-120b666bb785

</details>

## Inicio Rápido

### 1. Obtén tu Token de API

Ve a https://id.atlassian.com/manage-profile/security/api-tokens y crea un token.

> Para Server/Data Center, usa un Token de Acceso Personal en su lugar. Consulta [Autenticación](https://personal-1d37018d.mintlify.app/docs/authentication).

### 2. Configura tu IDE

#### Opción A: Claude Desktop

Agrega esto a tu configuración MCP de Claude Desktop (`~/Library/Application Support/Claude/claude_desktop_config.json` en macOS, `%APPDATA%\Claude\claude_desktop_config.json` en Windows):

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "JIRA_URL": "https://tu-empresa.atlassian.net",
        "JIRA_USERNAME": "tu.email@empresa.com",
        "JIRA_API_TOKEN": "tu_token_api",
        "CONFLUENCE_URL": "https://tu-empresa.atlassian.net/wiki",
        "CONFLUENCE_USERNAME": "tu.email@empresa.com",
        "CONFLUENCE_API_TOKEN": "tu_token_api"
      }
    }
  }
}
```

#### Opción B: Cursor

Agrega esto a tu configuración MCP de Cursor (`~/Library/Application Support/Cursor/mcp.json` en macOS, `%APPDATA%\Cursor\mcp.json` en Windows):

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "JIRA_URL": "https://tu-empresa.atlassian.net",
        "JIRA_USERNAME": "tu.email@empresa.com",
        "JIRA_API_TOKEN": "tu_token_api",
        "CONFLUENCE_URL": "https://tu-empresa.atlassian.net/wiki",
        "CONFLUENCE_USERNAME": "tu.email@empresa.com",
        "CONFLUENCE_API_TOKEN": "tu_token_api"
      }
    }
  }
}
```

> **Python 3.14 aún no es compatible.** Usa `["--python=3.12", "mcp-atlassian"]` como args si es necesario.

> **Usuarios de Server/Data Center**: Usa `JIRA_PERSONAL_TOKEN` en lugar de `JIRA_USERNAME` + `JIRA_API_TOKEN`. Consulta [Autenticación](https://personal-1d37018d.mintlify.app/docs/authentication) para más detalles.

### 3. Comienza a Usar

Pide a tu asistente de IA:
- **"Encuentra issues asignados a mí en el proyecto PROJ"**
- **"Busca en Confluence documentación de onboarding"**
- **"Crea un ticket de bug para el problema de login"**
- **"Actualiza el estado de PROJ-123 a Hecho"**

## Instalación con Docker Compose (Windows 11)

Para una guía detallada paso a paso de cómo usar este servidor MCP con Docker Compose en Windows 11 e integrarlo con Claude Desktop y Cursor, consulta:

**[📖 Guía de Docker Compose para Windows 11](./DOCKER-WINDOWS-11-es.md)**

La guía incluye:
- ✅ Instalación de Docker Desktop en Windows 11
- ✅ Configuración con Docker Compose
- ✅ Integración con Claude Desktop
- ✅ Integración con Cursor
- ✅ Solución de problemas comunes

## Documentación

La documentación completa está disponible en **[personal-1d37018d.mintlify.app](https://personal-1d37018d.mintlify.app)**.

La documentación también está disponible en [formato llms.txt](https://llmstxt.org/), que los LLMs pueden consumir fácilmente:
- [`llms.txt`](https://personal-1d37018d.mintlify.app/llms.txt) — mapa del sitio de documentación
- [`llms-full.txt`](https://personal-1d37018d.mintlify.app/llms-full.txt) — documentación completa

| Tema | Descripción |
|------|-------------|
| [Instalación](https://personal-1d37018d.mintlify.app/docs/installation) | uvx, Docker, pip, desde fuente |
| [Autenticación](https://personal-1d37018d.mintlify.app/docs/authentication) | Tokens API, PAT, OAuth 2.0 |
| [Configuración](https://personal-1d37018d.mintlify.app/docs/configuration) | Configuración IDE, variables de entorno |
| [Transporte HTTP](https://personal-1d37018d.mintlify.app/docs/http-transport) | SSE, streamable-http, multi-usuario |
| [Referencia de Herramientas](https://personal-1d37018d.mintlify.app/docs/tools-reference) | Todas las herramientas de Jira y Confluence |
| [Solución de Problemas](https://personal-1d37018d.mintlify.app/docs/troubleshooting) | Problemas comunes y depuración |

## Compatibilidad

| Producto | Despliegue | Soporte |
|----------|------------|---------|
| Confluence | Cloud | Totalmente soportado |
| Confluence | Server/Data Center | Soportado (v6.0+) |
| Jira | Cloud | Totalmente soportado |
| Jira | Server/Data Center | Soportado (v8.14+) |

## Herramientas Principales

| Jira | Confluence |
|------|------------|
| `jira_search` - Buscar con JQL | `confluence_search` - Buscar con CQL |
| `jira_get_issue` - Obtener detalles de issue | `confluence_get_page` - Obtener contenido de página |
| `jira_create_issue` - Crear issues | `confluence_create_page` - Crear páginas |
| `jira_update_issue` - Actualizar issues | `confluence_update_page` - Actualizar páginas |
| `jira_transition_issue` - Cambiar estado | `confluence_add_comment` - Agregar comentarios |

Consulta la [Referencia de Herramientas](https://personal-1d37018d.mintlify.app/docs/tools-reference) para la lista completa.

## Métodos de Instalación

### Opción 1: uvx (Recomendado - Sin instalación permanente)

```bash
uvx mcp-atlassian --help
```

### Opción 2: Docker

```bash
docker pull ghcr.io/sooperset/mcp-atlassian:latest

docker run --rm -i \
  -e JIRA_URL=https://tu-empresa.atlassian.net \
  -e JIRA_USERNAME=tu.email@empresa.com \
  -e JIRA_API_TOKEN=tu_token_api \
  ghcr.io/sooperset/mcp-atlassian:latest
```

### Opción 3: Docker Compose (Ver guía completa arriba)

```bash
docker compose up -d
```

### Opción 4: pip

```bash
pip install mcp-atlassian
mcp-atlassian --help
```

### Opción 5: Desde el código fuente (Desarrollo)

```bash
git clone https://github.com/sooperset/mcp-atlassian.git
cd mcp-atlassian
uv sync --frozen --all-extras --dev
uv run mcp-atlassian --help
```

## Configuración Avanzada

### Variables de Entorno Disponibles

Todas las variables de entorno están documentadas en el archivo `.env.example`. Las más importantes:

**URLs de Instancia:**
- `JIRA_URL` - URL de tu instancia Jira
- `CONFLUENCE_URL` - URL de tu instancia Confluence

**Autenticación (elige un método):**
- `JIRA_USERNAME` + `JIRA_API_TOKEN` - Para Atlassian Cloud
- `JIRA_PERSONAL_TOKEN` - Para Server/Data Center
- OAuth 2.0 - Para casos avanzados (consulta documentación)

**Filtrado de Contenido:**
- `JIRA_PROJECTS_FILTER` - Limitar a proyectos específicos (ej: "PROJ,DEV,SUPPORT")
- `CONFLUENCE_SPACES_FILTER` - Limitar a espacios específicos (ej: "DEV,TEAM,DOC")

**Modo de Solo Lectura:**
- `READ_ONLY_MODE=true` - Deshabilita todas las operaciones de escritura

**Logging:**
- `MCP_VERBOSE=true` - Habilita logging nivel INFO
- `MCP_VERY_VERBOSE=true` - Habilita logging nivel DEBUG

## Seguridad

Nunca compartas tokens de API. Mantén los archivos `.env` seguros. Consulta [SECURITY.md](SECURITY.md).

## Contribuir

Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para la configuración de desarrollo.

## Licencia

MIT - Consulta [LICENSE](LICENSE). No es un producto oficial de Atlassian.

---

## Recursos Adicionales

- 📚 [Documentación oficial](https://personal-1d37018d.mintlify.app)
- 🐳 [Guía Docker Compose Windows 11](./DOCKER-WINDOWS-11-es.md)
- 🔧 [Archivo de ejemplo .env](./.env.example)
- 🐛 [Reportar problemas](https://github.com/sooperset/mcp-atlassian/issues)
- 💬 [Discusiones](https://github.com/sooperset/mcp-atlassian/discussions)
