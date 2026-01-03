# Configuración de Base de Datos

## Overview
Guía detallada para configurar PostgreSQL para Bolsillo Flow API, incluyendo instalación, configuración de usuarios, bases de datos, y optimizaciones.

---

## Instalación de PostgreSQL

### macOS

#### Opción 1: Homebrew (Recomendado)

```bash
# Instalar PostgreSQL 15
brew install postgresql@15

# Agregar al PATH
echo 'export PATH="/opt/homebrew/opt/postgresql@15/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Iniciar servicio
brew services start postgresql@15

# Verificar
psql --version
```

---

#### Opción 2: Postgres.app

1. Descargar desde [postgresapp.com](https://postgresapp.com/)
2. Mover a Applications
3. Abrir Postgres.app
4. Agregar al PATH:
   ```bash
   echo 'export PATH="/Applications/Postgres.app/Contents/Versions/15/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

---

### Linux (Ubuntu/Debian)

```bash
# Agregar repositorio oficial
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Importar key
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Actualizar e instalar
sudo apt-get update
sudo apt-get install postgresql-15 postgresql-contrib-15

# Verificar instalación
psql --version

# Verificar que está corriendo
sudo systemctl status postgresql
```

---

### Windows

1. Descargar instalador desde [postgresql.org](https://www.postgresql.org/download/windows/)
2. Ejecutar instalador
3. Durante instalación:
   - Puerto: 5432 (default)
   - Locale: Spanish_Chile.UTF-8 o English
   - Password: Guardar en lugar seguro
4. Verificar instalación:
   ```cmd
   psql --version
   ```

---

## Configuración Inicial

### 1. Acceder a PostgreSQL

**macOS/Linux:**
```bash
# Como usuario postgres (super user)
sudo -u postgres psql

# O directamente
psql postgres
```

**Windows:**
```cmd
# Desde cmd o PowerShell
psql -U postgres
```

**Prompt esperado:**
```
psql (15.10)
Type "help" for help.

postgres=#
```

---

### 2. Crear Usuario de Desarrollo

```sql
-- Crear usuario para la aplicación
CREATE USER bolsillo_user WITH PASSWORD 'bolsillo_dev_password';

-- Dar privilegios de crear bases de datos
ALTER USER bolsillo_user CREATEDB;

-- Verificar
\du
```

**Output esperado:**
```
                                   List of roles
    Role name    |                         Attributes                         
-----------------+------------------------------------------------------------
 bolsillo_user   | Create DB
 postgres        | Superuser, Create role, Create DB, Replication, Bypass RLS
```

---

### 3. Crear Bases de Datos

#### Base de Datos de Desarrollo

```sql
-- Crear database
CREATE DATABASE bolsillo_flow_dev 
  WITH 
  OWNER = bolsillo_user
  ENCODING = 'UTF8'
  LC_COLLATE = 'en_US.UTF-8'
  LC_CTYPE = 'en_US.UTF-8'
  TEMPLATE = template0;

-- Dar todos los privilegios
GRANT ALL PRIVILEGES ON DATABASE bolsillo_flow_dev TO bolsillo_user;

-- Conectar a la nueva database
\c bolsillo_flow_dev

-- Dar privilegios en el schema public
GRANT ALL ON SCHEMA public TO bolsillo_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO bolsillo_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO bolsillo_user;
```

---

#### Base de Datos de Testing

```sql
-- Salir a postgres database
\c postgres

-- Crear database de testing
CREATE DATABASE bolsillo_flow_test 
  WITH 
  OWNER = bolsillo_user
  ENCODING = 'UTF8'
  LC_COLLATE = 'en_US.UTF-8'
  LC_CTYPE = 'en_US.UTF-8'
  TEMPLATE = template0;

-- Dar privilegios
GRANT ALL PRIVILEGES ON DATABASE bolsillo_flow_test TO bolsillo_user;

-- Conectar
\c bolsillo_flow_test

-- Dar privilegios en schema
GRANT ALL ON SCHEMA public TO bolsillo_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO bolsillo_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO bolsillo_user;
```

---

### 4. Verificar Conexión

```bash
# Salir de psql
\q

# Conectar como bolsillo_user
psql -U bolsillo_user -d bolsillo_flow_dev -h localhost
```

**Si pide password:** Ingresar `bolsillo_dev_password`

**Si conecta exitosamente:**
```sql
-- Listar databases
\l

-- Verificar usuario actual
SELECT current_user;

-- Salir
\q
```

---

## Configuración de Acceso (pg_hba.conf)

### Ubicación del Archivo

**macOS (Homebrew):**
```bash
/opt/homebrew/var/postgresql@15/pg_hba.conf
```

**Linux:**
```bash
/etc/postgresql/15/main/pg_hba.conf
```

**Windows:**
```
C:\Program Files\PostgreSQL\15\data\pg_hba.conf
```

---

### Configuración Recomendada para Desarrollo

**Editar `pg_hba.conf`:**

```bash
# Linux
sudo nano /etc/postgresql/15/main/pg_hba.conf

# macOS
nano /opt/homebrew/var/postgresql@15/pg_hba.conf
```

**Agregar/modificar líneas:**

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Local connections (development)
local   all             all                                     trust
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5

# Allow connections from Docker containers (si usas Docker)
host    all             all             172.16.0.0/12           md5
```

**Explicación:**
- `trust`: No requiere password (solo local)
- `md5`: Requiere password con hash MD5
- `127.0.0.1/32`: Localhost IPv4
- `::1/128`: Localhost IPv6

---

### Reiniciar PostgreSQL

**macOS (Homebrew):**
```bash
brew services restart postgresql@15
```

**Linux:**
```bash
sudo systemctl restart postgresql
```

**Windows:**
```cmd
# Como administrador
net stop postgresql-x64-15
net start postgresql-x64-15
```

---

## Configuración de PostgreSQL (postgresql.conf)

### Ubicación del Archivo

**macOS:**
```bash
/opt/homebrew/var/postgresql@15/postgresql.conf
```

**Linux:**
```bash
/etc/postgresql/15/main/postgresql.conf
```

**Windows:**
```
C:\Program Files\PostgreSQL\15\data\postgresql.conf
```

---

### Configuraciones Recomendadas para Desarrollo

**Editar `postgresql.conf`:**

```bash
# Linux
sudo nano /etc/postgresql/15/main/postgresql.conf

# macOS
nano /opt/homebrew/var/postgresql@15/postgresql.conf
```

**Configuraciones clave:**

```conf
# Conexiones
max_connections = 100
port = 5432

# Memoria (ajustar según RAM disponible)
shared_buffers = 256MB          # 25% de RAM
effective_cache_size = 1GB      # 50-75% de RAM
work_mem = 16MB
maintenance_work_mem = 128MB

# Logging (útil para desarrollo)
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_statement = 'all'           # Loguear todas las queries (solo dev)
log_duration = on
log_min_duration_statement = 100 # Queries > 100ms

# Timezone
timezone = 'America/Santiago'   # Ajustar según ubicación

# Encoding
client_encoding = utf8
```

**Reiniciar después de cambios:**
```bash
# macOS
brew services restart postgresql@15

# Linux
sudo systemctl restart postgresql
```

---

## Configuración con Prisma

### 1. Connection String

**Formato:**
```
postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=SCHEMA
```

**Desarrollo (`.env`):**
```env
DATABASE_URL="postgresql://bolsillo_user:bolsillo_dev_password@localhost:5432/bolsillo_flow_dev?schema=public"
```

**Testing (`.env.test`):**
```env
DATABASE_URL="postgresql://bolsillo_user:bolsillo_dev_password@localhost:5432/bolsillo_flow_test?schema=public"
```

---

### 2. Configuración del Schema Prisma

**`prisma/schema.prisma`:**

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Modelos...
```

---

### 3. Inicializar Migraciones

**Primera migración:**
```bash
npx prisma migrate dev --name init
```

**Esto crea:**
```
prisma/
  └─ migrations/
     └─ 20250102000000_init/
        └─ migration.sql
```

---

### 4. Verificar Tablas Creadas

```bash
psql -U bolsillo_user -d bolsillo_flow_dev
```

```sql
-- Listar tablas
\dt

-- Describir tabla específica
\d categories

-- Ver índices
\di

-- Salir
\q
```

---

## Optimización de Performance

### Índices Recomendados

**Crear índices manualmente si no están en Prisma:**

```sql
-- Expenses: búsquedas frecuentes
CREATE INDEX IF NOT EXISTS idx_expenses_date ON expenses(date);
CREATE INDEX IF NOT EXISTS idx_expenses_billing_period ON expenses(billing_period);
CREATE INDEX IF NOT EXISTS idx_expenses_expense_type_id ON expenses(expense_type_id);
CREATE INDEX IF NOT EXISTS idx_expenses_credit_card_id ON expenses(credit_card_id) WHERE credit_card_id IS NOT NULL;
CREATE INDEX IF NOT EXISTS idx_expenses_debit_card_id ON expenses(debit_card_id) WHERE debit_card_id IS NOT NULL;

-- Budgets
CREATE INDEX IF NOT EXISTS idx_budgets_period ON budgets(period);
CREATE INDEX IF NOT EXISTS idx_budgets_category_id ON budgets(category_id);

-- Categories
CREATE INDEX IF NOT EXISTS idx_categories_parent_id ON categories(parent_id) WHERE parent_id IS NOT NULL;

-- Soft delete
CREATE INDEX IF NOT EXISTS idx_categories_deleted_at ON categories(deleted_at) WHERE deleted_at IS NULL;
CREATE INDEX IF NOT EXISTS idx_expense_types_deleted_at ON expense_types(deleted_at) WHERE deleted_at IS NULL;
```

---

### Análisis de Queries

**Habilitar análisis:**
```sql
-- Ver plan de ejecución
EXPLAIN ANALYZE 
SELECT * FROM expenses 
WHERE date >= '2025-01-01' AND date < '2025-02-01';

-- Ver queries lentas
SELECT 
  query,
  calls,
  total_time,
  mean_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

---

### Vacuum y Analyze

**Mantenimiento regular:**
```sql
-- Vacuum (limpiar espacio)
VACUUM ANALYZE;

-- Vacuum específico
VACUUM ANALYZE expenses;

-- Autovacuum (verificar está habilitado)
SHOW autovacuum;
```

---

## Backup y Restore

### Backup

**Backup completo:**
```bash
# Backup de database completa
pg_dump -U bolsillo_user -d bolsillo_flow_dev -F c -f backup_dev_$(date +%Y%m%d).dump

# Backup solo schema
pg_dump -U bolsillo_user -d bolsillo_flow_dev --schema-only -f schema_backup.sql

# Backup solo data
pg_dump -U bolsillo_user -d bolsillo_flow_dev --data-only -f data_backup.sql
```

---

### Restore

**Restore desde dump:**
```bash
# Restore completo
pg_restore -U bolsillo_user -d bolsillo_flow_dev -c backup_dev_20250102.dump

# Restore solo data
psql -U bolsillo_user -d bolsillo_flow_dev -f data_backup.sql
```

---

### Automatizar Backups (Opcional)

**Script de backup diario (Linux/macOS):**

```bash
#!/bin/bash
# backup_db.sh

DB_NAME="bolsillo_flow_dev"
DB_USER="bolsillo_user"
BACKUP_DIR="/path/to/backups"
DATE=$(date +%Y%m%d_%H%M%S)

pg_dump -U $DB_USER -d $DB_NAME -F c -f "$BACKUP_DIR/backup_${DATE}.dump"

# Mantener solo últimos 7 días
find $BACKUP_DIR -name "backup_*.dump" -mtime +7 -delete
```

**Agregar a crontab:**
```bash
# Ejecutar diario a las 3 AM
0 3 * * * /path/to/backup_db.sh
```

---

## Monitoreo

### Ver Conexiones Activas

```sql
SELECT 
  pid,
  usename,
  application_name,
  client_addr,
  state,
  query
FROM pg_stat_activity
WHERE datname = 'bolsillo_flow_dev';
```

---

### Ver Tamaño de Base de Datos

```sql
SELECT 
  pg_database.datname AS database_name,
  pg_size_pretty(pg_database_size(pg_database.datname)) AS size
FROM pg_database
WHERE datname IN ('bolsillo_flow_dev', 'bolsillo_flow_test');
```

---

### Ver Tamaño de Tablas

```sql
SELECT 
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

---

## Solución de Problemas

### No Puedo Conectar

**Error:** `could not connect to server: Connection refused`

**Solución:**
```bash
# Verificar que PostgreSQL está corriendo
# macOS
brew services list | grep postgresql

# Linux
sudo systemctl status postgresql

# Si no está corriendo, iniciar
brew services start postgresql@15
# o
sudo systemctl start postgresql
```

---

### Error de Autenticación

**Error:** `FATAL: password authentication failed for user "bolsillo_user"`

**Solución:**
1. Verificar password en `.env`
2. Resetear password:
   ```sql
   ALTER USER bolsillo_user WITH PASSWORD 'nueva_password';
   ```
3. Verificar `pg_hba.conf`

---

### Base de Datos No Existe

**Error:** `FATAL: database "bolsillo_flow_dev" does not exist`

**Solución:**
```bash
# Conectar a postgres
psql -U postgres

# Crear database
CREATE DATABASE bolsillo_flow_dev OWNER bolsillo_user;
GRANT ALL PRIVILEGES ON DATABASE bolsillo_flow_dev TO bolsillo_user;
```

---

### Puerto Ya en Uso

**Error:** `could not bind IPv4 address "127.0.0.1": Address already in use`

**Solución:**
```bash
# Ver qué proceso usa el puerto 5432
lsof -i :5432

# Si hay otro PostgreSQL, cambiar puerto en postgresql.conf
port = 5433

# O matar el proceso conflictivo
kill -9 <PID>
```

---

### Migración Falla

**Error:** `Migration failed: relation "table_name" does not exist`

**Solución:**
```bash
# Reset completo (BORRA DATOS)
npx prisma migrate reset

# Verificar estado de migraciones
npx prisma migrate status

# Aplicar migraciones pendientes
npx prisma migrate deploy
```

---

## Extensiones Útiles

### Instalar Extensiones

```sql
-- Conectar a la database
\c bolsillo_flow_dev

-- UUID (si no está por defecto)
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- pg_stat_statements (análisis de queries)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- pg_trgm (búsqueda fuzzy)
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Verificar extensiones instaladas
\dx
```

---

## Scripts Útiles

### Script de Verificación

**`scripts/check_db.sh`:**

```bash
#!/bin/bash

echo "Checking PostgreSQL connection..."

psql -U bolsillo_user -d bolsillo_flow_dev -c "SELECT 'Connection OK' as status;"

if [ $? -eq 0 ]; then
  echo "✓ Database connection successful"
  
  echo ""
  echo "Tables:"
  psql -U bolsillo_user -d bolsillo_flow_dev -c "\dt"
  
  echo ""
  echo "Database size:"
  psql -U bolsillo_user -d bolsillo_flow_dev -c "SELECT pg_size_pretty(pg_database_size('bolsillo_flow_dev'));"
else
  echo "✗ Database connection failed"
  exit 1
fi
```

---

### Script de Reset

**`scripts/reset_db.sh`:**

```bash
#!/bin/bash

read -p "This will DELETE ALL DATA. Continue? (y/N) " -n 1 -r
echo

if [[ $REPLY =~ ^[Yy]$ ]]; then
  echo "Resetting database..."
  npx prisma migrate reset --force
  echo "✓ Database reset complete"
else
  echo "Cancelled"
fi
```

---

## Referencias

- [PostgreSQL Documentation](https://www.postgresql.org/docs/15/index.html)
- [Prisma PostgreSQL Guide](https://www.prisma.io/docs/concepts/database-connectors/postgresql)
- [Setup: Development](./development-setup.md)
- [Setup: Docker](./docker-setup.md)