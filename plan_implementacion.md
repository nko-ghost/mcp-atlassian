# 📋 Plan de Implementación - MCP Atlassian en VPS

**Proyecto:** Despliegue de MCP Atlassian en VPS (GCP)
**Dominio:** mcp-atlassian.tecnogato.cl
**Infraestructura:** Docker + Nginx Proxy Manager
**Fecha:** 2026-01-21

---

## 🎯 Objetivo General

Desplegar el servidor MCP Atlassian en un VPS con:
- Acceso remoto desde Cursor
- Preparación para integración con Google Vertex AI
- Seguridad adecuada para ambiente de producción
- SSL/TLS gestionado por Nginx Proxy Manager

---

## 📊 Fases de Implementación

### ✅ **FASE 1: Deployment Básico + Integración Cursor**
**Duración estimada:** 30-45 minutos
**Documento:** `impl_fase1.md` ⬅️ **COMENZAR AQUÍ**

#### Objetivos:
- [x] Analizar configuración actual del proyecto
- [ ] Crear estructura de carpetas en VPS
- [ ] Configurar archivo `.env` con credenciales de Atlassian
- [ ] Adaptar `docker-compose.yml` a la red `infra_net`
- [ ] Desplegar servicio MCP con Docker Compose
- [ ] Configurar proxy reverso en Nginx Proxy Manager
- [ ] Verificar health checks y endpoints
- [ ] Integrar con Cursor (configuración local)
- [ ] Pruebas básicas de funcionamiento

#### Entregables:
- ✅ Servidor MCP corriendo en VPS
- ✅ Accesible vía `https://mcp-atlassian.tecnogato.cl/sse`
- ✅ Cursor conectado y funcional
- ✅ Documentación de credenciales Atlassian generadas

#### Estado: **🟢 ACTIVA** - Comenzar con este documento

---

### 🔐 **FASE 2: API Key Middleware + Seguridad**
**Duración estimada:** 45-60 minutos
**Documento:** `impl_fase2.md` (se creará al finalizar Fase 1)

#### Objetivos:
- [ ] Implementar middleware de API Key para el servidor MCP
- [ ] Añadir variable `MCP_API_KEY` a configuración
- [ ] Validar API Key en requests HTTP antes de autenticación Atlassian
- [ ] Implementar rate limiting básico por IP
- [ ] Añadir logging de accesos con identificación de cliente
- [ ] Actualizar documentación con nuevo header `X-API-Key`
- [ ] Configurar rotación de API Keys
- [ ] Pruebas de seguridad básicas

#### Entregables:
- ✅ Capa adicional de seguridad con API Key
- ✅ Logs estructurados con identificación de clientes
- ✅ Rate limiting funcional
- ✅ Documentación actualizada para Cursor y Vertex AI

#### Cambios de Código:
- Nuevo archivo: `src/mcp_atlassian/servers/middleware/api_key.py`
- Modificación: `src/mcp_atlassian/servers/main.py` (añadir middleware)
- Modificación: `.env.example` (nueva variable)
- Modificación: `docker-compose.yml` (nueva variable de entorno)

#### Estado: **⏳ PENDIENTE** - Iniciar tras completar Fase 1

---

### 🚀 **FASE 3: Producción Completa + Vertex AI**
**Duración estimada:** 2-3 horas
**Documento:** `impl_fase3.md` (se creará al finalizar Fase 2)

#### Objetivos:
- [ ] Documentación detallada para integración con Google Vertex AI
- [ ] Ejemplos de código Python para llamar al MCP desde Vertex AI
- [ ] Configuración de Function Calling en Vertex AI Agent Builder
- [ ] Implementar monitoreo avanzado (métricas, alertas)
- [ ] Añadir backup automático de configuración
- [ ] Optimizar configuración de Docker (recursos, restart policies)
- [ ] Documentar arquitectura completa del sistema
- [ ] Scripts de mantenimiento y troubleshooting
- [ ] Plan de disaster recovery

#### Entregables:
- ✅ Integración funcional con Vertex AI
- ✅ Ejemplos de código Python para Vertex AI
- ✅ Monitoreo y alertas configuradas
- ✅ Documentación completa de operaciones
- ✅ Scripts de mantenimiento

#### Componentes Nuevos:
- Documento: `docs/vertex_ai_integration.md`
- Documento: `docs/architecture.md`
- Ejemplos: `examples/vertex_ai_client.py`
- Scripts: `scripts/health_check.sh`
- Scripts: `scripts/backup_config.sh`
- Configuración: `monitoring/prometheus.yml` (opcional)

#### Estado: **⏳ PENDIENTE** - Iniciar tras completar Fase 2

---

## 📐 Arquitectura del Sistema (Estado Final)

