# Guía de Docker Compose para Windows 11

Esta guía te ayudará a configurar e integrar MCP Atlassian usando Docker Compose en Windows 11 con Claude Desktop y Cursor.

## Tabla de Contenidos

- [Requisitos Previos](#requisitos-previos)
- [Paso 1: Instalar Docker Desktop](#paso-1-instalar-docker-desktop)
- [Paso 2: Obtener Token de API de Atlassian](#paso-2-obtener-token-de-api-de-atlassian)
- [Paso 3: Configurar el Proyecto](#paso-3-configurar-el-proyecto)
- [Paso 4: Iniciar el Contenedor](#paso-4-iniciar-el-contenedor)
- [Paso 5: Integración con Claude Desktop](#paso-5-integración-con-claude-desktop)
- [Paso 6: Integración con Cursor](#paso-6-integración-con-cursor)
- [Verificar la Configuración](#verificar-la-configuración)
- [Comandos Útiles](#comandos-útiles)
- [Solución de Problemas](#solución-de-problemas)

---

## Requisitos Previos

Antes de comenzar, asegúrate de tener:

- ✅ **Windows 11** (64-bit)
- ✅ **Cuenta de Atlassian** (Cloud, Server o Data Center)
- ✅ **Claude Desktop** y/o **Cursor** instalados
- ✅ Acceso de administrador en tu PC

---

## Paso 1: Instalar Docker Desktop

### 1.1 Descargar Docker Desktop

1. Ve a [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Descarga **Docker Desktop for Windows**
3. Ejecuta el instalador descargado

### 1.2 Instalar y Configurar

1. Durante la instalación, asegúrate de que **Use WSL 2 instead of Hyper-V** esté marcado
2. Sigue las instrucciones del instalador
3. **Reinicia tu computadora** cuando se te solicite

### 1.3 Verificar la Instalación

Abre PowerShell o Command Prompt y ejecuta:

```powershell
docker --version
docker compose version
```

Deberías ver algo como:

```
Docker version 24.0.7, build afdd53b
Docker Compose version v2.23.3
```

### 1.4 Iniciar Docker Desktop

1. Abre **Docker Desktop** desde el menú de inicio
2. Espera a que el indicador en la barra de tareas cambie a verde
3. Docker está listo cuando veas "Docker Desktop is running"

---

## Paso 2: Obtener Token de API de Atlassian

### Para Atlassian Cloud

1. Ve a [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Inicia sesión con tu cuenta de Atlassian
3. Haz clic en **"Create API token"**
4. Ingresa un nombre descriptivo (ej: "MCP Atlassian Docker")
5. Haz clic en **"Create"**
6. **Copia el token inmediatamente** (solo se muestra una vez)

### Para Server/Data Center

1. Inicia sesión en tu instancia Jira/Confluence
2. Ve a tu perfil → **"Personal Access Tokens"**
3. Crea un nuevo token con permisos apropiados
4. Copia el token generado

---

## Paso 3: Configurar el Proyecto

### 3.1 Clonar o Descargar el Repositorio

**Opción A: Con Git**

```powershell
git clone https://github.com/sooperset/mcp-atlassian.git
cd mcp-atlassian
```

**Opción B: Descarga Manual**

1. Descarga el repositorio desde GitHub (botón "Code" → "Download ZIP")
2. Extrae el ZIP en una carpeta de tu elección
3. Abre PowerShell en esa carpeta

### 3.2 Crear y Configurar el Archivo .env

Copia el archivo de ejemplo y edítalo:

```powershell
copy .env.example .env
notepad .env
```

Edita el archivo `.env` con tus credenciales:

```bash
# URLs de tu instancia Atlassian
JIRA_URL=https://tu-empresa.atlassian.net
CONFLUENCE_URL=https://tu-empresa.atlassian.net/wiki

# Autenticación para Atlassian Cloud (API Token)
JIRA_USERNAME=tu.email@empresa.com
JIRA_API_TOKEN=tu_token_api_aqui

CONFLUENCE_USERNAME=tu.email@empresa.com
CONFLUENCE_API_TOKEN=tu_token_api_aqui

# Para Server/Data Center, usa en su lugar:
# JIRA_PERSONAL_TOKEN=tu_token_personal
# CONFLUENCE_PERSONAL_TOKEN=tu_token_personal

# Configuración opcional
TRANSPORT=stdio
MCP_VERBOSE=false

# Filtros opcionales (comentados por defecto)
# JIRA_PROJECTS_FILTER=PROJ,DEV,SUPPORT
# CONFLUENCE_SPACES_FILTER=DEV,TEAM,DOC

# Modo de solo lectura (desactiva escritura)
# READ_ONLY_MODE=false
```

**Importante:**
- Reemplaza `tu-empresa` con el nombre de tu organización Atlassian
- Reemplaza `tu.email@empresa.com` con tu email
- Reemplaza `tu_token_api_aqui` con el token que copiaste en el Paso 2
- Guarda el archivo (Ctrl+S) y ciérralo

---

## Paso 4: Iniciar el Contenedor

### 4.1 Descargar la Imagen Docker

En PowerShell, en la carpeta del proyecto:

```powershell
docker compose pull
```

Esto descargará la imagen más reciente de MCP Atlassian (puede tardar unos minutos la primera vez).

### 4.2 Iniciar el Contenedor

```powershell
docker compose up -d
```

El flag `-d` ejecuta el contenedor en segundo plano (detached mode).

### 4.3 Verificar que el Contenedor Está Corriendo

```powershell
docker compose ps
```

Deberías ver:

```
NAME                IMAGE                                     STATUS
mcp-atlassian       ghcr.io/sooperset/mcp-atlassian:latest    Up X seconds
```

### 4.4 Ver los Logs (Opcional)

Para ver los logs del contenedor:

```powershell
docker compose logs -f
```

Presiona `Ctrl+C` para salir de los logs.

---

## Paso 5: Integración con Claude Desktop

### 5.1 Ubicar el Archivo de Configuración

El archivo de configuración de Claude Desktop se encuentra en:

```
%APPDATA%\Claude\claude_desktop_config.json
```

Para abrirlo fácilmente:

```powershell
notepad "$env:APPDATA\Claude\claude_desktop_config.json"
```

Si el archivo no existe, créalo con el contenido a continuación.

### 5.2 Agregar la Configuración MCP con Docker

Agrega la siguiente configuración al archivo JSON:

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "compose",
        "-f",
        "C:\\ruta\\completa\\al\\proyecto\\mcp-atlassian\\docker-compose.yml",
        "run",
        "--rm",
        "mcp-atlassian"
      ]
    }
  }
}
```

**Importante:** Reemplaza `C:\\ruta\\completa\\al\\proyecto\\mcp-atlassian` con la ruta absoluta donde clonaste/descargaste el proyecto. Usa doble backslash (`\\`) en Windows.

**Ejemplo real:**

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

### 5.3 Reiniciar Claude Desktop

1. Cierra completamente Claude Desktop
2. Abre Task Manager (Ctrl+Shift+Esc)
3. Busca cualquier proceso "Claude" y termínalo
4. Vuelve a abrir Claude Desktop

### 5.4 Verificar la Conexión

En Claude Desktop, intenta hacer una pregunta como:

- "¿Qué issues están asignados a mí en Jira?"
- "Busca documentación de onboarding en Confluence"
- "Lista los proyectos de Jira disponibles"

Si todo está configurado correctamente, Claude podrá acceder a tus datos de Atlassian.

---

## Paso 6: Integración con Cursor

### 6.1 Ubicar el Archivo de Configuración

El archivo de configuración de Cursor se encuentra en:

```
%APPDATA%\Cursor\mcp.json
```

Para abrirlo:

```powershell
notepad "$env:APPDATA\Cursor\mcp.json"
```

Si el archivo no existe, créalo.

### 6.2 Agregar la Configuración MCP con Docker

Agrega la siguiente configuración al archivo JSON:

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "compose",
        "-f",
        "C:\\ruta\\completa\\al\\proyecto\\mcp-atlassian\\docker-compose.yml",
        "run",
        "--rm",
        "mcp-atlassian"
      ]
    }
  }
}
```

**Importante:** Al igual que con Claude, reemplaza la ruta con tu ruta absoluta usando doble backslash.

### 6.3 Reiniciar Cursor

1. Cierra completamente Cursor
2. Vuelve a abrirlo

### 6.4 Verificar la Conexión

En Cursor, intenta comandos similares a los de Claude Desktop para verificar que puede acceder a Jira y Confluence.

---

## Verificar la Configuración

### Test Manual con Docker

Puedes probar la conexión manualmente:

```powershell
docker compose run --rm mcp-atlassian --help
```

Esto debería mostrar la ayuda del comando `mcp-atlassian`.

### Test de Conexión a Atlassian

Para verificar que las credenciales funcionan:

```powershell
docker compose logs mcp-atlassian
```

Busca mensajes de error relacionados con autenticación.

---

## Comandos Útiles

### Gestión del Contenedor

```powershell
# Iniciar el contenedor
docker compose up -d

# Detener el contenedor
docker compose stop

# Detener y eliminar el contenedor
docker compose down

# Ver logs en tiempo real
docker compose logs -f

# Reiniciar el contenedor
docker compose restart

# Ver estado del contenedor
docker compose ps
```

### Actualizar a la Última Versión

```powershell
# Detener el contenedor
docker compose down

# Descargar la última imagen
docker compose pull

# Iniciar con la nueva versión
docker compose up -d
```

### Limpiar Recursos

```powershell
# Eliminar contenedores detenidos, redes no usadas, imágenes dangling y caché de build
docker system prune -a

# Ver uso de disco de Docker
docker system df
```

---

## Solución de Problemas

### Error: "Docker daemon is not running"

**Solución:**
1. Abre Docker Desktop desde el menú de inicio
2. Espera a que el icono en la barra de tareas esté verde
3. Intenta de nuevo el comando

### Error: "Cannot connect to the Docker daemon"

**Solución:**
- Asegúrate de que Docker Desktop esté corriendo
- Verifica que WSL 2 esté habilitado:
  ```powershell
  wsl --status
  ```
- Reinicia Docker Desktop

### Error de Autenticación en Atlassian

**Posibles causas:**
- Token de API incorrecto o expirado
- URL incorrecta (verifica que termine en `.atlassian.net` para Cloud)
- Falta el `/wiki` al final de `CONFLUENCE_URL`

**Solución:**
1. Verifica tus credenciales en el archivo `.env`
2. Regenera tu token de API si es necesario
3. Reinicia el contenedor:
   ```powershell
   docker compose restart
   ```

### Claude Desktop o Cursor no Detecta el Servidor MCP

**Solución:**
1. Verifica que la ruta en el archivo de configuración sea correcta y use doble backslash (`\\`)
2. Asegúrate de que el archivo JSON tenga sintaxis válida (usa un validador JSON online)
3. Verifica que Docker Desktop esté corriendo
4. Reinicia completamente la aplicación (cierra todos los procesos)
5. Revisa los logs:
   ```powershell
   docker compose logs -f
   ```

### El Contenedor Se Detiene Inmediatamente

**Solución:**
1. Revisa los logs para ver el error:
   ```powershell
   docker compose logs
   ```
2. Verifica que todas las variables requeridas en `.env` estén configuradas
3. Asegúrate de que no haya caracteres especiales problemáticos en tus credenciales

### Problemas de Rendimiento

**Solución:**
1. Asigna más recursos a Docker Desktop:
   - Abre Docker Desktop
   - Settings → Resources
   - Aumenta CPU y Memory
   - Haz clic en "Apply & Restart"

### No Se Pueden Ver Proyectos/Espacios Específicos

**Solución:**
1. Verifica los permisos de tu usuario en Atlassian
2. Si usas filtros, verifica que estén configurados correctamente:
   ```bash
   JIRA_PROJECTS_FILTER=PROJ1,PROJ2
   CONFLUENCE_SPACES_FILTER=SPACE1,SPACE2
   ```
3. Reinicia el contenedor después de cambiar el `.env`

### Error: "Cannot find docker-compose.yml"

**Solución:**
1. Asegúrate de estar en el directorio correcto:
   ```powershell
   cd C:\ruta\al\proyecto\mcp-atlassian
   ```
2. Verifica que el archivo `docker-compose.yml` exista en ese directorio

### Problemas con Caracteres Especiales en Contraseñas

**Solución:**
Si tu token contiene caracteres especiales, asegúrate de que el archivo `.env` no use comillas:

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

Para limitar el acceso a proyectos o espacios específicos:

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

### Usar Proxy

Si estás detrás de un proxy corporativo:

```bash
# En .env
HTTP_PROXY=http://proxy.empresa.com:8080
HTTPS_PROXY=http://proxy.empresa.com:8080
```

---

## Recursos Adicionales

- 📚 [Documentación Oficial](https://personal-1d37018d.mintlify.app)
- 📖 [README en Español](./README-es.md)
- 🔧 [Archivo .env de Ejemplo](./.env.example)
- 🐛 [Reportar Problemas](https://github.com/sooperset/mcp-atlassian/issues)
- 💬 [Discusiones GitHub](https://github.com/sooperset/mcp-atlassian/discussions)

---

## Preguntas Frecuentes

### ¿Necesito tener Python instalado?

**No.** Docker incluye todo lo necesario. No necesitas instalar Python ni ninguna dependencia.

### ¿Puedo usar esto con Server/Data Center en lugar de Cloud?

**Sí.** Solo cambia:
- La URL a tu instancia interna (ej: `https://jira.miempresa.com`)
- Usa `JIRA_PERSONAL_TOKEN` en lugar de `JIRA_USERNAME` + `JIRA_API_TOKEN`

### ¿Cuánto espacio ocupa Docker?

La imagen de MCP Atlassian ocupa aproximadamente 200-300 MB. Docker Desktop en sí requiere unos 2-3 GB.

### ¿Los datos de Atlassian se almacenan localmente?

**No.** MCP Atlassian actúa como un proxy/cliente. Todos los datos permanecen en tu instancia de Atlassian. Solo se almacenan tokens de autenticación localmente.

### ¿Puedo usar esto en múltiples proyectos?

**Sí.** Una vez configurado, puedes usar el mismo servidor MCP en todos tus proyectos de Claude Desktop y Cursor.

### ¿Se actualiza automáticamente?

**No.** Para actualizar, ejecuta:

```powershell
docker compose down
docker compose pull
docker compose up -d
```

### ¿Es seguro almacenar tokens en .env?

El archivo `.env` debe mantenerse privado y **nunca** debe commitearse a Git. Asegúrate de que `.env` esté en tu `.gitignore`.

Para mayor seguridad:
- Usa permisos restrictivos en el archivo
- Considera usar OAuth 2.0 en lugar de tokens estáticos
- Revoca tokens cuando ya no los necesites

---

## Licencia

Este proyecto está licenciado bajo MIT. No es un producto oficial de Atlassian.

---

**¿Necesitas ayuda?** Abre un issue en [GitHub](https://github.com/sooperset/mcp-atlassian/issues) o consulta la [documentación oficial](https://personal-1d37018d.mintlify.app).
