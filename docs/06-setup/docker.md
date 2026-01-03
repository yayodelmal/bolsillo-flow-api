# Configuración con Docker

## Overview
Guía para configurar Bolsillo Flow API usando Docker y Docker Compose, facilitando un entorno de desarrollo consistente y portable.

---

## ¿Por Qué Docker?

### Ventajas

✅ **Entorno consistente** - Mismo ambiente en todos los equipos  
✅ **Setup rápido** - Un comando y todo está listo  
✅ **Aislamiento** - No contamina tu sistema local  
✅ **Portable** - Funciona igual en macOS, Linux, Windows  
✅ **Producción-ready** - Similar al ambiente de producción

### Cuándo Usar Docker

- ✅ Nuevo en el proyecto
- ✅ Múltiples desarrolladores
- ✅ CI/CD pipelines
- ✅ Testing en ambiente limpio

### Cuándo NO Usar Docker (Desarrollo Local)

- ❌ Si prefieres velocidad de hot-reload nativa
- ❌ Si ya tienes PostgreSQL local configurado
- ❌ Si trabajas solo y no necesitas consistencia

---

## Instalación de Docker

### macOS

**Opción 1: Docker Desktop**
1. Descargar desde [docker.com](https://www.docker.com/products/docker-desktop)
2. Instalar `.dmg`
3. Abrir Docker Desktop
4. Verificar: `docker --version`

**Opción 2: Homebrew**
```bash
brew install --cask docker
```

---

### Linux (Ubuntu/Debian)

```bash
# Actualizar paquetes
sudo apt-get update

# Instalar dependencias
sudo apt-get install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release

# Agregar GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Agregar repositorio
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Instalar Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Agregar usuario a grupo docker
sudo usermod -aG docker $USER
newgrp docker

# Verificar
docker --version
docker-compose --version
```

---

### Windows

1. Habilitar WSL 2
2. Descargar Docker Desktop desde [docker.com](https://www.docker.com/products/docker-desktop)
3. Instalar y reiniciar
4. Verificar en PowerShell: `docker --version`

---

## Estructura de Archivos Docker

### Dockerfile (API)

**`Dockerfile`:**

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

WORKDIR /app

# Copiar package files
COPY package*.json ./
COPY prisma ./prisma/

# Instalar dependencias
RUN npm ci

# Copiar código fuente
COPY . .

# Generar Prisma Client
RUN npx prisma generate

# Build de la aplicación
RUN npm run build

# Stage 2: Production
FROM node:20-alpine

WORKDIR /app

# Copiar solo lo necesario del builder
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/prisma ./prisma

# Usuario no-root
USER node

# Puerto
EXPOSE 3000

# Comando de inicio
CMD ["npm", "run", "start:prod"]
```

---

### Dockerfile para Desarrollo

**`Dockerfile.dev`:**

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Instalar dependencias globales
RUN npm install -g @nestjs/cli

# Copiar package files
COPY package*.json ./

# Instalar dependencias
RUN npm install

# Copiar Prisma schema
COPY prisma ./prisma/

# Generar Prisma Client
RUN npx prisma generate

# Copiar código fuente
COPY . .

# Puerto
EXPOSE 3000

# Modo desarrollo con hot-reload
CMD ["npm", "run", "start:dev"]
```

---

### Docker Compose

**`docker-compose.yml`:**

```yaml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: bolsillo-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: bolsillo_user
      POSTGRES_PASSWORD: bolsillo_dev_password
      POSTGRES_DB: bolsillo_flow_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U bolsillo_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  # NestJS API (Development)
  api:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: bolsillo-api
    restart: unless-stopped
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://bolsillo_user:bolsillo_dev_password@postgres:5432/bolsillo_flow_dev?schema=public
      PORT: 3000
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
      - /app/dist
    depends_on:
      postgres:
        condition: service_healthy
    command: sh -c "npx prisma migrate deploy && npm run start:dev"

  # Prisma Studio (Optional)
  studio:
    image: node:20-alpine
    container_name: bolsillo-studio
    working_dir: /app
    environment:
      DATABASE_URL: postgresql://bolsillo_user:bolsillo_dev_password@postgres:5432/bolsillo_flow_dev?schema=public
    ports:
      - "5555:5555"
    volumes:
      - .:/app
    depends_on:
      - postgres
    command: sh -c "npm install && npx prisma studio"

volumes:
  postgres_data:
```

---

### Docker Compose para Producción

**`docker-compose.prod.yml`:**

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: bolsillo-postgres-prod
    restart: always
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_prod_data:/var/lib/postgresql/data
    networks:
      - bolsillo-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: bolsillo-api-prod
    restart: always
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@postgres:5432/${DB_NAME}?schema=public
      PORT: 3000
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - bolsillo-network
    command: sh -c "npx prisma migrate deploy && node dist/main"

volumes:
  postgres_prod_data:

networks:
  bolsillo-network:
    driver: bridge
```

---

### .dockerignore

**`.dockerignore`:**

```
node_modules
npm-debug.log
dist
.git
.gitignore
.env
.env.local
.env.*.local
coverage
.vscode
.idea
*.md
docker-compose*.yml
Dockerfile*
```

---

## Scripts de Inicialización

### Init Database Script

**`scripts/init-db.sql`:**

```sql
-- Crear base de datos de testing si no existe
SELECT 'CREATE DATABASE bolsillo_flow_test OWNER bolsillo_user'
WHERE NOT EXISTS (SELECT FROM pg_database WHERE datname = 'bolsillo_flow_test')\gexec

-- Dar permisos
GRANT ALL PRIVILEGES ON DATABASE bolsillo_flow_test TO bolsillo_user;
```

---

## Uso Diario

### Iniciar Ambiente Completo

```bash
# Iniciar todos los servicios
docker-compose up

# En background
docker-compose up -d

# Ver logs
docker-compose logs -f

# Solo API
docker-compose logs -f api
```

---

### Detener Servicios

```bash
# Detener todos los servicios
docker-compose down

# Detener y eliminar volúmenes (BORRA DATOS)
docker-compose down -v
```

---

### Rebuild (Después de Cambios en Dockerfile)

```bash
# Rebuild e iniciar
docker-compose up --build

# Solo rebuild
docker-compose build

# Rebuild sin cache
docker-compose build --no-cache
```

---

### Ejecutar Comandos Dentro del Contenedor

```bash
# Entrar al contenedor
docker-compose exec api sh

# Ejecutar comando específico
docker-compose exec api npm run test

# Ejecutar Prisma commands
docker-compose exec api npx prisma migrate dev
docker-compose exec api npx prisma studio
docker-compose exec api npx prisma db seed
```

---

### Ver Estado de Servicios

```bash
# Listar contenedores
docker-compose ps

# Ver logs
docker-compose logs

# Ver recursos
docker stats
```

---

## Desarrollo con Docker

### Hot Reload

**Configurado en `docker-compose.yml` con volumes:**

```yaml
volumes:
  - .:/app              # Código fuente
  - /app/node_modules   # Excluir node_modules del host
```

**Al editar archivos localmente, se reflejan en el contenedor automáticamente.**

---

### Debugging

**Agregar en `docker-compose.yml`:**

```yaml
api:
  # ... otras configs
  ports:
    - "3000:3000"
    - "9229:9229"  # Debug port
  command: sh -c "npx prisma migrate deploy && npm run start:debug"
```

**VS Code `launch.json`:**

```json
{
  "type": "node",
  "request": "attach",
  "name": "Docker: Attach",
  "port": 9229,
  "restart": true,
  "remoteRoot": "/app"
}
```

---

### Testing dentro de Docker

```bash
# Unit tests
docker-compose exec api npm run test

# E2E tests
docker-compose exec api npm run test:e2e

# Coverage
docker-compose exec api npm run test:cov
```

---

## Base de Datos en Docker

### Acceder a PostgreSQL

```bash
# Conectar a psql
docker-compose exec postgres psql -U bolsillo_user -d bolsillo_flow_dev

# Ejecutar query desde host
docker-compose exec postgres psql -U bolsillo_user -d bolsillo_flow_dev -c "SELECT * FROM categories;"
```

---

### Backup de Base de Datos

```bash
# Crear backup
docker-compose exec postgres pg_dump -U bolsillo_user bolsillo_flow_dev > backup.sql

# Restore
cat backup.sql | docker-compose exec -T postgres psql -U bolsillo_user -d bolsillo_flow_dev
```

---

### Reset de Base de Datos

```bash
# Detener servicios
docker-compose down

# Eliminar volumen de datos
docker volume rm bolsillo-flow-api_postgres_data

# Iniciar de nuevo (crea DB limpia)
docker-compose up
```

---

## Optimizaciones

### Multi-stage Build

**Ya implementado en `Dockerfile` para reducir tamaño:**

```
Stage 1 (builder): ~800MB
Stage 2 (production): ~200MB
```

---

### Cache de Layers

**Ordenar COPY para aprovechar cache:**

```dockerfile
# Copiar solo package files primero
COPY package*.json ./
RUN npm ci

# Copiar código después
COPY . .
```

**Si solo cambias código, npm ci no se ejecuta de nuevo.**

---

### .dockerignore

**Excluir archivos innecesarios para reducir tamaño del build context.**

---

## Docker Compose Profiles

**`docker-compose.yml` con profiles:**

```yaml
services:
  postgres:
    # Siempre corre
  
  api:
    # Siempre corre
  
  studio:
    profiles: ["tools"]  # Solo con --profile tools
  
  pgadmin:
    image: dpage/pgadmin4
    profiles: ["tools"]
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
```

**Uso:**
```bash
# Solo DB + API
docker-compose up

# DB + API + Tools
docker-compose --profile tools up
```

---

## Solución de Problemas

### Puerto Ya en Uso

**Error:** `Bind for 0.0.0.0:5432 failed: port is already allocated`

**Solución:**
```bash
# Cambiar puerto en docker-compose.yml
ports:
  - "5433:5432"  # Usar 5433 en host
```

---

### Volumen con Permisos Incorrectos

**Error:** `Permission denied` al escribir en `/app`

**Solución en Dockerfile:**
```dockerfile
# Crear directorio con permisos correctos
RUN mkdir -p /app && chown -R node:node /app
USER node
```

---

### Cambios No se Reflejan

**Problema:** Editas código pero no cambia en el contenedor.

**Solución:**
```bash
# Rebuild
docker-compose up --build

# O verificar volumes en docker-compose.yml
volumes:
  - .:/app  # Debe estar presente
```

---

### Container Crashea al Inicio

**Ver logs:**
```bash
docker-compose logs api
```

**Ejecutar comando manualmente:**
```bash
docker-compose run api sh
# Dentro del container:
npm run start:dev
```

---

### Base de Datos No Conecta

**Verificar health check:**
```bash
docker-compose ps
```

**Si postgres no está healthy:**
```bash
docker-compose logs postgres
```

**Verificar connection string:**
```yaml
# En docker-compose.yml, debe ser:
DATABASE_URL: postgresql://bolsillo_user:bolsillo_dev_password@postgres:5432/...
# Nota: "postgres" es el nombre del servicio, no "localhost"
```

---

## Scripts de Utilidad

### Makefile (Opcional)

**`Makefile`:**

```makefile
.PHONY: up down build logs shell migrate seed test clean

up:
	docker-compose up -d

down:
	docker-compose down

build:
	docker-compose build --no-cache

logs:
	docker-compose logs -f

shell:
	docker-compose exec api sh

migrate:
	docker-compose exec api npx prisma migrate dev

seed:
	docker-compose exec api npx prisma db seed

test:
	docker-compose exec api npm run test

clean:
	docker-compose down -v
	docker system prune -f
```

**Uso:**
```bash
make up
make logs
make migrate
```

---

### Package.json Scripts

**Agregar a `package.json`:**

```json
{
  "scripts": {
    "docker:up": "docker-compose up -d",
    "docker:down": "docker-compose down",
    "docker:build": "docker-compose build",
    "docker:logs": "docker-compose logs -f",
    "docker:migrate": "docker-compose exec api npx prisma migrate dev",
    "docker:seed": "docker-compose exec api npx prisma db seed",
    "docker:shell": "docker-compose exec api sh"
  }
}
```

---

## CI/CD con Docker

### GitHub Actions Example

**`.github/workflows/ci.yml`:**

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: bolsillo_user
          POSTGRES_PASSWORD: bolsillo_test_password
          POSTGRES_DB: bolsillo_flow_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t bolsillo-api:test -f Dockerfile.dev .
      
      - name: Run tests
        run: docker run --network host -e DATABASE_URL=postgresql://bolsillo_user:bolsillo_test_password@localhost:5432/bolsillo_flow_test bolsillo-api:test npm run test
```

---

## Referencias

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [NestJS Docker Guide](https://docs.nestjs.com/recipes/prisma#docker)
- [Setup: Development](./development-setup.md)
- [Setup: Database](./database-setup.md)