```
┌─────────────────────────────────────────────────────────────┐
│                    INTERNET                                 │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      │ HTTPS (443)
                      │
┌─────────────────────▼───────────────────────────────────────┐
│              Nginx Proxy Manager                            │
│         (gateway container - infra_net)                     │
│                                                             │
│  - SSL/TLS Termination (Let's Encrypt)                     │
│  - Reverse Proxy                                           │
│  - mcp-atlassian.tecnogato.cl → mcp-atlassian:9889        │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      │ HTTP (9889 interno)
                      │ Network: infra_net
                      │
┌─────────────────────▼───────────────────────────────────────┐
│           MCP Atlassian Server                              │
│       (mcp-atlassian-http container)                        │
│                                                             │
│  Middlewares:                                               │
│  ┌─────────────────────────────────────────────────┐       │
│  │ 1. API Key Middleware (Fase 2)                  │       │
│  │    - Valida X-API-Key header                    │       │
│  │    - Rate limiting por IP/API Key               │       │
│  └─────────────────────────────────────────────────┘       │
│  ┌─────────────────────────────────────────────────┐       │
│  │ 2. User Token Middleware (Actual)               │       │
│  │    - Valida Authorization header                │       │
│  │    - Crea contexto por request                  │       │
│  └─────────────────────────────────────────────────┘       │
│                                                             │
│  Endpoints:                                                 │
│  - GET  /healthz        → Health check                     │
│  - GET  /sse            → SSE streaming (MCP)              │
│  - POST /mcp            → Streamable HTTP (alternativo)    │
│                                                             │
│  Transport: SSE (Server-Sent Events)                       │
│  Port: 9889 (interno)                                      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      │ HTTPS + Auth Headers
                      │
┌─────────────────────▼───────────────────────────────────────┐
│              Atlassian Cloud/Server                         │
│                                                             │
│  - Jira API (tu-empresa.atlassian.net)                     │
│  - Confluence API (tu-empresa.atlassian.net/wiki)          │
│                                                             │
│  Auth Methods:                                              │
│  - API Token (Cloud) ✅ Recomendado                        │
│  - PAT (Server/DC)                                         │
│  - OAuth 2.0 (Advanced)                                    │
└─────────────────────────────────────────────────────────────┘

Clientes:
┌─────────────────┐  ┌──────────────────┐  ┌─────────────────┐
│     Cursor      │  │  Google Vertex   │  │  Claude Desktop │
│     (Local)     │  │   AI Agents      │  │     (Local)     │
└─────────────────┘  └──────────────────┘  └─────────────────┘
        │                     │                      │
        └─────────────────────┴──────────────────────┘
                              │
         Todos conectan a: https://mcp-atlassian.tecnogato.cl/sse
         Headers requeridos:
         - X-API-Key: <api_key_del_servidor>     [Fase 2]
         - Authorization: Bearer <atlassian_token>
         - X-Atlassian-Cloud-Id: <cloud_id>       [Opcional]
```

---

## 🔧 Diferencias entre Transportes MCP

### **SSE (Server-Sent Events)** ⭐ **Recomendado para tu caso**

**Características:**
- Conexión HTTP persistente (long-lived)
- Servidor envía eventos al cliente en tiempo real
- Protocolo unidireccional (servidor → cliente)
- Reconexión automática en caso de desconexión
- Ideal para streaming de respuestas largas

**Ventajas:**
- ✅ Mejor rendimiento para interacciones continuas (Cursor)
- ✅ Menor latencia en respuestas sucesivas
- ✅ Manejo automático de reconexión
- ✅ Estándar web nativo (EventSource API)
- ✅ Compatible con MCP oficial de Anthropic

**Desventajas:**
- ⚠️ Requiere conexión persistente (más recursos de red)
- ⚠️ No es stateless (cada cliente mantiene sesión)

**Endpoint:** `https://mcp-atlassian.tecnogato.cl/sse`

**Caso de uso ideal:**
- Cursor (uso interactivo continuo)
- Claude Desktop (sesiones largas)
- Desarrollo y debugging

---

### **Streamable HTTP (Stateless)**

**Características:**
- Cada request es independiente (stateless)
- HTTP POST estándar
- Sin conexiones persistentes
- Diseñado para escalabilidad horizontal

**Ventajas:**
- ✅ Escalable horizontalmente (load balancing fácil)
- ✅ Compatible con Kubernetes/Cloud Run
- ✅ Sin estado = más fácil de escalar
- ✅ Mejor para sistemas distribuidos

**Desventajas:**
- ⚠️ Mayor latencia por request (handshake cada vez)
- ⚠️ No mantiene contexto entre requests
- ⚠️ Más overhead de red

**Endpoint:** `https://mcp-atlassian.tecnogato.cl/mcp`

**Caso de uso ideal:**
- Google Vertex AI (requests ocasionales)
- APIs serverless
- Integraciones batch
- Sistemas multi-tenant a gran escala

