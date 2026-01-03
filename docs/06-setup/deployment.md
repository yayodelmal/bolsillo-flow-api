# Configuración del Entorno de Desarrollo

## Overview
Esta guía te llevará paso a paso por la configuración completa del entorno de desarrollo para Bolsillo Flow API.

---

## Requisitos Previos

### Software Requerido

| Software | Versión Mínima | Versión Recomendada | Verificación |
|----------|----------------|---------------------|--------------|
| Node.js | 20.x | 20.19.0 | `node --version` |
| npm | 10.x | 10.2.4 | `npm --version` |
| PostgreSQL | 15.x | 15.10 | `psql --version` |
| Git | 2.x | Latest | `git --version` |

### Editores Recomendados

- **VS Code** (recomendado)
  - Extensiones sugeridas (ver `.vscode/extensions.json`)
- **WebStorm**
- **Cursor**

---

## Instalación de Dependencias Base

### 1. Instalar Node.js

#### macOS (usando Homebrew)
```bash
brew install node@20
```

#### Linux (Ubuntu/Debian)
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

#### Windows
Descargar instalador desde [nodejs.org](https://nodejs.org/)

**Verificar instalación:**
```bash
node --version
# Esperado: v20.19.0 o superior

npm --version
# Esperado: 10.2.4 o superior
```

---

### 2. Instalar PostgreSQL

#### macOS (usando Homebrew)
```bash
brew install postgresql@15
brew services start postgresql@15
```

#### Linux (Ubuntu/Debian)
```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install postgresql-15
```

#### Windows
Descargar instalador desde [postgresql.org](https://www.postgresql.org/download/windows/)

**Verificar instalación:**
```bash
psql --version
# Esperado: psql (PostgreSQL) 15.x
```

---

### 3. Configurar PostgreSQL

#### Crear Usuario y Base de Datos

**Conectar a PostgreSQL:**
```bash
psql postgres
```

**Crear usuario:**
```sql
CREATE USER bolsillo_user WITH PASSWORD 'bolsillo_dev_password';
```

**Crear base de datos de desarrollo:**
```sql
CREATE DATABASE bolsillo_flow_dev OWNER bolsillo_user;
```

**Crear base de datos de testing:**
```sql
CREATE DATABASE bolsillo_flow_test OWNER bolsillo_user;
```

**Dar permisos:**
```sql
GRANT ALL PRIVILEGES ON DATABASE bolsillo_flow_dev TO bolsillo_user;
GRANT ALL PRIVILEGES ON DATABASE bolsillo_flow_test TO bolsillo_user;
```

**Salir:**
```sql
\q
```

**Verificar conexión:**
```bash
psql -U bolsillo_user -d bolsillo_flow_dev -h localhost
# Ingresar password cuando se solicite

# Si conecta exitosamente:
\l  # Listar bases de datos
\q  # Salir
```

---

## Configuración del Proyecto

### 1. Clonar el Repositorio

```bash
git clone https://github.com/tu-usuario/bolsillo-flow-api.git
cd bolsillo-flow-api
```

---

### 2. Instalar Dependencias del Proyecto

```bash
npm install
```

**Esto instalará:**
- NestJS framework
- Prisma ORM
- TypeScript
- Validation libraries
- Testing tools
- Todas las dependencias listadas en `package.json`

**Verificar instalación:**
```bash
npm list --depth=0
```

---

### 3. Configurar Variables de Entorno

**Crear archivo `.env` en la raíz del proyecto:**

```bash
cp .env.example .env
```

**Editar `.env` con tus valores:**

```env
# Server Configuration
NODE_ENV=development
PORT=3000

# Database Configuration
DATABASE_URL="postgresql://bolsillo_user:bolsillo_dev_password@localhost:5432/bolsillo_flow_dev?schema=public"

# Testing Database
DATABASE_URL_TEST="postgresql://bolsillo_user:bolsillo_dev_password@localhost:5432/bolsillo_flow_test?schema=public"

# JWT Configuration (futuro)
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
JWT_EXPIRATION=7d

# CORS Configuration
CORS_ORIGIN=http://localhost:3000,http://localhost:3001

# Logging
LOG_LEVEL=debug

# API Configuration
API_PREFIX=/api
```

**Importante:**
- **Nunca** commitear el archivo `.env` al repositorio
- Cambiar `JWT_SECRET` en producción
- Las contraseñas de desarrollo son solo para desarrollo local

---

### 4. Configurar Prisma

#### Generar Prisma Client

```bash
npx prisma generate
```

**Esto genera:**
- Cliente de Prisma tipado según tu schema
- Tipos TypeScript para tus modelos

---

#### Crear y Aplicar Migraciones

**Crear migración inicial:**
```bash
npx prisma migrate dev --name init
```

**Esto hace:**
1. Crea carpeta `prisma/migrations/`
2. Genera SQL para crear tablas
3. Aplica migración a la base de datos
4. Regenera Prisma Client

**Output esperado:**
```
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db": PostgreSQL database "bolsillo_flow_dev" at "localhost:5432"

✔ Generated Prisma Client to ./node_modules/.prisma/client

The following migration(s) have been created and applied:

migrations/
  └─ 20250102000000_init/
    └─ migration.sql

Your database is now in sync with your schema.
```

---

#### Verificar Tablas Creadas

```bash
psql -U bolsillo_user -d bolsillo_flow_dev -h localhost
```

```sql
-- Listar tablas
\dt

-- Describir una tabla
\d categories

-- Salir
\q
```

**Tablas esperadas:**
- categories
- budgets
- expense_types
- expenses
- credit_cards
- debit_cards
- recurring_expenses

---

### 5. Ejecutar Seeds (Datos Iniciales)

**Crear datos iniciales para desarrollo:**

```bash
npx prisma db seed
```

**Esto crea:**
- Categorías predefinidas (Departamento, Transporte, etc.)
- Tarjetas de ejemplo (BCI Crédito, BCI Débito, Cuenta RUT)
- Opcionalmente: Algunos gastos de ejemplo

**Verificar datos:**
```bash
psql -U bolsillo_user -d bolsillo_flow_dev -h localhost
```

```sql
-- Ver categorías
SELECT id, name, parent_id FROM categories;

-- Ver tarjetas
SELECT id, name, bank FROM credit_cards;
SELECT id, name, bank FROM debit_cards;

\q
```

---

### 6. Ejecutar Aplicación

#### Modo Desarrollo (con hot-reload)

```bash
npm run start:dev
```

**Output esperado:**
```
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG [NestFactory] Starting Nest application...
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG [InstanceLoader] AppModule dependencies initialized
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG [InstanceLoader] PrismaModule dependencies initialized
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG [InstanceLoader] CategoriesModule dependencies initialized
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG [RoutesResolver] CategoriesController {/api/categories}:
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG [RouterExplorer] Mapped {/api/categories, GET} route
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG [RouterExplorer] Mapped {/api/categories/:id, GET} route
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG [RouterExplorer] Mapped {/api/categories, POST} route
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG [NestApplication] Nest application successfully started
[Nest] 12345  - 01/02/2025, 10:00:00 AM     LOG Application is running on: http://localhost:3000
```

---

#### Verificar que la API Funciona

**Opción 1: Browser**
```
http://localhost:3000/api/health
```

**Opción 2: curl**
```bash
curl http://localhost:3000/api/health
```

**Response esperado:**
```json
{
  "status": "ok",
  "timestamp": "2025-01-02T10:00:00.000Z",
  "uptime": 1.234,
  "database": "connected"
}
```

**Probar endpoint de categorías:**
```bash
curl http://localhost:3000/api/categories
```

**Response esperado:**
```json
[
  {
    "id": "uuid...",
    "name": "Departamento",
    "description": "Gastos del departamento",
    ...
  },
  ...
]
```

---

### 7. Ejecutar Tests

#### Unit Tests

```bash
npm run test
```

**Output esperado:**
```
 PASS  src/financial/categories/categories.service.spec.ts
  CategoriesService
    ✓ should be defined (3 ms)
    ✓ should create category (5 ms)
    ✓ should find all categories (4 ms)

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
```

---

#### E2E Tests

```bash
npm run test:e2e
```

---

#### Test Coverage

```bash
npm run test:cov
```

**Genera reporte en:**
```
coverage/
  └─ lcov-report/
     └─ index.html
```

**Abrir reporte:**
```bash
open coverage/lcov-report/index.html
```

---

## Herramientas de Desarrollo

### 1. Prisma Studio (Database GUI)

**Abrir interfaz visual de la base de datos:**

```bash
npx prisma studio
```

**Se abre en:** `http://localhost:5555`

**Funcionalidades:**
- Ver todas las tablas
- Editar registros
- Crear registros manualmente
- Filtrar y buscar datos

---

### 2. ESLint (Linting)

**Ejecutar linter:**
```bash
npm run lint
```

**Fix automático:**
```bash
npm run lint -- --fix
```

---

### 3. Prettier (Formatting)

**Formatear código:**
```bash
npm run format
```

---

### 4. VS Code Extensions

**Crear `.vscode/extensions.json`:**

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "prisma.prisma",
    "ms-vscode.vscode-typescript-next",
    "firsttris.vscode-jest-runner"
  ]
}
```

**Instalar extensiones recomendadas:**
VS Code mostrará notificación para instalarlas automáticamente.

---

## Configuración de Git

### 1. Verificar .gitignore

**Debe incluir:**
```gitignore
# Dependencies
node_modules/

