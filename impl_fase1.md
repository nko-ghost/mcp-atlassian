# 🚀 Fase 1: Deployment Básico + Integración Cursor

**Objetivo:** Desplegar MCP Atlassian en VPS y conectar con Cursor
**Duración estimada:** 30-45 minutos
**Dominio:** mcp-atlassian.tecnogato.cl
**Puerto interno:** 9889

---

## 📋 Tabla de Contenidos

1. [Confirmación de Configuración Actual](#1-confirmación-de-configuración-actual)
2. [Generar Credenciales de Atlassian](#2-generar-credenciales-de-atlassian)
3. [Preparar Estructura en VPS](#3-preparar-estructura-en-vps)
4. [Configurar Variables de Entorno](#4-configurar-variables-de-entorno)
5. [Crear Docker Compose Adaptado](#5-crear-docker-compose-adaptado)
6. [Desplegar el Servicio](#6-desplegar-el-servicio)
7. [Configurar Nginx Proxy Manager](#7-configurar-nginx-proxy-manager)
8. [Verificar Funcionamiento](#8-verificar-funcionamiento)
9. [Integrar con Cursor](#9-integrar-con-cursor)
10. [Prueba desde Vertex AI](#10-prueba-desde-vertex-ai-preview)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Confirmación de Configuración Actual

### ✅ Estado del Proyecto MCP-Atlassian

El proyecto **está listo para usar** con las siguientes características:

**Componentes verificados:**
- ✅ `.env.example` completo (157 líneas)
- ✅ `docker-compose.yml` con perfil HTTP
- ✅ `Dockerfile` optimizado (Alpine Linux, non-root user)
- ✅ Endpoint `/healthz` para health checks
- ✅ Soporte SSE (Server-Sent Events)
- ✅ Middleware de autenticación por request
- ✅ 32 herramientas Jira + 11 herramientas Confluence
- ✅ Documentación en español

**Ajustes necesarios para tu infraestructura:**
1. ⚠️ Bug en healthcheck: usa `/health` debe ser `/healthz` (lo corregiremos)
2. 🔧 Adaptar red a `infra_net` (red compartida con NPM)
3. 🔧 Cambiar puerto de 9000 a 9889
4. 🔧 Configurar para producción

---

## 2. Generar Credenciales de Atlassian

Antes de continuar, necesitas generar credenciales de autenticación para Atlassian. Elige según tu tipo de instalación:

### 📘 OPCIÓN A: Atlassian Cloud (Recomendado - Más Común)

**Características:**
- URL tipo: `https://tu-empresa.atlassian.net`
- Autenticación: API Token
- Más simple de configurar

#### Paso 1: Generar API Token para Jira

1. **Accede a tu perfil de Atlassian:**
   - Ve a: https://id.atlassian.com/manage-profile/security/api-tokens
   - Inicia sesión con tu cuenta

2. **Crea un nuevo token:**
   - Click en **"Create API token"**
   - Nombre sugerido: `MCP Atlassian Server - VPS`
   - Click en **"Create"**

3. **Copia el token inmediatamente:**
   ```
   ⚠️ IMPORTANTE: El token solo se muestra UNA vez
   Cópialo y guárdalo en un lugar seguro
   ```
   - Formato: `ATATT3xFfGF0...` (muy largo, ~200 caracteres)

4. **Guarda estos datos:**
   ```bash
   JIRA_URL=https://tu-empresa.atlassian.net
   JIRA_USERNAME=tu-email@empresa.com
   JIRA_API_TOKEN=ATATT3xFfGF0...  # El token que acabas de copiar
   ```

#### Paso 2: API Token para Confluence

**Nota:** Si usas Atlassian Cloud, **el mismo token funciona** para Jira y Confluence.

```bash
# Mismo token para ambos en Cloud
CONFLUENCE_URL=https://tu-empresa.atlassian.net/wiki
CONFLUENCE_USERNAME=tu-email@empresa.com  # Mismo email
CONFLUENCE_API_TOKEN=ATATT3xFfGF0...      # Mismo token
```

#### Paso 3: Verificar Permisos

Tu cuenta debe tener permisos para:
- ✅ Ver proyectos Jira
- ✅ Ver/crear/editar issues (si quieres modo escritura)
- ✅ Ver espacios Confluence
- ✅ Ver/crear páginas (si quieres modo escritura)

**Para verificar:**
1. Intenta acceder a Jira: https://tu-empresa.atlassian.net
2. Verifica que puedes ver proyectos y issues
3. Accede a Confluence: https://tu-empresa.atlassian.net/wiki
4. Verifica que puedes ver espacios y páginas

---

### 📗 OPCIÓN B: Atlassian Server/Data Center (On-Premise)

**Características:**
- URL tipo: `https://jira.tu-empresa.com`
- Autenticación: Personal Access Token (PAT) o Basic Auth
- Requiere acceso a servidor interno

#### Método 1: Personal Access Token (Recomendado para Server/DC)

**Jira Server/DC:**

1. **Accede a tu perfil:**
   - Ve a: `https://jira.tu-empresa.com`
   - Click en tu avatar (esquina superior derecha)
   - Selecciona **"Profile"** o **"Perfil"**

2. **Crea un PAT:**
   - Click en **"Personal Access Tokens"** (panel izquierdo)
   - Click en **"Create token"**
   - Nombre: `MCP Atlassian Server`
   - Expiration: Elige según política de seguridad (recomendado: 1 año)
   - Click en **"Create"**

3. **Copia el token:**
   ```
   ⚠️ El token solo se muestra UNA vez
   ```
   - Formato: `NjQy...` (largo, sin prefijo)

4. **Guarda estos datos:**
   ```bash
   JIRA_URL=https://jira.tu-empresa.com
   JIRA_PERSONAL_TOKEN=NjQy...  # El PAT que acabas de copiar
   # NO necesitas JIRA_USERNAME con PAT
   ```

**Confluence Server/DC:**

1. Repite el proceso en Confluence:
   - Ve a: `https://confluence.tu-empresa.com`
   - Perfil → Personal Access Tokens
   - Create token

2. Guarda:
   ```bash
   CONFLUENCE_URL=https://confluence.tu-empresa.com
   CONFLUENCE_PERSONAL_TOKEN=MjE4...  # PAT de Confluence
   ```

#### Método 2: Basic Authentication (Alternativo para Server/DC)

Si tu instalación Server/DC no soporta PAT:

```bash
JIRA_URL=https://jira.tu-empresa.com
JIRA_USERNAME=tu_usuario_jira
JIRA_API_TOKEN=tu_contraseña_jira  # Nota: API_TOKEN contiene la contraseña en Server/DC

CONFLUENCE_URL=https://confluence.tu-empresa.com
CONFLUENCE_USERNAME=tu_usuario_confluence
CONFLUENCE_API_TOKEN=tu_contraseña_confluence
```

⚠️ **Menos seguro:** Las contraseñas en texto plano son menos seguras que PAT. Usa PAT si es posible.

#### Verificaciones Adicionales para Server/DC

**SSL/TLS:**
Si tu instalación usa certificados autofirmados:
```bash
JIRA_SSL_VERIFY=false          # Solo para desarrollo/testing
CONFLUENCE_SSL_VERIFY=false
```

**mTLS (Mutual TLS):**
Si requiere certificados de cliente:
```bash
JIRA_CLIENT_CERT=/ruta/al/certificado.pem
JIRA_CLIENT_KEY=/ruta/al/key.pem
JIRA_CLIENT_KEY_PASSWORD=contraseña_del_key  # Si está encriptado
```

---

### 📝 Resumen de Credenciales Generadas

Al finalizar este paso, debes tener:

**Para Atlassian Cloud:**
```bash
JIRA_URL=https://tu-empresa.atlassian.net
JIRA_USERNAME=tu-email@empresa.com
JIRA_API_TOKEN=ATATT3xFfGF0...

CONFLUENCE_URL=https://tu-empresa.atlassian.net/wiki
CONFLUENCE_USERNAME=tu-email@empresa.com
CONFLUENCE_API_TOKEN=ATATT3xFfGF0...  # Mismo token
```

**Para Atlassian Server/DC:**
```bash
JIRA_URL=https://jira.tu-empresa.com
JIRA_PERSONAL_TOKEN=NjQy...

CONFLUENCE_URL=https://confluence.tu-empresa.com
CONFLUENCE_PERSONAL_TOKEN=MjE4...
```

✅ **Checkpoint:** Guarda estas credenciales de forma segura. Las usaremos en el paso 4.

---

## 3. Preparar Estructura en VPS

### Conectar al VPS

```bash
# Desde tu PC local
ssh usuario@tu-vps-ip

# O si usas Google Cloud SDK
gcloud compute ssh tu-instancia-nombre
```

### Crear Estructura de Carpetas

```bash
# Navegar a tu directorio de servicios
cd ~  # O donde tengas tus servicios

# Crear carpeta para MCP Atlassian
mkdir -p mcp-atlassian
cd mcp-atlassian

# Crear subcarpetas
mkdir -p logs
mkdir -p config

# Verificar estructura
tree -L 2
```

**Estructura esperada:**
```
~/mcp-atlassian/
├── docker-compose.yml    # Lo crearemos en paso 5
├── .env                  # Lo crearemos en paso 4
├── logs/                 # Logs persistentes (opcional)
└── config/               # Configuraciones adicionales (futuro)
```

### Verificar Red `infra_net`

```bash
# Verificar que la red existe
docker network ls | grep infra_net

# Debería mostrar:
# NETWORK ID     NAME        DRIVER    SCOPE
# xxxxx          infra_net   bridge    local
```

✅ Si no existe la red (error), créala:
```bash
docker network create infra_net
```

---

## 4. Configurar Variables de Entorno

### Crear archivo `.env`

```bash
cd ~/mcp-atlassian

# Crear archivo .env
nano .env
```

### Contenido del `.env` (Atlassian Cloud)

Copia y **personaliza** este template:

```bash
# =============================================
# MCP-ATLASSIAN CONFIGURATION - VPS PRODUCTION
# =============================================
# Fecha: 2026-01-21
# Dominio: mcp-atlassian.tecnogato.cl
# Puerto: 9889

# =============================================
# ATLASSIAN CLOUD - URLS
# =============================================
JIRA_URL=https://tu-empresa.atlassian.net
CONFLUENCE_URL=https://tu-empresa.atlassian.net/wiki

# =============================================
# AUTENTICACIÓN - ATLASSIAN CLOUD (API TOKEN)
# =============================================
# IMPORTANTE: Reemplaza con tus credenciales generadas en el paso 2
JIRA_USERNAME=tu-email@empresa.com
JIRA_API_TOKEN=ATATT3xFfGF0_TU_TOKEN_AQUI

CONFLUENCE_USERNAME=tu-email@empresa.com
CONFLUENCE_API_TOKEN=ATATT3xFfGF0_TU_TOKEN_AQUI

# =============================================
# SERVIDOR MCP - CONFIGURACIÓN
# =============================================
TRANSPORT=sse
PORT=9889
HOST=0.0.0.0

# =============================================
# LOGGING
# =============================================
MCP_VERBOSE=true
MCP_VERY_VERBOSE=false
MCP_LOGGING_STDOUT=true

# =============================================
# SEGURIDAD Y FILTROS
# =============================================
READ_ONLY_MODE=false

# Opcional: Limitar a proyectos/espacios específicos
#JIRA_PROJECTS_FILTER=PROJ1,PROJ2
#CONFLUENCE_SPACES_FILTER=DEV,DOCS

# =============================================
# DOCKER ESPECÍFICO
# =============================================
COMPOSE_PROJECT_NAME=mcp-atlassian
```

### Contenido del `.env` (Atlassian Server/Data Center)

Si usas Server/DC con PAT:

```bash
# =============================================
# MCP-ATLASSIAN CONFIGURATION - VPS PRODUCTION
# =============================================

# =============================================
# ATLASSIAN SERVER/DC - URLS
# =============================================
JIRA_URL=https://jira.tu-empresa.com
CONFLUENCE_URL=https://confluence.tu-empresa.com

# =============================================
# AUTENTICACIÓN - SERVER/DC (PAT)
# =============================================
JIRA_PERSONAL_TOKEN=TU_PAT_DE_JIRA_AQUI
CONFLUENCE_PERSONAL_TOKEN=TU_PAT_DE_CONFLUENCE_AQUI

# =============================================
# SSL/TLS (Server/DC)
# =============================================
JIRA_SSL_VERIFY=true
CONFLUENCE_SSL_VERIFY=true

# Si usas certificados autofirmados (solo dev/testing):
#JIRA_SSL_VERIFY=false
#CONFLUENCE_SSL_VERIFY=false

# =============================================
# SERVIDOR MCP - CONFIGURACIÓN
# =============================================
TRANSPORT=sse
PORT=9889
HOST=0.0.0.0

# =============================================
# LOGGING
# =============================================
MCP_VERBOSE=true
MCP_VERY_VERBOSE=false
MCP_LOGGING_STDOUT=true

# =============================================
# SEGURIDAD Y FILTROS
# =============================================
READ_ONLY_MODE=false

# =============================================
# DOCKER ESPECÍFICO
# =============================================
COMPOSE_PROJECT_NAME=mcp-atlassian
```

### Guardar y Proteger el Archivo

```bash
# Guardar en nano: Ctrl+O, Enter, Ctrl+X

# Proteger permisos (solo owner puede leer)
chmod 600 .env

# Verificar permisos
ls -la .env
# Debería mostrar: -rw------- 1 usuario usuario
```

✅ **Checkpoint:** Verifica que `.env` tiene tus credenciales correctas.

---

## 5. Crear Docker Compose Adaptado

### Crear `docker-compose.yml`

```bash
cd ~/mcp-atlassian
nano docker-compose.yml
```

### Contenido del `docker-compose.yml`

Copia este archivo **adaptado a tu infraestructura**:

```yaml
version: '3.8'

# ============================================================================
# MCP Atlassian - Servidor HTTP para VPS
# ============================================================================
# Dominio: mcp-atlassian.tecnogato.cl
# Puerto interno: 9889
# Red: infra_net (compartida con Nginx Proxy Manager)
# ============================================================================

services:
  mcp-atlassian-http:
    image: ghcr.io/sooperset/mcp-atlassian:latest
    container_name: mcp-atlassian-http
    restart: unless-stopped

    # Variables de entorno desde .env
    env_file:
      - .env

    environment:
      # Configuración de transporte
      - TRANSPORT=${TRANSPORT:-sse}
      - PORT=${PORT:-9889}
      - HOST=${HOST:-0.0.0.0}

      # Atlassian Cloud (API Token)
      - JIRA_URL=${JIRA_URL}
      - JIRA_USERNAME=${JIRA_USERNAME}
      - JIRA_API_TOKEN=${JIRA_API_TOKEN}
      - CONFLUENCE_URL=${CONFLUENCE_URL}
      - CONFLUENCE_USERNAME=${CONFLUENCE_USERNAME}
      - CONFLUENCE_API_TOKEN=${CONFLUENCE_API_TOKEN}

      # Atlassian Server/DC (PAT) - Descomenta si usas Server/DC
      #- JIRA_PERSONAL_TOKEN=${JIRA_PERSONAL_TOKEN}
      #- CONFLUENCE_PERSONAL_TOKEN=${CONFLUENCE_PERSONAL_TOKEN}
      #- JIRA_SSL_VERIFY=${JIRA_SSL_VERIFY:-true}
      #- CONFLUENCE_SSL_VERIFY=${CONFLUENCE_SSL_VERIFY:-true}

      # Logging
      - MCP_VERBOSE=${MCP_VERBOSE:-false}
      - MCP_VERY_VERBOSE=${MCP_VERY_VERBOSE:-false}
      - MCP_LOGGING_STDOUT=${MCP_LOGGING_STDOUT:-true}

      # Seguridad
      - READ_ONLY_MODE=${READ_ONLY_MODE:-false}
      - JIRA_PROJECTS_FILTER=${JIRA_PROJECTS_FILTER:-}
      - CONFLUENCE_SPACES_FILTER=${CONFLUENCE_SPACES_FILTER:-}

    # NO exponer puerto directamente
    # Nginx Proxy Manager manejará el acceso
    expose:
      - "9889"

    # Volumes para logs persistentes (opcional)
    volumes:
      - ./logs:/app/logs

    # Red compartida con Nginx Proxy Manager
    networks:
      - infra_net

    # Health check corregido (era /health, ahora /healthz)
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:9889/healthz"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s

    # Labels para organización
    labels:
      - "com.tecnogato.service=mcp-atlassian"
      - "com.tecnogato.domain=mcp-atlassian.tecnogato.cl"
      - "com.tecnogato.transport=sse"

# ============================================================================
# RED EXTERNA
# ============================================================================
networks:
  infra_net:
    external: true
    name: infra_net

# ============================================================================
# NOTAS DE USO
# ============================================================================
#
# DEPLOYMENT:
# -----------
# docker compose up -d
#
# LOGS:
# -----
# docker logs -f mcp-atlassian-http
#
# HEALTH CHECK:
# -------------
# docker exec mcp-atlassian-http wget -O- http://localhost:9889/healthz
#
# STOP:
# -----
# docker compose down
#
# REBUILD:
# --------
# docker compose pull && docker compose up -d
#
# ============================================================================
```

### Guardar y Validar

```bash
# Guardar: Ctrl+O, Enter, Ctrl+X

# Validar sintaxis YAML
docker compose config

# Debería mostrar la configuración parseada sin errores
```

✅ **Checkpoint:** `docker compose config` debe ejecutarse sin errores.

---

## 6. Desplegar el Servicio

### Descargar Imagen

```bash
cd ~/mcp-atlassian

# Descargar la imagen desde GitHub Container Registry
docker compose pull
```

Salida esperada:
```
[+] Pulling 1/1
 ✔ mcp-atlassian-http Pulled
```

### Iniciar Servicio

```bash
# Iniciar en modo daemon (background)
docker compose up -d
```

Salida esperada:
```
[+] Running 1/1
 ✔ Container mcp-atlassian-http  Started
```

### Verificar Estado

```bash
# Ver estado del contenedor
docker ps | grep mcp-atlassian

# Ver logs en tiempo real
docker logs -f mcp-atlassian-http
```

**Logs esperados (exitosos):**
```
INFO:mcp-atlassian.server.main:Main Atlassian MCP server lifespan starting...
INFO:mcp-atlassian.server.main:Jira configuration loaded and authentication is configured.
INFO:mcp-atlassian.server.main:Confluence configuration loaded and authentication is configured.
INFO:mcp-atlassian.server.main:Read-only mode: DISABLED
INFO:mcp-atlassian.server.main:Enabled tools filter: All tools enabled
INFO:     Started server process
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:9889 (Press CTRL+C to quit)
```

✅ **Checkpoint:** El contenedor debe estar en estado `Up` y los logs deben mostrar "Application startup complete".

---

## 7. Configurar Nginx Proxy Manager

Ahora debes configurar el proxy reverso en Nginx Proxy Manager para exponer el servicio con SSL.

### Acceder a Nginx Proxy Manager

1. Abre tu navegador
2. Ve a: `http://tu-vps-ip:81`
3. Inicia sesión con tus credenciales de NPM

### Crear Proxy Host

1. **Click en "Proxy Hosts"** (panel izquierdo)

2. **Click en "Add Proxy Host"**

3. **Tab "Details":**
   - **Domain Names:** `mcp-atlassian.tecnogato.cl`
   - **Scheme:** `http`
   - **Forward Hostname/IP:** `mcp-atlassian-http` (nombre del contenedor)
   - **Forward Port:** `9889`
   - **Cache Assets:** ❌ NO (desactivado)
   - **Block Common Exploits:** ✅ SI (activado)
   - **Websockets Support:** ✅ SI (activado) - **IMPORTANTE para SSE**
   - **Access List:** Ninguno (por ahora)

4. **Tab "SSL":**
   - **SSL Certificate:** Request a new SSL Certificate
   - **Force SSL:** ✅ SI (activado)
   - **HTTP/2 Support:** ✅ SI (activado)
   - **HSTS Enabled:** ✅ SI (activado)
   - **HSTS Subdomains:** ❌ NO
   - **Use a DNS Challenge:** ❌ NO (usa HTTP Challenge)
   - **Email Address for Let's Encrypt:** tu-email@empresa.com
   - **I Agree to the Let's Encrypt Terms of Service:** ✅ SI

5. **Click en "Save"**

### Verificación en NPM

Espera 30-60 segundos para que Let's Encrypt genere el certificado SSL.

**Indicadores de éxito:**
- 🟢 SSL status: "Valid" (icono verde de candado)
- 🟢 Certificate expiry: ~3 meses en el futuro
- 🟢 Proxy host status: Online

### Configuración DNS (si aún no está)

⚠️ **IMPORTANTE:** Asegúrate de que el DNS apunta a tu VPS:

```bash
# Verificar desde tu PC local
nslookup mcp-atlassian.tecnogato.cl

# O
dig mcp-atlassian.tecnogato.cl

# Debe resolver a tu IP del VPS
```

Si no resuelve, configura en tu proveedor DNS:
- **Type:** A
- **Name:** mcp-atlassian
- **Value:** tu-vps-ip
- **TTL:** 300 (5 minutos)

Espera 5-10 minutos para propagación DNS.

---

## 8. Verificar Funcionamiento

### Test 1: Health Check Local (desde VPS)

```bash
# Dentro del VPS
curl -f http://localhost:9889/healthz

# Respuesta esperada:
# {"status":"ok"}
```

### Test 2: Health Check desde Internet

```bash
# Desde tu PC local
curl -f https://mcp-atlassian.tecnogato.cl/healthz

# Respuesta esperada:
# {"status":"ok"}
```

### Test 3: SSE Endpoint

```bash
# Desde tu PC local
curl -N https://mcp-atlassian.tecnogato.cl/sse

# Respuesta esperada (stream que se mantiene abierto):
# (Sin errores, conexión establecida)
# Presiona Ctrl+C para salir
```

### Test 4: Logs del Contenedor

```bash
# Desde VPS
docker logs --tail 50 mcp-atlassian-http

# Buscar líneas como:
# INFO: ... GET /healthz - 200 OK
# INFO: ... GET /sse - Connection established
```

### Test 5: Verificar Autenticación con Atlassian

```bash
# Desde VPS, probar llamada a Jira API
docker exec mcp-atlassian-http sh -c '
  wget -q -O- --header="Authorization: Basic $(echo -n '$JIRA_USERNAME:$JIRA_API_TOKEN' | base64)" \
  $JIRA_URL/rest/api/3/myself
'

# Debería mostrar tu información de usuario en JSON
```

✅ **Checkpoint:** Todos los tests deben pasar exitosamente.

---

## 9. Integrar con Cursor

### Configurar Cursor en tu PC Local

1. **Abrir Cursor**

2. **Abrir configuración de MCP:**
   - **Mac/Linux:** `~/.cursor/mcp_settings.json`
   - **Windows:** `%APPDATA%\Cursor\mcp_settings.json`

3. **Editar archivo (si no existe, créalo):**

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "url": "https://mcp-atlassian.tecnogato.cl/sse",
      "name": "Atlassian (Jira + Confluence)",
      "description": "Acceso a Jira y Confluence desde Cursor"
    }
  }
}
```

4. **Guardar y reiniciar Cursor**

### Verificar Conexión en Cursor

1. **Abrir Cursor**

2. **Abrir panel de MCP:**
   - Presiona `Cmd/Ctrl + Shift + P`
   - Escribe: "MCP: Show Servers"
   - Presiona Enter

3. **Verificar estado:**
   - Deberías ver: `mcp-atlassian` con estado 🟢 **Connected**
   - Click para expandir y ver herramientas disponibles

4. **Herramientas esperadas (43 total):**
   - **Jira (32):** `jira_search`, `jira_get_issue`, `jira_create_issue`, etc.
   - **Confluence (11):** `confluence_search`, `confluence_get_page`, etc.

### Probar Funcionalidad en Cursor

1. **Abre el chat de Cursor** (Cmd/Ctrl + L)

2. **Prueba con un prompt:**
   ```
   Busca los issues abiertos en Jira asignados a mí
   ```

3. **Cursor debería:**
   - Detectar que puede usar la herramienta `jira_search`
   - Hacer el request al MCP server
   - Mostrar resultados

4. **Otra prueba:**
   ```
   Busca la página "Onboarding" en Confluence
   ```

✅ **Checkpoint:** Cursor debe mostrar las herramientas disponibles y responder a prompts.

---

## 10. Prueba desde Vertex AI (Preview)

Esta es una **prueba preliminar** para validar que el endpoint es accesible desde servicios externos. La integración completa con Vertex AI se realizará en la Fase 3.

### Método 1: Prueba con cURL

Desde tu PC local (o desde Vertex AI si ya tienes acceso):

```bash
# Test básico: Health check
curl -f https://mcp-atlassian.tecnogato.cl/healthz

# Test avanzado: SSE connection
curl -N -H "Accept: text/event-stream" \
  https://mcp-atlassian.tecnogato.cl/sse
```

### Método 2: Prueba con Python (Simulación Vertex AI)

Crea un archivo `test_mcp.py`:

```python
import requests
import json

# Configuración
MCP_URL = "https://mcp-atlassian.tecnogato.cl/sse"

# Test 1: Health check
health_url = "https://mcp-atlassian.tecnogato.cl/healthz"
response = requests.get(health_url)
print(f"Health check: {response.status_code} - {response.json()}")

# Test 2: SSE connection (básico)
print("\nIntentando conectar a SSE endpoint...")
response = requests.get(
    MCP_URL,
    headers={"Accept": "text/event-stream"},
    stream=True,
    timeout=10
)
print(f"SSE connection: {response.status_code}")
print(f"Content-Type: {response.headers.get('content-type')}")

# Test 3: Verificar que el stream está disponible
if response.status_code == 200:
    print("✅ Conexión SSE establecida correctamente")
    print("📡 Endpoint listo para integración con Vertex AI")
else:
    print(f"❌ Error: {response.status_code}")
```

Ejecutar:
```bash
python test_mcp.py
```

Salida esperada:
```
Health check: 200 - {'status': 'ok'}

Intentando conectar a SSE endpoint...
SSE connection: 200
Content-Type: text/event-stream
✅ Conexión SSE establecida correctamente
📡 Endpoint listo para integración con Vertex AI
```

✅ **Checkpoint:** Los tests deben confirmar que el endpoint es accesible externamente.

---

## 11. Troubleshooting

### Problema 1: Contenedor no inicia

**Síntoma:**
```bash
docker ps  # No muestra mcp-atlassian-http
```

**Solución:**
```bash
# Ver logs de error
docker logs mcp-atlassian-http

# Errores comunes:
# - Credenciales incorrectas en .env
# - Puerto 9889 ya en uso
# - Variables de entorno mal formateadas
```

### Problema 2: Health check falla

**Síntoma:**
```bash
curl http://localhost:9889/healthz
# curl: (7) Failed to connect
```

**Solución:**
```bash
# Verificar que el servicio escucha en el puerto
docker exec mcp-atlassian-http netstat -tuln | grep 9889

# Verificar logs
docker logs mcp-atlassian-http | grep "Uvicorn running"

# Reiniciar contenedor
docker restart mcp-atlassian-http
```

### Problema 3: Nginx Proxy Manager no conecta

**Síntoma:**
- SSL funciona, pero da 502 Bad Gateway

**Solución:**
```bash
# Verificar que ambos contenedores están en infra_net
docker network inspect infra_net

# Debe mostrar:
# - gateway (Nginx Proxy Manager)
# - mcp-atlassian-http

# Ping desde gateway a mcp
docker exec gateway ping -c 3 mcp-atlassian-http

# Si falla, verificar red:
docker network connect infra_net mcp-atlassian-http
```

### Problema 4: Cursor no conecta

**Síntoma:**
- Cursor muestra ⚠️ "Connection error" en MCP panel

**Solución:**
1. Verificar URL en `mcp_settings.json` (debe terminar en `/sse`)
2. Probar URL en navegador: `https://mcp-atlassian.tecnogato.cl/sse`
3. Revisar logs de Cursor:
   - **Mac:** `~/Library/Logs/Cursor/`
   - **Windows:** `%APPDATA%\Cursor\logs\`
4. Reiniciar Cursor completamente

### Problema 5: Autenticación con Atlassian falla

**Síntoma:**
```
ERROR: Failed to validate user Jira token
401 Unauthorized
```

**Solución:**
```bash
# Verificar variables en .env
cat .env | grep JIRA_

# Verificar que el token es correcto
# Probar manualmente con curl
curl -u "tu-email@empresa.com:tu_token" \
  https://tu-empresa.atlassian.net/rest/api/3/myself

# Si falla: regenerar token en Atlassian
```

### Problema 6: Logs muy verbosos

**Síntoma:**
- Logs del contenedor son excesivos

**Solución:**
```bash
# Editar .env
nano .env

# Cambiar:
MCP_VERBOSE=false
MCP_VERY_VERBOSE=false

# Reiniciar
docker compose down && docker compose up -d
```

### Comandos Útiles de Diagnóstico

```bash
# Estado completo del contenedor
docker inspect mcp-atlassian-http | jq '.[0].State'

# Variables de entorno del contenedor
docker exec mcp-atlassian-http env | grep -E '(JIRA|CONFLUENCE|MCP)'

# Procesos dentro del contenedor
docker exec mcp-atlassian-http ps aux

# Conectividad desde el contenedor
docker exec mcp-atlassian-http wget -O- https://tu-empresa.atlassian.net

# Test de DNS
docker exec mcp-atlassian-http nslookup tu-empresa.atlassian.net
```

---

## ✅ Checklist Final de Fase 1

Marca cada item al completarlo:

### Preparación
- [ ] Credenciales de Atlassian generadas (API Token o PAT)
- [ ] Conexión SSH al VPS establecida
- [ ] Red `infra_net` verificada

### Configuración
- [ ] Carpeta `~/mcp-atlassian` creada
- [ ] Archivo `.env` creado y protegido (chmod 600)
- [ ] Archivo `docker-compose.yml` creado
- [ ] Sintaxis YAML validada (`docker compose config`)

### Deployment
- [ ] Imagen Docker descargada (`docker compose pull`)
- [ ] Contenedor iniciado (`docker compose up -d`)
- [ ] Contenedor en estado "Up" (`docker ps`)
- [ ] Logs sin errores (autenticación OK)

### Networking
- [ ] Health check local exitoso (localhost:9889/healthz)
- [ ] Nginx Proxy Manager configurado
- [ ] DNS apunta al VPS
- [ ] SSL Let's Encrypt generado
- [ ] Health check público exitoso (https://...)

### Integración
- [ ] Cursor configurado con MCP server
- [ ] Cursor muestra estado "Connected"
- [ ] Herramientas Jira/Confluence listadas
- [ ] Prueba de funcionalidad exitosa (búsqueda en Jira)

### Validación
- [ ] Test con cURL desde PC local
- [ ] Test con Python (opcional)
- [ ] Sin errores en logs de NPM
- [ ] Sin errores en logs del contenedor

---

## 🎉 Fase 1 Completada

Si todos los checkboxes están marcados, **¡felicitaciones!**

Has desplegado exitosamente el MCP Atlassian Server en tu VPS con:
- ✅ SSL/TLS automático
- ✅ Health checks funcionando
- ✅ Cursor conectado
- ✅ Acceso seguro a Jira y Confluence

### Próximos Pasos

1. **Usa el servidor desde Cursor** durante unos días para familiarizarte
2. **Monitorea los logs** para detectar problemas potenciales
3. **Documenta casos de uso** que descubras

### Cuando estés listo para Fase 2:

➡️ **Notifícame** y crearemos el documento `impl_fase2.md` para añadir:
- Capa de API Key adicional
- Rate limiting
- Logging avanzado
- Seguridad mejorada

---

## 📞 Soporte Rápido

**Comandos de uso frecuente:**

```bash
# Ver logs en tiempo real
docker logs -f mcp-atlassian-http

# Reiniciar servicio
docker restart mcp-atlassian-http

# Ver estado
docker ps | grep mcp-atlassian

# Verificar health
curl https://mcp-atlassian.tecnogato.cl/healthz

# Parar servicio
docker compose down

# Actualizar imagen
docker compose pull && docker compose up -d
```

---

**Última actualización:** 2026-01-21
**Versión:** 1.0
**Estado:** ✅ Listo para ejecutar

🗑️ **RECUERDA:** Borra este archivo (`impl_fase1.md`) al completar todos los checkboxes.