---

### **¿Por qué elegimos SSE para Fase 1?**

Para tu caso de uso (Cursor + eventualmente Vertex AI):

1. **Cursor necesita SSE:**
   - Interacciones frecuentes y continuas
   - Mejor experiencia de usuario con conexión persistente
   - Cursor espera SSE por defecto

2. **Vertex AI puede usar ambos:**
   - En Fase 3, puedes habilitar ambos transportes simultáneamente
   - Vertex AI puede hacer requests HTTP estándar a `/sse` sin problema
   - O configurar para usar `/mcp` si prefieres stateless

3. **Flexibilidad:**
   - El servidor puede exponer **ambos endpoints simultáneamente**
   - Cursor usa `/sse`
   - Vertex AI usa `/mcp` (stateless)
   - Solo necesitas cambiar la URL del endpoint

**Configuración dual (Fase 3 opcional):**
```yaml
# Exponer ambos transportes
environment:
  - TRANSPORT=sse  # Default para /sse
  # /mcp también estará disponible automáticamente
```

---

## 🔐 Variables de Entorno por Fase

### **Fase 1: Variables Básicas**
```bash
# Atlassian URLs
JIRA_URL=https://tu-empresa.atlassian.net
CONFLUENCE_URL=https://tu-empresa.atlassian.net/wiki

# Autenticación (una de las opciones)
JIRA_USERNAME=tu-email@empresa.com
JIRA_API_TOKEN=tu_api_token
CONFLUENCE_USERNAME=tu-email@empresa.com
CONFLUENCE_API_TOKEN=tu_api_token

# Server Config
TRANSPORT=sse
PORT=9889
HOST=0.0.0.0

# Logging
MCP_VERBOSE=true
```

### **Fase 2: Variables de Seguridad (adicionales)**
```bash
# API Key del servidor MCP
MCP_API_KEY=tu_api_key_secreta_generada

# Rate Limiting
MCP_RATE_LIMIT_PER_MINUTE=60
MCP_RATE_LIMIT_PER_HOUR=1000

# Logging avanzado
MCP_VERY_VERBOSE=true
MCP_LOG_ACCESS=true
```

### **Fase 3: Variables de Producción (adicionales)**
```bash
# Monitoring
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT=9090

# Backup
BACKUP_ENABLED=true
BACKUP_SCHEDULE="0 2 * * *"  # Daily at 2 AM

# Multi-tenant (opcional)
MULTI_TENANT_MODE=true
```

---

## 📝 Checklist de Progreso

### Fase 1: Deployment Básico
- [x] Análisis de configuración actual
- [ ] Estructura de carpetas creada
- [ ] `.env` configurado
- [ ] `docker-compose.yml` adaptado
- [ ] Servicio desplegado y corriendo
- [ ] Nginx Proxy Manager configurado
- [ ] Health checks funcionando
- [ ] Cursor conectado
- [ ] Pruebas básicas exitosas
- [ ] Documentación completada

### Fase 2: Seguridad (pendiente)
- [ ] Middleware API Key implementado
- [ ] Rate limiting configurado
- [ ] Logging avanzado
- [ ] Rotación de keys documentada
- [ ] Pruebas de seguridad
- [ ] Documentación actualizada

### Fase 3: Producción (pendiente)
- [ ] Vertex AI integrado
- [ ] Ejemplos Python creados
- [ ] Monitoreo configurado
- [ ] Backup automatizado
- [ ] Scripts de mantenimiento
- [ ] Documentación completa

---

## 🗑️ Limpieza de Documentos

**Instrucción:** Al completar cada fase, **BORRAR** el documento `impl_faseX.md` correspondiente.

- ✅ Fase 1 completada → Borrar `impl_fase1.md`
- ✅ Fase 2 completada → Borrar `impl_fase2.md`
- ✅ Fase 3 completada → Borrar `impl_fase3.md`

Este documento maestro (`plan_implementacion.md`) se mantiene hasta el final del proyecto.

---

## 📞 Soporte y Troubleshooting

### Health Check Manual
```bash
# Verificar que el servidor responde
curl -f http://localhost:9889/healthz

# Verificar SSE endpoint
curl -N http://localhost:9889/sse
```

### Logs del Contenedor
```bash
# Ver logs en tiempo real
docker logs -f mcp-atlassian-http

# Ver últimas 100 líneas
docker logs --tail 100 mcp-atlassian-http
```

### Verificar Conectividad desde Cursor
```bash
# Probar desde tu PC local
curl -N https://mcp-atlassian.tecnogato.cl/sse
```

---

## 🎯 Próximo Paso

➡️ **Abrir y seguir: `impl_fase1.md`**

---

**Última actualización:** 2026-01-21
**Versión del plan:** 1.0
**Estado:** Fase 1 en progreso
