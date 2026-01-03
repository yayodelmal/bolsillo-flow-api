# Bolsillo Flow API

API REST para gestión de finanzas personales construida con NestJS, Prisma y PostgreSQL.

## 🚀 Características

- ✅ Gestión de gastos con categorías jerárquicas
- ✅ Seguimiento de presupuestos mensuales
- ✅ Cálculo automático de períodos de facturación (tarjetas de crédito)
- ✅ Gestión de cuotas con auto-completado
- ✅ Gastos recurrentes con generación automática
- ✅ Soporte para múltiples métodos de pago
- ✅ Reportes mensuales y análisis de gastos

## 📋 Requisitos

- Node.js 20+
- PostgreSQL 15+
- npm 10+

## 🛠️ Stack Tecnológico

- **Framework:** NestJS
- **ORM:** Prisma
- **Base de datos:** PostgreSQL
- **Lenguaje:** TypeScript
- **Testing:** Jest
- **Validación:** class-validator

## ⚡ Quick Start

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/bolsillo-flow-api.git
cd bolsillo-flow-api
```

### 2. Instalar dependencias

```bash
npm install
```

### 3. Configurar variables de entorno

```bash
cp .env.example .env
```

Editar `.env` con tus valores:

```env
DATABASE_URL="postgresql://user:password@localhost:5432/bolsillo_flow_dev"
PORT=3000
NODE_ENV=development
```

### 4. Configurar base de datos

```bash
# Crear base de datos en PostgreSQL
psql postgres
CREATE DATABASE bolsillo_flow_dev;
CREATE USER bolsillo_user WITH PASSWORD 'tu_password';
GRANT ALL PRIVILEGES ON DATABASE bolsillo_flow_dev TO bolsillo_user;
\q
```

### 5. Ejecutar migraciones

```bash
npx prisma migrate dev
```

### 6. (Opcional) Cargar datos de ejemplo

```bash
npx prisma db seed
```

### 7. Iniciar servidor de desarrollo

```bash
npm run start:dev
```

La API estará disponible en `http://localhost:3000`

## 🐳 Docker (Alternativa)

```bash
# Iniciar todos los servicios
docker-compose up

# En background
docker-compose up -d
```

## 📝 Scripts Disponibles

```bash
# Desarrollo
npm run start:dev        # Servidor con hot-reload
npm run start:debug      # Modo debug

# Build
npm run build            # Compilar para producción
npm run start:prod       # Ejecutar en producción

# Testing
npm run test             # Unit tests
npm run test:watch       # Tests en modo watch
npm run test:cov         # Coverage
npm run test:e2e         # End-to-end tests

# Base de datos
npm run prisma:generate  # Generar Prisma Client
npm run prisma:migrate   # Ejecutar migraciones
npm run prisma:studio    # Abrir Prisma Studio
npm run prisma:seed      # Cargar datos de ejemplo

# Calidad de código
npm run lint             # Ejecutar ESLint
npm run format           # Formatear con Prettier
```

## 🧪 Testing

```bash
# Unit tests
npm run test

# E2E tests
npm run test:e2e

# Coverage report
npm run test:cov
```

## 📚 Documentación

Documentación completa disponible en la carpeta `docs/`:

- **Setup:** [Configuración del entorno](docs/06-setup/development-setup.md)
- **API:** [Endpoints completos](docs/04-api/endpoints.md)
- **Arquitectura:** [Overview del sistema](docs/02-architecture/overview.md)
- **Workflows:** [Flujos principales](docs/05-workflows/)
- **Reglas de negocio:** [Business rules](docs/03-data-model/business-rules.md)

## 🔌 API Endpoints

### Categories

```
GET    /api/categories           # Listar categorías
GET    /api/categories/:id       # Obtener categoría
POST   /api/categories           # Crear categoría
PUT    /api/categories/:id       # Actualizar categoría
DELETE /api/categories/:id       # Eliminar categoría (soft)
```

### Expenses

```
GET    /api/expenses             # Listar gastos
GET    /api/expenses/:id         # Obtener gasto
POST   /api/expenses             # Crear gasto
PUT    /api/expenses/:id         # Actualizar gasto
DELETE /api/expenses/:id         # Eliminar gasto
GET    /api/expenses/summary     # Resumen de gastos
```

