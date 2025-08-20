# 🚀 n8n - Configuración Local y Deployment en Producción

Este proyecto proporciona una configuración completa de **n8n** con Docker Compose, incluyendo PostgreSQL, autenticación básica y soporte para webhooks tanto en desarrollo local como en producción.

## 📁 Estructura del Proyecto

```
n8n-local/
├── .env                    # Variables de entorno (NO subir al repositorio)
├── .env.example           # Plantilla de variables de entorno
├── docker-compose.yml     # Stack n8n + PostgreSQL
├── local-files/           # Archivos locales para workflows
└── README.md             # Esta documentación
```

## 🛠️ Configuración Inicial

### 1. Preparar Variables de Entorno

```bash
# Copiar la plantilla y configurar
cp .env.example .env

# Editar con tus valores
nano .env
```

### 2. Variables Críticas a Configurar

```bash
# Seguridad (generadas automáticamente)
N8N_BASIC_AUTH_USER=luca
N8N_BASIC_AUTH_PASSWORD=tVKHtXzpktEb8aPiJCeyWYfxlJpmKnM3
N8N_ENCRYPTION_KEY=5a3f3234-334f-4ce9-830c-688e104a75821984dbde-bf86-4ff3-b6ac-3b57c32c4131fe50d292-b12a-47

# Base de datos PostgreSQL
POSTGRES_PASSWORD=5c0ZXQHkwsRL37mPZKcaneBL
DB_POSTGRESDB_PASSWORD=5c0ZXQHkwsRL37mPZKcaneBL
```

### 3. Configurar PostgreSQL Local (Desarrollo)

#### Opción A: PostgreSQL Local (Recomendado para Desarrollo)

**Ventajas:**
- ✅ Más rápido que contenedor
- ✅ Acceso directo a la base de datos
- ✅ Menos uso de recursos
- ✅ Persistencia nativa

**Paso a paso:**

1. **Instalar PostgreSQL** (si no lo tienes):
   ```bash
   # macOS con Homebrew
   brew install postgresql@15
   brew services start postgresql@15
   
   # Ubuntu/Debian
   sudo apt update
   sudo apt install postgresql postgresql-contrib
   sudo systemctl start postgresql
   sudo systemctl enable postgresql
   ```

2. **Crear base de datos y usuario**:
   ```bash
   # Conectar como superusuario
   psql postgres
   
   # Crear usuario y base de datos
   CREATE USER n8n WITH PASSWORD '5c0ZXQHkwsRL37mPZKcaneBL';
   CREATE DATABASE n8n OWNER n8n;
   GRANT ALL PRIVILEGES ON DATABASE n8n TO n8n;
   \q
   ```

3. **Actualizar variables de entorno**:
   ```bash
   # En .env, cambiar:
   DB_POSTGRESDB_HOST=localhost
   DB_POSTGRESDB_PORT=5432
   DB_POSTGRESDB_DATABASE=n8n
   DB_POSTGRESDB_USER=n8n
   DB_POSTGRESDB_PASSWORD=5c0ZXQHkwsRL37mPZKcaneBL
   ```

4. **Modificar docker-compose.yml**:
   ```yaml
   # Comentar o eliminar el servicio db
   # db:
   #   image: postgres:15
   #   ...
   
   # En el servicio n8n, cambiar depends_on:
   depends_on:
     # db:
     #   condition: service_healthy
   ```

#### Opción B: PostgreSQL en Contenedor (Producción)

**Para producción, mantener el servicio `db` en docker-compose.yml**:
```yaml
# Mantener estas variables en .env para producción:
DB_POSTGRESDB_HOST=db
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=5c0ZXQHkwsRL37mPZKcaneBL
```

### 4. Estructura de Carpetas Local

El proyecto incluye una estructura organizada para archivos:

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

**Uso en workflows n8n:**
- **Leer archivos**: `/files/input/csv/datos.csv`
- **Escribir archivos**: `/files/output/reports/reporte.pdf`
- **Archivos temporales**: `/files/temp/archivo_temporal.txt`

## 🏃‍♂️ Desarrollo Local

### Iniciar el Entorno

```bash
# Verificar que PostgreSQL esté corriendo (si usas PostgreSQL local)
brew services list | grep postgresql

# Si no está corriendo:
brew services start postgresql@15

# Levantar servicios
docker-compose up -d

# Ver logs en tiempo real
docker-compose logs -f n8n
```

### Verificar Conexión a Base de Datos

```bash
# Probar conexión a PostgreSQL local
psql -h localhost -U n8n -d n8n -c "SELECT version();"

# Verificar que n8n puede conectarse
docker-compose exec n8n n8n --version
```

### Acceso y Configuración

1. **Abrir n8n**: `http://localhost:5678`
2. **Credenciales**: 
   - Usuario: `luca`
   - Contraseña: `tVKHtXzpktEb8aPiJCeyWYfxlJpmKnM3`

### 🧪 Testing de Webhooks en Local

#### Configuración del Túnel

El proyecto incluye **túnel automático** para desarrollo local:

```yaml
# En docker-compose.yml (solo para desarrollo)
command: ["start", "--tunnel"]
```

#### Crear y Probar Webhooks

1. **Crear Workflow con Webhook**:
   - Agregar nodo "Webhook" como trigger
   - Configurar método HTTP (GET, POST, etc.)
   - Guardar y activar el workflow