# Environment
.env
.env.local
.env.*.local

# Build
dist/
build/

# Testing
coverage/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
logs/
*.log
npm-debug.log*

# Prisma
*.db
*.db-journal
```

---

### 2. Configurar Git Hooks (Husky - Opcional)

**Instalar Husky:**
```bash
npm install --save-dev husky
npx husky install
```

**Crear pre-commit hook:**
```bash
npx husky add .husky/pre-commit "npm run lint && npm run test"
```

**Esto ejecutará linter y tests antes de cada commit.**

---

## Solución de Problemas Comunes

### Error: "Cannot connect to database"

**Verificar:**
1. PostgreSQL está corriendo
   ```bash
   # macOS
   brew services list | grep postgresql
   
   # Linux
   sudo systemctl status postgresql
   ```

2. Credenciales en `.env` son correctas

3. Database existe
   ```bash
   psql -U bolsillo_user -l
   ```

**Solución:**
```bash
# Reiniciar PostgreSQL
# macOS
brew services restart postgresql@15

# Linux
sudo systemctl restart postgresql
```

---

### Error: "Port 3000 is already in use"

**Verificar qué usa el puerto:**
```bash
# macOS/Linux
lsof -i :3000

# Windows
netstat -ano | findstr :3000
```

**Solución 1:** Matar el proceso
```bash
# macOS/Linux
kill -9 <PID>