### Budgets

```
GET    /api/budgets              # Listar presupuestos
POST   /api/budgets              # Crear/actualizar presupuesto
GET    /api/budgets/category/:categoryId/period/:period
```

### Recurring Expenses

```
GET    /api/recurring-expenses   # Listar recurrentes
POST   /api/recurring-expenses   # Crear recurrente
POST   /api/recurring-expenses/generate  # Generar gastos del mes
```

[Ver documentación completa de endpoints](docs/04-api/endpoints.md)

## 🏗️ Estructura del Proyecto

```
src/
├── common/              # Utilidades compartidas
│   ├── decorators/
│   ├── filters/
│   └── pipes/
├── financial/           # Dominio financiero
│   ├── categories/
│   ├── budgets/
│   ├── expenses/
│   └── expense-types/
├── payment-methods/     # Dominio de métodos de pago
│   ├── credit-cards/
│   └── debit-cards/
├── automation/          # Dominio de automatización
│   └── recurring-expenses/
├── prisma/              # Módulo de Prisma
│   └── prisma.service.ts
└── main.ts              # Entry point
```

## 🗄️ Modelo de Datos

### Entidades Principales

- **Category:** Categorías jerárquicas (2 niveles)
- **Budget:** Presupuestos mensuales por categoría
- **ExpenseType:** Tipos de gasto (con soporte para cuotas y recurrentes)
- **Expense:** Registros de gastos individuales
- **CreditCard:** Tarjetas de crédito
- **DebitCard:** Tarjetas de débito
- **RecurringExpense:** Configuración de gastos recurrentes

[Ver diagrama completo](docs/03-data-model/relationships.md)

## 🔐 Variables de Entorno

```env
# Server
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/db_name

# JWT (futuro)
JWT_SECRET=your-secret-key
JWT_EXPIRATION=7d
```

## 🤝 Contribuir

1. Fork el proyecto
2. Crear una rama (`git checkout -b feature/nueva-feature`)
3. Commit cambios (`git commit -m 'Add: nueva feature'`)
4. Push a la rama (`git push origin feature/nueva-feature`)
5. Abrir Pull Request

### Convenciones de Commits

```
Add: nueva funcionalidad
Fix: corrección de bug
Update: actualización de funcionalidad existente
Refactor: refactorización de código
Docs: cambios en documentación
Test: agregar o modificar tests
```

## 📄 Licencia

Este proyecto está bajo la licencia MIT.

## 👤 Autor

**Tu Nombre**
- GitHub: [@tu-usuario](https://github.com/tu-usuario)

## 📞 Soporte

- Documentación: [/docs](docs/)
- Issues: [GitHub Issues](https://github.com/tu-usuario/bolsillo-flow-api/issues)

## 🗺️ Roadmap

- [x] CRUD de categorías
- [x] CRUD de gastos
- [x] Gestión de presupuestos
- [x] Cálculo automático de billing cycle
- [x] Gestión de cuotas
- [x] Gastos recurrentes
- [ ] Autenticación y autorización
- [ ] Multi-usuario
- [ ] Reportes avanzados
- [ ] Exportación de datos
- [ ] API pública con rate limiting

[Ver backlog completo](docs/01-product/backlog.md)

## ⚙️ Configuración Adicional

### Prisma Studio

Interfaz visual para explorar la base de datos:

```bash
npx prisma studio
```

Abre en `http://localhost:5555`

### ESLint + Prettier

```bash
# Ejecutar linter
npm run lint

# Fix automático
npm run lint -- --fix

# Formatear código
npm run format
```

## 🐛 Troubleshooting

### No puedo conectar a la base de datos

Verifica que PostgreSQL esté corriendo:

```bash
# macOS
brew services list | grep postgresql

# Linux
sudo systemctl status postgresql
```

### Error en migraciones

Reset completo de la base de datos (⚠️ borra todos los datos):

```bash
npx prisma migrate reset
```

### Puerto 3000 ya en uso

Cambiar puerto en `.env`:

```env
PORT=3001
```

[Ver guía completa de troubleshooting](docs/06-setup/development-setup.md#solución-de-problemas-comunes)

---

**Happy coding! 🚀**