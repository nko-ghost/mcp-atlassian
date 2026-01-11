# Guía Completa de Instalación - Windows 11

Guía paso a paso para instalar y configurar MCP Atlassian en Windows 11 usando Docker, e integrarlo con Claude Desktop y Cursor.

## Tabla de Contenidos

- [Requisitos Previos](#requisitos-previos)
- [Parte 1: Instalar Docker Desktop](#parte-1-instalar-docker-desktop)
- [Parte 2: Obtener Credenciales de Atlassian](#parte-2-obtener-credenciales-de-atlassian)
- [Parte 3: Configurar el Proyecto](#parte-3-configurar-el-proyecto)
- [Parte 4: Integración con Claude Desktop](#parte-4-integración-con-claude-desktop)
- [Parte 5: Integración con Cursor](#parte-5-integración-con-cursor)
- [Parte 6: Uso Remoto / VPS (Opcional)](#parte-6-uso-remoto--vps-opcional)
- [Verificar Configuración](#verificar-configuración)
- [Comandos Útiles](#comandos-útiles)
- [Solución de Problemas](#solución-de-problemas)

---

## Requisitos Previos

Antes de comenzar, asegúrate de tener:

- ✅ **Windows 11** (64-bit)
- ✅ **Cuenta de Atlassian** (Jira/Confluence Cloud, Server o Data Center)
- ✅ **Claude Desktop** y/o **Cursor** instalados
- ✅ Acceso de administrador en tu PC
- ✅ 4GB de espacio libre en disco (para Docker)

---

## Parte 1: Instalar Docker Desktop

### Paso 1.1: Descargar Docker Desktop

1. Abre tu navegador y ve a: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Haz clic en **"Download for Windows"**
3. Guarda el archivo `Docker Desktop Installer.exe`

### Paso 1.2: Instalar Docker Desktop

1. Ejecuta el instalador descargado
2. En la pantalla de configuración:
   - ✅ Marca **"Use WSL 2 instead of Hyper-V"** (recomendado)
   - ✅ Marca **"Add shortcut to desktop"** (opcional)
3. Haz clic en **"Ok"**
4. Espera a que se complete la instalación (puede tardar 5-10 minutos)
5. Cuando termine, haz clic en **"Close and restart"**
6. **Reinicia tu computadora**

### Paso 1.3: Verificar la Instalación

Después de reiniciar:

1. Abre **Docker Desktop** desde el menú de inicio
2. Acepta los términos de servicio si se te solicita
3. Espera a que Docker inicie (icono en la barra de tareas debe estar verde)
4. Abre **PowerShell** (búscalo en el menú de inicio)
5. Ejecuta estos comandos:

```powershell
docker --version
docker compose version
```

Deberías ver algo como:

```
Docker version 24.0.7, build afdd53b
Docker Compose version v2.23.3
```

✅ Si ves las versiones, Docker está instalado correctamente.

---

## Parte 2: Obtener Credenciales de Atlassian

### Para Atlassian Cloud (más común)

#### Paso 2.1: Crear Token de API

1. Ve a: [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Inicia sesión con tu cuenta de Atlassian
3. Haz clic en **"Create API token"**
4. Ingresa un nombre descriptivo: `MCP Atlassian Docker`
5. Haz clic en **"Create"**
6. **IMPORTANTE:** Haz clic en **"Copy"** inmediatamente (solo se muestra una vez)
7. Pega el token en un lugar seguro temporalmente (Notepad)

#### Paso 2.2: Anotar tus URLs

Anota las URLs de tu instancia Atlassian:

**Para Jira Cloud:**
```
https://tu-empresa.atlassian.net
```
(Reemplaza `tu-empresa` con el nombre de tu organización)

**Para Confluence Cloud:**
```
https://tu-empresa.atlassian.net/wiki
```

**Tu Email:**
```
tu.email@empresa.com
```

### Para Server/Data Center

Si usas Jira/Confluence Server o Data Center:

1. Inicia sesión en tu instancia
2. Ve a tu perfil → **"Personal Access Tokens"**
3. Crea un nuevo token con nombre descriptivo
4. Copia el token generado
5. Anota las URLs de tu servidor:
   - Jira: `https://jira.tuempresa.com`
   - Confluence: `https://confluence.tuempresa.com`

---

## Parte 3: Configurar el Proyecto

### Paso 3.1: Descargar el Proyecto

**Opción A: Con Git (Recomendado)**

Si tienes Git instalado:

```powershell
cd C:\Users\TuUsuario\Documents
git clone https://github.com/sooperset/mcp-atlassian.git
cd mcp-atlassian
```

**Opción B: Descarga Manual**

1. Ve a: [https://github.com/sooperset/mcp-atlassian](https://github.com/sooperset/mcp-atlassian)
2. Haz clic en el botón verde **"Code"**
3. Selecciona **"Download ZIP"**
4. Extrae el ZIP en: `C:\Users\TuUsuario\Documents\mcp-atlassian`
5. Abre PowerShell en esa carpeta:
   - Click derecho en la carpeta → **"Abrir en Terminal"**
   - O en PowerShell: `cd C:\Users\TuUsuario\Documents\mcp-atlassian`

### Paso 3.2: Crear Archivo .env con Credenciales

En PowerShell, dentro de la carpeta del proyecto:

```powershell
# Copiar el archivo de ejemplo
copy .env.example .env

# Abrir el archivo con Notepad
notepad .env
```

Edita el archivo `.env` con tus credenciales:

**Para Atlassian Cloud:**

```bash
# URLs de tu instancia Atlassian
JIRA_URL=https://tu-empresa.atlassian.net
CONFLUENCE_URL=https://tu-empresa.atlassian.net/wiki

# Autenticación con API Token
JIRA_USERNAME=tu.email@empresa.com
JIRA_API_TOKEN=ATATTxxxxxxxxxxxxxxxxxxxx

CONFLUENCE_USERNAME=tu.email@empresa.com
CONFLUENCE_API_TOKEN=ATATTxxxxxxxxxxxxxxxxxxxx

# Configuración básica
TRANSPORT=stdio
MCP_VERBOSE=false
```

**Para Server/Data Center:**

```bash
# URLs de tu servidor
JIRA_URL=https://jira.tuempresa.com
CONFLUENCE_URL=https://confluence.tuempresa.com

# Autenticación con Personal Access Token
JIRA_PERSONAL_TOKEN=tu_token_personal_aqui
CONFLUENCE_PERSONAL_TOKEN=tu_token_personal_aqui

# Configuración básica
TRANSPORT=stdio
MCP_VERBOSE=false
```

**IMPORTANTE:**
- Reemplaza `tu-empresa` con tu organización real
- Reemplaza `tu.email@empresa.com` con tu email
- Reemplaza los tokens con los que copiaste en el Paso 2
- Guarda el archivo: **Archivo → Guardar** (o Ctrl+S)
- Cierra Notepad

### Paso 3.3: Descargar la Imagen Docker

En PowerShell:

```powershell
docker pull ghcr.io/sooperset/mcp-atlassian:latest
```

Espera a que se descargue (puede tardar 2-5 minutos la primera vez).

✅ Cuando veas `Status: Downloaded newer image for...`, está listo.

### Paso 3.4: Probar la Configuración

Prueba que Docker puede leer tus credenciales:

```powershell
docker compose run --rm mcp-atlassian --help
```

Deberías ver el mensaje de ayuda del comando `mcp-atlassian`.

---

## Parte 4: Integración con Claude Desktop

### Paso 4.1: Ubicar el Archivo de Configuración

En PowerShell, ejecuta:

```powershell
# Abrir el archivo de configuración de Claude
notepad "$env:APPDATA\Claude\claude_desktop_config.json"
```

Si el archivo **no existe**, créalo con este contenido básico:

```json
{
  "mcpServers": {}
}
```

### Paso 4.2: Agregar MCP Atlassian

Edita el archivo para agregar la configuración de MCP Atlassian.

**Reemplaza** todo el contenido con esto (ajusta la ruta):

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "compose",
        "-f",
        "C:\\Users\\TuUsuario\\Documents\\mcp-atlassian\\docker-compose.yml",
        "run",
        "--rm",
        "mcp-atlassian"
      ]
    }
  }
}
```

**IMPORTANTE:**
- Reemplaza `C:\\Users\\TuUsuario\\Documents\\mcp-atlassian` con la ruta completa donde descargaste el proyecto
- Usa **doble backslash** (`\\`) en Windows
- Para obtener la ruta correcta, en PowerShell ejecuta: `pwd`

**Ejemplo real:**

Si descargaste en `C:\Users\Juan\Documents\mcp-atlassian`, la configuración sería:

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "compose",
        "-f",
        "C:\\Users\\Juan\\Documents\\mcp-atlassian\\docker-compose.yml",
        "run",
        "--rm",
        "mcp-atlassian"
      ]
    }
  }
}
```

Guarda el archivo (Ctrl+S) y cierra Notepad.

### Paso 4.3: Reiniciar Claude Desktop

1. Cierra completamente Claude Desktop
2. Abre **Task Manager** (Ctrl+Shift+Esc)
3. Busca cualquier proceso **"Claude"** y termínalo
4. Vuelve a abrir Claude Desktop
5. Espera unos segundos a que se conecte

### Paso 4.4: Verificar Conexión en Claude

En Claude Desktop, escribe:

> "Lista los proyectos de Jira disponibles"

o

> "¿Qué issues están asignados a mí en Jira?"

Si Claude responde con información de tu Jira, ¡está funcionando! ✅

---

## Parte 5: Integración con Cursor

### Paso 5.1: Ubicar el Archivo de Configuración

En PowerShell:

```powershell
# Abrir el archivo de configuración de Cursor
notepad "$env:APPDATA\Cursor\mcp.json"
```

Si el archivo **no existe**, créalo con:

```json
{
  "mcpServers": {}
}
```

### Paso 5.2: Agregar MCP Atlassian

Edita el archivo (igual que Claude):

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "compose",
        "-f",
        "C:\\Users\\TuUsuario\\Documents\\mcp-atlassian\\docker-compose.yml",
        "run",
        "--rm",
        "mcp-atlassian"
      ]
    }
  }
}
```

Recuerda ajustar la ruta con tu usuario y doble backslash.

Guarda (Ctrl+S) y cierra.

### Paso 5.3: Reiniciar Cursor

1. Cierra completamente Cursor
2. Vuelve a abrirlo

### Paso 5.4: Verificar Conexión en Cursor

En Cursor, abre el chat y escribe:

> "Busca en Confluence documentación sobre onboarding"

Si funciona, ¡Cursor está conectado! ✅

---

## Parte 6: Uso Remoto / VPS (Opcional)

Si quieres exponer el servidor MCP para acceso remoto o multi-usuario:

### Paso 6.1: Configurar Modo HTTP

Edita tu archivo `.env`:

```bash
# Cambiar a modo HTTP
TRANSPORT=sse  # o streamable-http
PORT=9000
HOST=0.0.0.0

# Mantener el resto de configuración igual
JIRA_URL=https://tu-empresa.atlassian.net
# ... etc
```

### Paso 6.2: Iniciar Servidor HTTP

En PowerShell:

```powershell
docker compose --profile http up -d
```

### Paso 6.3: Verificar que Está Corriendo

```powershell
docker compose --profile http ps
```

Deberías ver:

```
NAME                    STATUS
mcp-atlassian-http      Up X seconds
```

### Paso 6.4: Acceder al Servidor

**Localmente:**
- URL: `http://localhost:9000/sse`

**Remotamente (desde otro PC en tu red):**
- URL: `http://TU_IP_LOCAL:9000/sse`
- Para obtener tu IP: `ipconfig` en PowerShell

**Desde VPS:**
- Sube el proyecto a tu VPS
- Ejecuta `docker compose --profile http up -d`
- Accede desde: `http://tu-vps.com:9000/sse`

### Paso 6.5: Configurar Claude/Cursor para Servidor HTTP

En lugar de la configuración anterior, usa:

```json
{
  "mcpServers": {
    "mcp-atlassian-http": {
      "url": "http://localhost:9000/sse"
    }
  }
}
```

Para acceso remoto, reemplaza `localhost` con la IP o dominio de tu servidor.

### Paso 6.6: Ver Logs del Servidor

```powershell
docker compose --profile http logs -f
```

Presiona Ctrl+C para salir.

### Paso 6.7: Detener el Servidor HTTP

```powershell
docker compose --profile http down
```

---

## Verificar Configuración

### Test 1: Verificar Docker

```powershell
docker --version
docker compose version
```

### Test 2: Verificar Imagen Descargada

```powershell
docker images | findstr mcp-atlassian
```

Deberías ver `ghcr.io/sooperset/mcp-atlassian`

### Test 3: Probar Conexión Manual

```powershell
cd C:\Users\TuUsuario\Documents\mcp-atlassian
docker compose run --rm mcp-atlassian --help
```

### Test 4: Verificar Variables de Entorno

```powershell
# Ver que el archivo .env existe
dir .env

# Ver contenido (sin mostrar tokens)
type .env | findstr "URL"
```

---

## Comandos Útiles

### Gestión Básica

```powershell
# Ver contenedores corriendo
docker ps

# Ver todos los contenedores (incluyendo detenidos)
docker ps -a

# Ver logs de un contenedor
docker logs mcp-atlassian-local

# Detener un contenedor
docker stop mcp-atlassian-local
```

### Docker Compose - Modo Local

```powershell
# Ejecutar comando (modo stdio para Claude/Cursor)
docker compose run --rm mcp-atlassian

# Ver logs
docker compose logs

# Limpiar contenedores detenidos
docker compose down
```

### Docker Compose - Modo HTTP

```powershell
# Iniciar servidor HTTP en background
docker compose --profile http up -d

# Ver estado
docker compose --profile http ps

# Ver logs en tiempo real
docker compose --profile http logs -f

# Reiniciar servidor
docker compose --profile http restart

# Detener servidor
docker compose --profile http stop

# Detener y eliminar
docker compose --profile http down
```

### Actualización

```powershell
# Descargar última versión
docker pull ghcr.io/sooperset/mcp-atlassian:latest

# Si usas modo HTTP, reinicia:
docker compose --profile http down
docker compose --profile http up -d
```

### Limpieza

```powershell
# Limpiar contenedores, redes e imágenes no usadas
docker system prune -a

# Ver uso de espacio
docker system df
```

---

## Solución de Problemas

### Error: "Docker daemon is not running"

**Causa:** Docker Desktop no está iniciado.

**Solución:**
1. Abre **Docker Desktop** desde el menú de inicio
2. Espera a que el icono en la barra de tareas esté verde
3. Intenta de nuevo

### Error: "Cannot connect to the Docker daemon"

**Causa:** Docker no está corriendo o WSL2 no está habilitado.

**Solución:**
1. Abre PowerShell como administrador
2. Ejecuta: `wsl --status`
3. Si WSL2 no está instalado:
   ```powershell
   wsl --install
   ```
4. Reinicia tu PC
5. Abre Docker Desktop

### Error de Autenticación en Atlassian

**Mensaje:** "401 Unauthorized" o "Invalid credentials"

**Solución:**
1. Verifica que tu token API no haya expirado
2. Regenera el token en: https://id.atlassian.com/manage-profile/security/api-tokens
3. Actualiza el archivo `.env` con el nuevo token
4. Asegúrate de que no haya espacios extras en `.env`
5. Verifica que la URL sea correcta:
   - Cloud: `https://tu-empresa.atlassian.net`
   - Server/DC: `https://jira.tuempresa.com`

### Claude/Cursor No Detecta el Servidor MCP

**Solución:**
1. Verifica que la ruta en el JSON sea correcta:
   - Usa doble backslash: `C:\\Users\\...`
   - Verifica que el archivo `docker-compose.yml` exista en esa ruta
2. Verifica que el JSON sea válido:
   - Usa un validador: https://jsonlint.com/
   - Asegúrate de tener comas correctas
3. Reinicia completamente la aplicación:
   - Cierra Claude/Cursor
   - Abre Task Manager y termina cualquier proceso relacionado
   - Vuelve a abrir
4. Verifica logs de Docker:
   ```powershell
   docker compose logs
   ```

### El Contenedor Se Detiene Inmediatamente

**Solución:**
1. Revisa los logs para ver el error:
   ```powershell
   docker compose logs
   ```
2. Errores comunes:
   - **Variables faltantes en .env**: Verifica que todas las variables requeridas estén configuradas
   - **Formato incorrecto en .env**: No uses comillas alrededor de valores
   - **URL incorrecta**: Asegúrate de que la URL sea válida

### Problemas de Rendimiento

**Solución:**
1. Asigna más recursos a Docker Desktop:
   - Abre **Docker Desktop**
   - Settings → Resources
   - Aumenta **Memory** a 4GB (o más)
   - Aumenta **CPUs** a 2 (o más)
   - Haz clic en **"Apply & Restart"**

### No Se Pueden Ver Proyectos/Espacios Específicos

**Solución:**
1. Verifica los permisos de tu usuario en Atlassian
2. Si usas filtros en `.env`, verifica que sean correctos:
   ```bash
   JIRA_PROJECTS_FILTER=PROJ1,PROJ2
   CONFLUENCE_SPACES_FILTER=SPACE1,SPACE2
   ```
3. Verifica que los nombres de proyectos/espacios sean exactos (case-sensitive)

### Error: "Cannot find docker-compose.yml"

**Solución:**
1. Verifica que estés en el directorio correcto:
   ```powershell
   cd C:\Users\TuUsuario\Documents\mcp-atlassian
   pwd  # Muestra la ruta actual
   ```
2. Verifica que el archivo exista:
   ```powershell
   dir docker-compose.yml
   ```
3. Si no existe, descarga el proyecto nuevamente

### Puerto 9000 Ya Está en Uso

**Mensaje:** "Bind for 0.0.0.0:9000 failed: port is already allocated"

**Solución:**
1. Cambia el puerto en `.env`:
   ```bash
   PORT=9001  # o cualquier otro puerto disponible
   ```
2. O detén el servicio que usa el puerto 9000:
   ```powershell
   netstat -ano | findstr :9000
   # Anota el PID y luego:
   taskkill /PID <PID> /F
   ```

### Caracteres Especiales en Contraseñas

Si tu token contiene caracteres especiales (`!@#$%`):

**Solución:**
- En `.env`, NO uses comillas:
  ```bash
  # ✅ Correcto
  JIRA_API_TOKEN=abc123!@#$%xyz

  # ❌ Incorrecto
  JIRA_API_TOKEN="abc123!@#$%xyz"
  ```

---

## Configuración Avanzada

### Modo de Solo Lectura

Si solo quieres leer datos sin permitir modificaciones:

```bash
# En .env
READ_ONLY_MODE=true
```

### Filtrar por Proyectos/Espacios

```bash
# En .env
JIRA_PROJECTS_FILTER=PROYECTO1,PROYECTO2,PROYECTO3
CONFLUENCE_SPACES_FILTER=ESPACIO1,ESPACIO2
```

### Habilitar Logging Detallado

Para debugging:

```bash
# En .env
MCP_VERBOSE=true
# O para logs muy detallados:
MCP_VERY_VERBOSE=true
```

Luego revisa los logs:

```powershell
docker compose logs -f
```

### Usar Proxy Corporativo

Si estás detrás de un proxy:

```bash
# En .env
HTTP_PROXY=http://proxy.empresa.com:8080
HTTPS_PROXY=http://proxy.empresa.com:8080
```

### Modo Stateless (Kubernetes)

Para despliegues sin estado:

```bash
# En .env
TRANSPORT=streamable-http
STATELESS=true
PORT=9000
```

---

## Preguntas Frecuentes

### ¿Necesito tener Python instalado?

**No.** Docker incluye todo lo necesario. No necesitas instalar Python ni ninguna dependencia.

### ¿Puedo usar el mismo token para Jira y Confluence?

**Sí.** En Atlassian Cloud, el mismo API token funciona para ambos servicios.

### ¿Cuánto espacio ocupa Docker?

- Docker Desktop: ~2-3 GB
- Imagen de MCP Atlassian: ~200-300 MB
- Total: ~2.5-3.5 GB

### ¿Los datos de Atlassian se almacenan localmente?

**No.** MCP Atlassian actúa como un cliente/proxy. Todos los datos permanecen en tu instancia de Atlassian. Solo se almacenan tokens de autenticación localmente en el archivo `.env`.

### ¿Puedo usar esto en múltiples proyectos?

**Sí.** Una vez configurado, el mismo servidor MCP funciona para todos tus proyectos en Claude Desktop y Cursor.

### ¿Se actualiza automáticamente?

**No.** Para actualizar a la última versión:

```powershell
docker pull ghcr.io/sooperset/mcp-atlassian:latest
docker compose --profile http restart  # si usas modo HTTP
```

### ¿Puedo tener múltiples instancias (trabajo + personal)?

**Sí.** Crea carpetas separadas con diferentes archivos `.env`:

```
C:\Users\TuUsuario\mcp-atlassian-trabajo\
C:\Users\TuUsuario\mcp-atlassian-personal\
```

Luego configura diferentes servidores MCP en Claude/Cursor:

```json
{
  "mcpServers": {
    "mcp-atlassian-trabajo": {
      "command": "docker",
      "args": ["compose", "-f", "C:\\...\\mcp-atlassian-trabajo\\docker-compose.yml", "run", "--rm", "mcp-atlassian"]
    },
    "mcp-atlassian-personal": {
      "command": "docker",
      "args": ["compose", "-f", "C:\\...\\mcp-atlassian-personal\\docker-compose.yml", "run", "--rm", "mcp-atlassian"]
    }
  }
}
```

### ¿Funciona en Windows 10?

**Sí**, pero requiere WSL 2. Sigue los mismos pasos.

---

## Recursos Adicionales

- 📚 [Documentación Oficial](https://personal-1d37018d.mintlify.app)
- 📖 [README Principal](./README.md)
- 🔧 [Archivo .env de Ejemplo](./.env.example)
- 🐛 [Reportar Problemas](https://github.com/sooperset/mcp-atlassian/issues)
- 💬 [Discusiones GitHub](https://github.com/sooperset/mcp-atlassian/discussions)

---

## Licencia

MIT - Este no es un producto oficial de Atlassian.

---

**¿Necesitas ayuda adicional?**

1. Revisa la sección de [Solución de Problemas](#solución-de-problemas)
2. Consulta la [documentación oficial](https://personal-1d37018d.mintlify.app)
3. Abre un issue en [GitHub](https://github.com/sooperset/mcp-atlassian/issues)