# Windows
taskkill /PID <PID> /F
```

**Solución 2:** Cambiar puerto en `.env`
```env
PORT=3001
```

---

### Error: "Prisma generate failed"

**Limpiar y regenerar:**
```bash
rm -rf node_modules
rm -rf node_modules/.prisma
npm install
npx prisma generate
```

---

### Error: "Migration failed"

**Reset completo de base de datos (¡Cuidado! Borra todos los datos):**
```bash
npx prisma migrate reset
```

**Esto hace:**
1. Borra la base de datos
2. Recrea la base de datos
3. Aplica todas las migraciones
4. Ejecuta seeds

---

### Tests Fallan con "Database not found"

**Configurar base de datos de testing:**
```bash
# En .env.test
DATABASE_URL="postgresql://bolsillo_user:bolsillo_dev_password@localhost:5432/bolsillo_flow_test?schema=public"

# Aplicar migraciones a DB de test
npx prisma migrate deploy --schema=./prisma/schema.prisma
```

---

## Verificación Final

### Checklist de Setup Completo

- [ ] Node.js 20.x instalado
- [ ] PostgreSQL 15.x instalado
- [ ] Base de datos creada
- [ ] Usuario de BD creado
- [ ] Proyecto clonado
- [ ] Dependencies instaladas (`npm install`)
- [ ] `.env` configurado
- [ ] Prisma Client generado
- [ ] Migraciones aplicadas
- [ ] Seeds ejecutados
- [ ] Aplicación corre en modo dev
- [ ] Health endpoint responde
- [ ] Endpoints de categories funcionan
- [ ] Tests unitarios pasan
- [ ] Prisma Studio abre correctamente

**Ejecutar script de verificación:**
```bash
npm run verify-setup
```

---

## Próximos Pasos

Una vez completado el setup:

1. **Explorar la API:**
   - Revisar endpoints disponibles en `docs/04-api/endpoints.md`
   - Probar con Postman o Insomnia

2. **Familiarizarse con el código:**
   - Revisar estructura de módulos en `src/`
   - Leer documentación de arquitectura

3. **Empezar a desarrollar:**
   - Crear una feature branch
   - Implementar un endpoint nuevo
   - Escribir tests

4. **Recursos adicionales:**
   - [NestJS Documentation](https://docs.nestjs.com/)
   - [Prisma Documentation](https://www.prisma.io/docs)
   - [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

## Scripts NPM Disponibles

```bash
# Desarrollo
npm run start:dev          # Modo desarrollo con hot-reload
npm run start:debug        # Modo debug
npm run start:prod         # Modo producción

# Build
npm run build              # Compilar TypeScript

# Testing
npm run test               # Unit tests
npm run test:watch         # Tests en watch mode
npm run test:cov           # Test coverage
npm run test:e2e           # E2E tests

# Linting y Formatting
npm run lint               # Ejecutar ESLint
npm run format             # Formatear con Prettier

# Prisma
npm run prisma:generate    # Generar Prisma Client
npm run prisma:migrate     # Crear y aplicar migración
npm run prisma:studio      # Abrir Prisma Studio
npm run prisma:seed        # Ejecutar seeds

# Utilities
npm run verify-setup       # Verificar setup completo
```

---

## Referencias

- [Setup: Database](./database-setup.md)
- [Setup: IDE](./ide-setup.md)
- [Setup: Docker](./docker-setup.md) (opcional)
- [Tech Stack](../02-architecture/tech-stack.md)