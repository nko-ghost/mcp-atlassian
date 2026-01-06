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

---

## 📖 Guía Rápida para Windows 11

**¿Nuevo en esto?** Consulta nuestra guía paso a paso completa para Windows 11:

**👉 [Guía de Instalación Windows 11 con Docker](./GUIA-WINDOWS-11.md)**

La guía incluye:
- ✅ Instalación de Docker Desktop
- ✅ Configuración paso a paso de credenciales
- ✅ Integración con Claude Desktop y Cursor
- ✅ Solución de problemas comunes
- ✅ Modo local y remoto (VPS)

---

## Inicio Rápido

### 1. Obtén tu Token de API

Ve a https://id.atlassian.com/manage-profile/security/api-tokens y crea un token.

> Para Server/Data Center, usa un Token de Acceso Personal en su lugar. Consulta [Autenticación](https://personal-1d37018d.mintlify.app/docs/authentication).

### 2. Configura tu IDE

#### Opción A: Claude Desktop

Edita tu archivo de configuración:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
- **Linux**: `~/.config/Claude/claude_desktop_config.json`

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

Abre **Settings** → **MCP** → **+ Add new global MCP server**, luego usa la misma configuración JSON.

> **Python 3.14 aún no es compatible.** Usa `["--python=3.12", "mcp-atlassian"]` como args si es necesario.

> **Usuarios de Server/Data Center**: Usa `JIRA_PERSONAL_TOKEN` en lugar de `JIRA_USERNAME` + `JIRA_API_TOKEN`. Consulta [Autenticación](https://personal-1d37018d.mintlify.app/docs/authentication) para más detalles.

### 3. Comienza a Usar

Pide a tu asistente de IA:
- **"Encuentra issues asignados a mí en el proyecto PROJ"**
- **"Busca en Confluence documentación de onboarding"**
- **"Crea un ticket de bug para el problema de login"**
- **"Actualiza el estado de PROJ-123 a Hecho"**

---

## Métodos de Instalación

### Opción 1: uvx (Recomendado - Sin instalación permanente)

```bash
# Ejecutar directamente (descarga en primer uso, cache para usos posteriores)
uvx mcp-atlassian --help

# Ejecutar con versión específica de Python (requerido para Python 3.14+)
uvx --python=3.12 mcp-atlassian --help
```

### Opción 2: Docker (Uso Local)

```bash
# Descargar la imagen
docker pull ghcr.io/sooperset/mcp-atlassian:latest

# Ejecutar con variables de entorno
docker run --rm -i \
  -e JIRA_URL=https://tu-empresa.atlassian.net \
  -e JIRA_USERNAME=tu.email@empresa.com \
  -e JIRA_API_TOKEN=tu_token_api \
  ghcr.io/sooperset/mcp-atlassian:latest
```

**Configuración en Claude/Cursor:**

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-e", "JIRA_URL",
        "-e", "JIRA_USERNAME",
        "-e", "JIRA_API_TOKEN",
        "-e", "CONFLUENCE_URL",
        "-e", "CONFLUENCE_USERNAME",
        "-e", "CONFLUENCE_API_TOKEN",
        "ghcr.io/sooperset/mcp-atlassian:latest"
      ],
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

### Opción 3: Docker Compose (Uso Local - Recomendado)

**Paso 1:** Crea un archivo `.env` en el directorio del proyecto:

```bash
# URLs de tu instancia Atlassian
JIRA_URL=https://tu-empresa.atlassian.net
CONFLUENCE_URL=https://tu-empresa.atlassian.net/wiki

# Autenticación para Atlassian Cloud
JIRA_USERNAME=tu.email@empresa.com
JIRA_API_TOKEN=tu_token_api_aqui
CONFLUENCE_USERNAME=tu.email@empresa.com
CONFLUENCE_API_TOKEN=tu_token_api_aqui

# Para Server/Data Center usa:
# JIRA_PERSONAL_TOKEN=tu_token_personal
# CONFLUENCE_PERSONAL_TOKEN=tu_token_personal
```

**Paso 2:** Configura Claude/Cursor:

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "compose",
        "-f",
        "/ruta/completa/al/proyecto/docker-compose.yml",
        "run",
        "--rm",
        "mcp-atlassian"
      ]
    }
  }
}
```

> **Windows**: Usa doble backslash en rutas: `C:\\Users\\TuUsuario\\mcp-atlassian\\docker-compose.yml`

**Consulta la [Guía de Windows 11](./GUIA-WINDOWS-11.md) para instrucciones detalladas paso a paso.**

### Opción 4: Docker Compose (Servidor HTTP - Uso Remoto/VPS)

Para exponer el servidor MCP como servicio HTTP accesible remotamente:

**Paso 1:** Configura las variables en `.env`:

```bash
# Modo de transporte HTTP
TRANSPORT=sse  # o streamable-http
PORT=9000
HOST=0.0.0.0