2. **URL del Túnel**:
   - n8n mostrará una URL como: `https://abc123.n8n.cloudhook.dev/webhook/xyz`
   - Esta URL es accesible desde internet para testing

3. **Probar Webhook**:
   ```bash
   # Ejemplo con curl
   curl -X POST https://abc123.n8n.cloudhook.dev/webhook/xyz \
     -H "Content-Type: application/json" \
     -d '{"test": "data"}'
   ```

#### ⚠️ Importante: Túnel Solo para Desarrollo

- **NO usar en producción** - El túnel es solo para testing
- **URLs temporales** - Cambian en cada reinicio
- **Sin SSL propio** - Usa certificados de n8n
- **Límites de uso** - Para desarrollo, no carga alta

## 🚀 Deployment en Producción

### 1. Preparar el Servidor

#### Requisitos Mínimos
- **CPU**: 2 cores
- **RAM**: 4GB mínimo, 8GB recomendado
- **Almacenamiento**: 20GB mínimo
- **Sistema**: Linux (Ubuntu 20.04+, Debian 11+)

#### Instalar Dependencias

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Docker y Docker Compose
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Instalar Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 2. Configurar Dominio y SSL

#### DNS
```bash
# Agregar registro A en tu proveedor DNS
automations.tudominio.com  A  TU_IP_SERVIDOR
```

#### SSL con Let's Encrypt (Nginx)
```bash
# Instalar Nginx y Certbot
sudo apt install nginx certbot python3-certbot-nginx

# Configurar Nginx
sudo nano /etc/nginx/sites-available/n8n
```

**Configuración Nginx**:
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

### 3. Configurar Variables de Producción

#### Actualizar `.env` para Producción

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

### 4. Desplegar en Producción

```bash
# Clonar proyecto en servidor
git clone https://github.com/tuusuario/n8n-local.git
cd n8n-local

# Configurar variables
cp .env.example .env
nano .env  # Editar con valores de producción

# Levantar servicios
docker-compose up -d

# Verificar estado
docker-compose ps
docker-compose logs n8n
```

### 5. Configurar Webhooks en Producción

#### URLs de Webhook
- **Testing**: `https://automations.tudominio.com/webhook-test/xyz`
- **Producción**: `https://automations.tudominio.com/webhook/xyz`

#### Migrar de Testing a Producción
1. Desactivar workflow en modo testing
2. Cambiar a URL de producción
3. Activar workflow en modo producción

## 🔄 Operaciones de Mantenimiento

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

### Migración Entre Servidores

```bash
# En servidor origen
docker-compose down
# Hacer backup completo (ver sección anterior)

# En servidor destino
git clone https://github.com/tuusuario/n8n-local.git
cd n8n-local
# Restaurar backup (ver sección anterior)
docker-compose up -d
```

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

#### Base de Datos
```bash
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=db
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=password-seguro
```

### Monitoreo y Logs

```bash
# Ver logs en tiempo real
docker-compose logs -f n8n

# Ver logs de base de datos
docker-compose logs -f db

# Estadísticas de uso
docker stats

# Espacio en disco
docker system df
```

### Troubleshooting

#### Problemas Comunes

1. **Webhooks no funcionan**:
   - Verificar `WEBHOOK_URL` en `.env`
   - Comprobar configuración de proxy
   - Revisar logs: `docker-compose logs n8n`

2. **Error de conexión a base de datos**:
   - Verificar variables `DB_POSTGRESDB_*`
   - Comprobar que PostgreSQL esté corriendo: `docker-compose ps`

3. **Problemas de SSL**:
   - Verificar certificados de Let's Encrypt
   - Comprobar configuración de Nginx
   - Revisar logs: `sudo journalctl -u nginx`

## 📚 Recursos Adicionales

### Documentación Oficial
- [Variables de Entorno](https://docs.n8n.io/hosting/configuration/environment-variables/)
- [Docker Compose](https://docs.n8n.io/hosting/installation/server-setups/docker-compose/)
- [Webhooks](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/)
- [Configuración de Proxy](https://docs.n8n.io/hosting/configuration/configuration-examples/webhook-url/)

### Comunidad
- [n8n Community](https://community.n8n.io/)
- [GitHub Issues](https://github.com/n8n-io/n8n/issues)
- [Discord](https://discord.gg/n8n)

## 🔒 Seguridad

### Mejores Prácticas
- ✅ Usar contraseñas fuertes y únicas
- ✅ Mantener `N8N_ENCRYPTION_KEY` segura
- ✅ Actualizar regularmente
- ✅ Hacer backups frecuentes
- ✅ Usar HTTPS en producción
- ✅ Configurar firewall
- ✅ Monitorear logs

### Variables Sensibles
- `N8N_ENCRYPTION_KEY`: **CRÍTICA** - No cambiar sin backup
- `N8N_BASIC_AUTH_PASSWORD`: Cambiar en cada deployment
- `POSTGRES_PASSWORD`: Usar contraseña fuerte
- `DB_POSTGRESDB_PASSWORD`: Misma que POSTGRES_PASSWORD

---

## 📝 Notas de Desarrollo

### Estructura de URLs
- **Local con túnel**: `https://abc123.n8n.cloudhook.dev/webhook/xyz`
- **Producción**: `https://automations.tudominio.com/webhook/xyz`
- **Testing**: `https://automations.tudominio.com/webhook-test/xyz`

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
```

---

**¡Listo para automatizar! 🎉**
