# 🚀 n8n - Guía Completa de Instalación y Configuración Local

Guía paso a paso para configurar y ejecutar **n8n** localmente con Docker Compose, PostgreSQL y soporte completo para webhooks.

## 📋 Tabla de Contenidos

- [📁 Estructura del Proyecto](#-estructura-del-proyecto)
- [⚡ Instalación Rápida](#-instalación-rápida)
- [🛠️ Configuración Detallada](#️-configuración-detallada)
- [🗄️ Configuración de PostgreSQL](#️-configuración-de-postgresql)
- [📂 Estructura de Archivos](#-estructura-de-archivos)
- [🚀 Primeros Pasos](#-primeros-pasos)
- [🔧 Configuración Avanzada](#-configuración-avanzada)
- [🔄 Operaciones de Mantenimiento](#-operaciones-de-mantenimiento)
- [🐛 Troubleshooting](#-troubleshooting)
- [📚 Recursos Adicionales](#-recursos-adicionales)

## 📁 Estructura del Proyecto

```
n8n-local/
├── .env                    # Variables de entorno (NO subir al repo)
├── .env.example           # Plantilla de variables de entorno
├── docker-compose.yml     # Stack n8n + PostgreSQL
├── local-files/           # Archivos para workflows
│   ├── input/             # Archivos de entrada
│   ├── output/            # Archivos generados
│   └── temp/              # Archivos temporales
└── README.md             # Esta documentación
```

## ⚡ Instalación Rápida

### Prerrequisitos

- **Docker** y **Docker Compose** instalados
- **PostgreSQL** local (recomendado) o usar contenedor
- **Git** para clonar el repositorio

### Pasos Iniciales

```bash
# 1. Clonar el repositorio
git clone <tu-repositorio>
cd n8n-local

# 2. Configurar variables de entorno
cp .env.example .env
nano .env  # Editar con tus valores

# 3. Configurar PostgreSQL local (recomendado)
# Ver sección "Configuración de PostgreSQL"

# 4. Levantar n8n
docker-compose up -d

# 5. Acceder a n8n
open http://localhost:5678
```

## 🛠️ Configuración Detallada

### 1. Variables de Entorno Críticas

**Crear archivo `.env` basado en `.env.example`:**

```bash
# Seguridad (generar valores únicos)
N8N_BASIC_AUTH_USER=luca
N8N_BASIC_AUTH_PASSWORD=tVKHtXzpktEb8aPiJCeyWYfxlJpmKnM3
N8N_ENCRYPTION_KEY=5a3f3234-334f-4ce9-830c-688e104a75821984dbde-bf86-4ff3-b6ac-3b57c32c4131fe50d292-b12a-47

# Base de datos PostgreSQL
DB_POSTGRESDB_HOST=host.docker.internal
DB_POSTGRESDB_PORT=5433
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=5c0ZXQHkwsRL37mPZKcaneBL

# Timezone
TZ=America/Argentina/Buenos_Aires
GENERIC_TIMEZONE=America/Argentina/Buenos_Aires

# Licencia (agregar después de activar)
N8N_LICENSE_ACTIVATION_KEY=5cef0c97-1bf8-4af7-9098-a20191749841
```

### 2. Generar Claves de Seguridad

```bash
# Generar contraseña segura para autenticación
openssl rand -base64 32

# Generar clave de encriptación (80 caracteres)
openssl rand -hex 40
```

## 🗄️ Configuración de PostgreSQL

### Opción A: PostgreSQL Local (Recomendado)

**Ventajas:**
- ✅ Más rápido que contenedor
- ✅ Acceso directo a la base de datos
- ✅ Menos uso de recursos
- ✅ Persistencia nativa

#### Instalación en macOS

```bash
# Instalar PostgreSQL
brew install postgresql@15
brew services start postgresql@15

# Configurar puerto personalizado (evitar conflictos)
echo "port = 5433" >> /opt/homebrew/var/postgresql@15/postgresql.conf
echo "listen_addresses = '*'" >> /opt/homebrew/var/postgresql@15/postgresql.conf

# Reiniciar PostgreSQL
brew services restart postgresql@15
```

#### Instalación en Ubuntu/Debian

```bash
# Instalar PostgreSQL
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Configurar puerto personalizado
sudo sed -i 's/#port = 5432/port = 5433/' /etc/postgresql/*/main/postgresql.conf
sudo systemctl restart postgresql
```

#### Crear Base de Datos y Usuario

```bash
# Conectar como superusuario
psql postgres

# Crear usuario y base de datos
CREATE USER n8n WITH PASSWORD '5c0ZXQHkwsRL37mPZKcaneBL';
CREATE DATABASE n8n OWNER n8n;
GRANT ALL PRIVILEGES ON DATABASE n8n TO n8n;
\q

# Verificar conexión
psql -h localhost -p 5433 -U n8n -d n8n -c "SELECT version();"
```

### Opción B: PostgreSQL en Contenedor

**Para usar PostgreSQL en contenedor (producción):**

1. **Descomentar servicio `db` en `docker-compose.yml`**
2. **Actualizar variables en `.env`:**
   ```bash
   DB_POSTGRESDB_HOST=db
   DB_POSTGRESDB_PORT=5432
   ```

## 📂 Estructura de Archivos

### Organización de `local-files`

```bash
local-files/
├── input/          # Archivos de entrada
│   ├── csv/        # Datos CSV
│   ├── pdf/        # Documentos PDF
│   ├── images/     # Imágenes
│   └── json/       # Datos JSON
├── output/         # Archivos generados
│   ├── reports/    # Reportes
│   ├── processed/  # Datos procesados
│   └── logs/       # Logs de workflows
└── temp/           # Archivos temporales
```

### Crear Estructura Base

```bash
# Crear estructura de carpetas
mkdir -p local-files/{input,output,temp}/{csv,pdf,images,json,reports,processed,logs}

# Verificar permisos
chmod 755 local-files/
```

### Uso en Workflows n8n

**Rutas en workflows:**
- **Leer archivos**: `/files/input/csv/datos.csv`
- **Escribir archivos**: `/files/output/reports/reporte.pdf`
- **Archivos temporales**: `/files/temp/archivo_temporal.txt`

**⚠️ IMPORTANTE:** n8n NO crea carpetas automáticamente. Debes crearlas antes de usar.

## 🚀 Primeros Pasos

### 1. Levantar n8n

```bash
# Verificar que PostgreSQL esté corriendo
brew services list | grep postgresql

# Levantar servicios
docker-compose up -d

# Ver logs en tiempo real
docker-compose logs -f n8n
```

### 2. Acceder y Configurar

1. **Abrir n8n**: `http://localhost:5678`
2. **Credenciales**: 
   - Usuario: `luca`
   - Contraseña: `tVKHtXzpktEb8aPiJCeyWYfxlJpmKnM3`

### 3. Crear Cuenta de Propietario

**Primera vez que accedes:**
1. Completar formulario de registro:
   - Email: `lucamazza02@gmail.com`
   - First Name: `Luca`
   - Last Name: `Mazzarello`
   - Password: Crear contraseña segura

### 4. Activar Licencia Premium

1. **Hacer clic** en "Get paid features for free (forever)"
2. **Recibir email** con clave de licencia
3. **Activar licencia** en n8n
4. **Agregar al `.env`** para persistencia:
   ```bash
   N8N_LICENSE_ACTIVATION_KEY=tu-clave-de-licencia
   ```

### 5. Configurar Organización

**Crear Tags para Organizar:**
1. **Settings** → **Tags**
2. **Crear tags principales:**
   - `secretario`
   - `agente_pauta`
   - `marketing_digital`
   - `automatizacion`
   - `desarrollo`
   - `produccion`

### 6. Testing de Webhooks

**El proyecto incluye túnel automático para desarrollo:**

```bash
# URLs de webhook generadas automáticamente
# Ejemplo: https://abc123.n8n.cloudhook.dev/webhook/xyz

# Probar webhook con curl
curl -X POST https://abc123.n8n.cloudhook.dev/webhook/xyz \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'
```

**⚠️ IMPORTANTE:** El túnel es solo para desarrollo. NO usar en producción.

## 🔧 Configuración Avanzada

### Variables de Entorno Importantes

#### Seguridad
```bash
N8N_ENCRYPTION_KEY=clave-super-secreta-80-caracteres
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=password-seguro
```

#### Ejecuciones
```bash
EXECUTIONS_MODE=regular  # regular|queue
EXECUTIONS_TIMEOUT=-1    # -1 = sin timeout
EXECUTIONS_TIMEOUT_MAX=3600  # 1h máximo
EXECUTIONS_DATA_PRUNE=true   # Limpiar historial automáticamente
EXECUTIONS_DATA_MAX_AGE=336  # 14 días
```

#### Webhooks y Túnel
```bash
# Desarrollo local (túnel automático)
command: ["start", "--tunnel"]

# URLs de webhook generadas automáticamente
# Ejemplo: https://abc123.n8n.cloudhook.dev/webhook/xyz
```

### Herramientas Visuales para PostgreSQL

#### TablePlus (Recomendado para macOS)
```bash
# Configuración de conexión:
Host: localhost
Port: 5433
User: n8n
Password: 5c0ZXQHkwsRL37mPZKcaneBL
Database: n8n
```

#### Estructura de Base de Datos n8n
```sql
-- Tablas principales
├── workflows          -- Todos los workflows
├── credentials        -- Credenciales de APIs y servicios
├── executions         -- Historial de ejecuciones
├── users              -- Usuarios del sistema
├── tags               -- Etiquetas para organizar
├── variables          -- Variables globales
└── settings           -- Configuraciones de n8n
```

## 🔄 Operaciones de Mantenimiento

### Comandos Útiles

```bash
# Reiniciar solo n8n
docker-compose restart n8n

# Ver variables de entorno
docker-compose exec n8n env | grep N8N

# Acceder a PostgreSQL
docker-compose exec db psql -U n8n -d n8n

# Limpiar volúmenes no usados
docker volume prune

# Ver logs en tiempo real
docker-compose logs -f n8n

# Verificar estado de servicios
docker-compose ps
```

### Actualizar n8n

```bash
# Actualizar imagen
docker pull docker.n8n.io/n8nio/n8n

# Reiniciar servicios
docker-compose down
docker-compose up -d

# Verificar versión
docker-compose exec n8n n8n --version
```

### Backups y Restauración

#### Backup Completo
```bash
# Crear directorio de backup
mkdir -p ~/n8n-backups/$(date +%Y%m%d)

# Backup de volúmenes Docker
docker run --rm -v n8n_local_n8n_data:/data -v ~/n8n-backups/$(date +%Y%m%d):/backup alpine tar czf /backup/n8n_data.tar.gz -C /data .
docker run --rm -v n8n_local_pg_data:/data -v ~/n8n-backups/$(date +%Y%m%d):/backup alpine tar czf /backup/pg_data.tar.gz -C /data .

# Backup de configuración
cp .env ~/n8n-backups/$(date +%Y%m%d)/
cp docker-compose.yml ~/n8n-backups/$(date +%Y%m%d)/
```

#### Restaurar Backup
```bash
# Detener servicios
docker-compose down

# Restaurar volúmenes
docker run --rm -v n8n_local_n8n_data:/data -v ~/n8n-backups/20241201:/backup alpine tar xzf /backup/n8n_data.tar.gz -C /data
docker run --rm -v n8n_local_pg_data:/data -v ~/n8n-backups/20241201:/backup alpine tar xzf /backup/pg_data.tar.gz -C /data

# Restaurar configuración
cp ~/n8n-backups/20241201/.env .
cp ~/n8n-backups/20241201/docker-compose.yml .

# Levantar servicios
docker-compose up -d
```

## 🐛 Troubleshooting

### Problemas Comunes

#### 1. n8n no inicia
```bash
# Verificar logs
docker-compose logs n8n

# Verificar conexión a base de datos
psql -h localhost -p 5433 -U n8n -d n8n -c "SELECT 1;"

# Verificar variables de entorno
docker-compose exec n8n env | grep DB_POSTGRESDB_HOST

# Reiniciar servicios
docker-compose down && docker-compose up -d
```

#### 2. Error de conexión a PostgreSQL
```bash
# Verificar que PostgreSQL esté corriendo
brew services list | grep postgresql

# Verificar puerto
lsof -i :5433

# Reiniciar PostgreSQL
brew services restart postgresql@15
```

#### 3. Webhooks no funcionan
- Verificar `WEBHOOK_URL` en `.env`
- Comprobar configuración de proxy
- Revisar logs: `docker-compose logs n8n`

#### 4. Problemas de permisos
```bash
# Verificar permisos de carpetas
ls -la local-files/
chmod 755 local-files/
```

#### 5. Error `ECONNREFUSED ::1:5433`
**Solución:** Usar `host.docker.internal` en lugar de `localhost` en `.env`

#### 6. Error de licencia perdida
```bash
# Verificar que la licencia esté en .env
grep N8N_LICENSE_ACTIVATION_KEY .env

# Si no está, agregar la clave de licencia
echo "N8N_LICENSE_ACTIVATION_KEY=tu-clave-aqui" >> .env

# Reiniciar n8n
docker-compose restart n8n
```

#### 7. Problemas de memoria/recursos
```bash
# Verificar uso de recursos
docker stats

# Limpiar recursos no usados
docker system prune -f
docker volume prune -f

# Verificar espacio en disco
df -h
```

### Verificación de Configuración

```bash
# Verificar que todos los servicios estén corriendo
docker-compose ps

# Verificar conectividad de red
docker-compose exec n8n ping host.docker.internal

# Verificar variables de entorno críticas
docker-compose exec n8n env | grep -E "(DB_POSTGRESDB|N8N_BASIC_AUTH|N8N_ENCRYPTION)"

# Verificar acceso a archivos locales
docker-compose exec n8n ls -la /files/

# Verificar versión de n8n
docker-compose exec n8n n8n --version

# Verificar logs de PostgreSQL
docker-compose logs db 2>/dev/null || echo "PostgreSQL local - verificar con: brew services list"
```

### Comandos de Diagnóstico Rápido

```bash
# Script de verificación completa
echo "=== Verificación n8n ==="
docker-compose ps
echo "=== Variables críticas ==="
docker-compose exec n8n env | grep -E "(DB_POSTGRESDB|N8N_BASIC_AUTH|N8N_ENCRYPTION)" | head -5
echo "=== Conexión PostgreSQL ==="
psql -h localhost -p 5433 -U n8n -d n8n -c "SELECT version();" 2>/dev/null || echo "Error de conexión"
echo "=== Puerto 5433 ==="
lsof -i :5433
echo "=== Recursos Docker ==="
docker stats --no-stream
```

## 🔒 Seguridad y Mejores Prácticas

### Variables Sensibles
- `N8N_ENCRYPTION_KEY`: **CRÍTICA** - No cambiar sin backup
- `N8N_BASIC_AUTH_PASSWORD`: Cambiar en cada deployment
- `POSTGRES_PASSWORD`: Usar contraseña fuerte
- `DB_POSTGRESDB_PASSWORD`: Misma que POSTGRES_PASSWORD

### Mejores Prácticas
- ✅ Usar contraseñas fuertes y únicas
- ✅ Mantener `N8N_ENCRYPTION_KEY` segura
- ✅ Actualizar regularmente
- ✅ Hacer backups frecuentes
- ✅ Usar HTTPS en producción
- ✅ Configurar firewall
- ✅ Monitorear logs

### Generar Claves Seguras
```bash
# Generar contraseña segura
openssl rand -base64 32

# Generar clave de encriptación (80 caracteres)
openssl rand -hex 40

# Generar UUID para licencia
uuidgen
```

## 📚 Recursos Adicionales

### Documentación Oficial
- [Variables de Entorno](https://docs.n8n.io/hosting/configuration/environment-variables/)
- [Docker Compose](https://docs.n8n.io/hosting/installation/server-setups/docker-compose/)
- [Webhooks](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/)
- [Configuración de Proxy](https://docs.n8n.io/hosting/configuration/configuration-examples/webhook-url/)

### Herramientas Recomendadas
- [TablePlus](https://tableplus.com/) - PostgreSQL GUI
- [DBeaver](https://dbeaver.io/) - Multiplataforma
- [pgAdmin](https://www.pgadmin.org/) - Herramienta oficial

### Comunidad
- [n8n Community](https://community.n8n.io/)
- [GitHub Issues](https://github.com/n8n-io/n8n/issues)
- [Discord](https://discord.gg/n8n)

## 🚀 Deployment en Producción

### Preparar para Producción

**Cambios necesarios en `.env`:**
```bash
# Quitar túnel (IMPORTANTE)
# command: ["start", "--tunnel"]  # ← ELIMINAR ESTA LÍNEA

# Configurar deployment
N8N_HOST=automations.tudominio.com
N8N_PROTOCOL=https
N8N_PORT=5678
WEBHOOK_URL=https://automations.tudominio.com/

# Editor y UI
N8N_EDITOR_BASE_URL=https://automations.tudominio.com
N8N_PATH=/
N8N_LISTEN_ADDRESS=::

# Seguridad adicional
N8N_PUBLIC_API_DISABLED=false
N8N_DIAGNOSTICS_ENABLED=false  # Desactivar telemetría
N8N_TEMPLATES_ENABLED=false

# Proxy (si usas Nginx)
N8N_PROXY_HOPS=1
```

### Configurar SSL con Let's Encrypt

```bash
# Instalar Nginx y Certbot
sudo apt install nginx certbot python3-certbot-nginx

# Configurar Nginx
sudo nano /etc/nginx/sites-available/n8n
```

**Configuración Nginx:**
```nginx
server {
    listen 80;
    server_name automations.tudominio.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name automations.tudominio.com;

    ssl_certificate /etc/letsencrypt/live/automations.tudominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/automations.tudominio.com/privkey.pem;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

```bash
# Habilitar sitio y obtener SSL
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
sudo certbot --nginx -d automations.tudominio.com
```

## ✅ Checklist Final

### 🔧 Configuración del Sistema
- [ ] Docker y Docker Compose instalados
- [ ] PostgreSQL local configurado en puerto 5433
- [ ] Base de datos n8n creada con usuario y permisos
- [ ] Variables de entorno configuradas en `.env`
- [ ] Docker Compose actualizado para PostgreSQL local

### 🗂️ Estructura del Proyecto
- [ ] Carpetas local-files creadas y organizadas
- [ ] Archivo .gitignore configurado
- [ ] Backup del .env realizado

### 🔐 Seguridad
- [ ] Claves de seguridad generadas y seguras
- [ ] Contraseñas cambiadas de valores por defecto
- [ ] Archivo .env NO subido al repositorio

### 🚀 Próximos Pasos
1. **Levantar n8n**: `docker-compose up -d`
2. **Acceder**: `http://localhost:5678`
3. **Crear cuenta de propietario**
4. **Activar licencia premium**
5. **Configurar tags** en n8n UI
6. **Crear primer workflow** de prueba

---

## 🎉 ¡Listo para Automatizar!

Tu entorno n8n está completamente configurado y listo para:
- ✅ **Desarrollo local** con PostgreSQL
- ✅ **Testing de webhooks** con túnel
- ✅ **Organización de múltiples proyectos**
- ✅ **Backups y mantenimiento**

**¡Comienza a crear tus workflows de automatización! 🚀**