# Resto de configuración igual que arriba
JIRA_URL=https://tu-empresa.atlassian.net
# ... etc
```

**Paso 2:** Inicia el servidor en modo HTTP:

```bash
docker compose --profile http up -d
```

**Paso 3:** Configura Claude/Cursor con la URL:

```json
{
  "mcpServers": {
    "mcp-atlassian-http": {
      "url": "http://localhost:9000/sse"
    }
  }
}
```

> Para acceso remoto desde VPS, reemplaza `localhost` con la IP o dominio de tu servidor.
> Para `streamable-http`, usa `/mcp` en lugar de `/sse`

**Ver estado del servidor:**

```bash
docker compose --profile http ps
docker compose --profile http logs -f
```

### Opción 5: pip

```bash
pip install mcp-atlassian
mcp-atlassian --help
```

### Opción 6: uv (Gestor de paquetes)

```bash
uv add mcp-atlassian
uv run mcp-atlassian --help
```

### Opción 7: Desde el código fuente (Desarrollo)

```bash
git clone https://github.com/sooperset/mcp-atlassian.git
cd mcp-atlassian
uv sync --frozen --all-extras --dev
uv run mcp-atlassian --help
```

Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para detalles de configuración de desarrollo.

---

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

---

## Compatibilidad

| Producto | Despliegue | Soporte |
|----------|------------|---------|
| Confluence | Cloud | Totalmente soportado |
| Confluence | Server/Data Center | Soportado (v6.0+) |
| Jira | Cloud | Totalmente soportado |
| Jira | Server/Data Center | Soportado (v8.14+) |

---

## Herramientas Principales

| Jira | Confluence |
|------|------------|
| `jira_search` - Buscar con JQL | `confluence_search` - Buscar con CQL |
| `jira_get_issue` - Obtener detalles de issue | `confluence_get_page` - Obtener contenido de página |
| `jira_create_issue` - Crear issues | `confluence_create_page` - Crear páginas |
| `jira_update_issue` - Actualizar issues | `confluence_update_page` - Actualizar páginas |
| `jira_transition_issue` - Cambiar estado | `confluence_add_comment` - Agregar comentarios |

Consulta la [Referencia de Herramientas](https://personal-1d37018d.mintlify.app/docs/tools-reference) para la lista completa.

---

## Configuración Avanzada

### Variables de Entorno Principales

Todas las variables están documentadas en el archivo `.env.example`. Las más importantes:

**URLs de Instancia:**
```bash
JIRA_URL=https://tu-empresa.atlassian.net
CONFLUENCE_URL=https://tu-empresa.atlassian.net/wiki
```

**Autenticación (elige un método):**
```bash
# Método 1: API Token (Cloud - Recomendado)
JIRA_USERNAME=tu.email@empresa.com
JIRA_API_TOKEN=tu_token_api

# Método 2: Personal Access Token (Server/DC)
JIRA_PERSONAL_TOKEN=tu_token_personal

# Método 3: OAuth 2.0 (Avanzado)
# Ver documentación de autenticación
```

**Transporte:**
```bash
# Local (default)
TRANSPORT=stdio

# HTTP para acceso remoto
TRANSPORT=sse              # Server-Sent Events
# o
TRANSPORT=streamable-http  # HTTP streaming
PORT=9000
HOST=0.0.0.0
```

**Filtrado de Contenido:**
```bash
# Limitar a proyectos específicos
JIRA_PROJECTS_FILTER=PROJ,DEV,SUPPORT

# Limitar a espacios específicos
CONFLUENCE_SPACES_FILTER=DEV,TEAM,DOC
```

**Modo de Solo Lectura:**
```bash
READ_ONLY_MODE=true  # Deshabilita todas las operaciones de escritura
```

**Logging:**
```bash
MCP_VERBOSE=true        # Habilita logging nivel INFO
MCP_VERY_VERBOSE=true   # Habilita logging nivel DEBUG
```

---

## Casos de Uso

### 1. Desarrollo Local (Windows/Mac/Linux)

**Mejor opción:** uvx o Docker Compose con stdio

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": { ... }
    }
  }
}
```

### 2. Servidor Remoto / VPS

**Mejor opción:** Docker Compose con transporte HTTP

```bash
# En tu VPS
docker compose --profile http up -d
```

```json
// En tu Claude/Cursor local
{
  "mcpServers": {
    "mcp-atlassian-remote": {
      "url": "http://tu-vps.com:9000/sse"
    }
  }
}
```

### 3. Equipo Multi-usuario

**Mejor opción:** Servidor HTTP con autenticación por solicitud

Ver [documentación de transporte HTTP](https://personal-1d37018d.mintlify.app/docs/http-transport) para configuración multi-usuario.

### 4. Kubernetes / Contenedores

**Mejor opción:** Modo stateless con streamable-http

```bash
docker run -p 9000:9000 \
  --env-file .env \
  ghcr.io/sooperset/mcp-atlassian:latest \
  --transport streamable-http --stateless --port 9000
```

---

## Seguridad

- ❌ **Nunca compartas tokens de API** en repositorios públicos
- ✅ Mantén los archivos `.env` fuera de Git (agrega `.env` a `.gitignore`)
- ✅ Usa variables de entorno o gestores de secretos en producción
- ✅ Considera usar OAuth 2.0 para mayor seguridad
- ✅ Revoca tokens cuando ya no los necesites

Consulta [SECURITY.md](SECURITY.md) para más detalles.

---

## Contribuir

¡Las contribuciones son bienvenidas! Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para:
- Configuración del entorno de desarrollo
- Estándares de código
- Proceso de pull request
- Guía de testing

---

## Licencia

MIT - Consulta [LICENSE](LICENSE).

**Nota:** Este no es un producto oficial de Atlassian.

---

## Recursos Adicionales

- 📚 [Documentación Completa](https://personal-1d37018d.mintlify.app)
- 🪟 [Guía de Instalación Windows 11](./GUIA-WINDOWS-11.md)
- 🔧 [Archivo de Ejemplo .env](./.env.example)
- 🐛 [Reportar Problemas](https://github.com/sooperset/mcp-atlassian/issues)
- 💬 [Discusiones](https://github.com/sooperset/mcp-atlassian/discussions)
- 🎓 [Guía para Agentes IA](./AGENTS.md)

---

## Preguntas Frecuentes

### ¿Necesito Python instalado?

Si usas **uvx** o **Docker**, no necesitas instalar Python. Ambos métodos incluyen todo lo necesario.

### ¿Puedo usarlo con Server/Data Center?

Sí, totalmente soportado. Usa `JIRA_PERSONAL_TOKEN` en lugar de API Token. Consulta la [guía de autenticación](https://personal-1d37018d.mintlify.app/docs/authentication).

### ¿Funciona con múltiples usuarios?

Sí, usa el modo HTTP (sse o streamable-http) con autenticación por solicitud. Ver [documentación de transporte HTTP](https://personal-1d37018d.mintlify.app/docs/http-transport).

### ¿Puedo limitarlo a proyectos específicos?

Sí, usa `JIRA_PROJECTS_FILTER` y `CONFLUENCE_SPACES_FILTER` en tu archivo `.env`.

### ¿Qué diferencia hay entre stdio y HTTP?

- **stdio**: Conexión local directa, un proceso por sesión, ideal para uso personal
- **HTTP (sse/streamable-http)**: Servidor persistente, acceso remoto, multi-usuario, ideal para VPS/equipos

### ¿Es seguro almacenar tokens en .env?

El archivo `.env` debe mantenerse privado y **nunca** debe commitearse a Git. Para producción, considera usar:
- Gestores de secretos (AWS Secrets Manager, HashiCorp Vault)
- Variables de entorno del sistema
- OAuth 2.0 con refresh tokens

---

**¿Necesitas ayuda?** Abre un issue en [GitHub](https://github.com/sooperset/mcp-atlassian/issues) o consulta la [documentación oficial](https://personal-1d37018d.mintlify.app).
