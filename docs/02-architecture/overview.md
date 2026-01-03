# Architecture Overview

## Resumen Ejecutivo

Bolsillo Flow API es una aplicación backend RESTful construida con **NestJS**, **TypeScript** y **PostgreSQL**, diseñada para gestionar finanzas personales con enfoque en control de gastos, presupuestos y cuotas.

La arquitectura sigue principios de **Domain-Driven Design (lite)**, priorizando:
- ✅ Separación clara de responsabilidades por dominio
- ✅ Type safety end-to-end
- ✅ Escalabilidad y mantenibilidad
- ✅ Testing como ciudadano de primera clase

---

## Vista de Alto Nivel

```
┌─────────────────────────────────────────────────────────────┐
│                         Cliente                             │
│              (Postman, Frontend futuro, Mobile)             │
└────────────────────────────┬────────────────────────────────┘
                             │
                             │ HTTP/REST
                             │
┌────────────────────────────▼────────────────────────────────┐
│                      NestJS API Server                      │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Controllers (REST Endpoints)             │  │
│  └────────┬──────────────┬──────────────┬────────────────┘  │
│           │              │              │                   │
│  ┌────────▼─────┐   ┌────▼─────┐  ┌─────▼─────────┐         │
│  │  Financial   │   │ Payment  │  │ Automation    │         │
│  │   Domain     │   │ Methods  │  │   Domain      │         │
│  │              │   │  Domain  │  │               │         │
│  │ • Categories │   │ • Credit │  │ • Recurring   │         │
│  │ • Budgets    │   │   Cards  │  │   Expenses    │         │
│  │ • Expenses   │   │ • Debit  │  │               │         │
│  │ • Exp.Types  │   │   Cards  │  │               │         │
│  └────────┬─────┘   └────┬─────┘  └───────┬───────┘         │
│           │              │                │                 │
│           └──────────────┼────────────────┘                 │
│                          │                                  │
│           ┌──────────────▼───────────────┐                  │
│           │      Prisma ORM Service      │                  │
│           │   (Database Abstraction)     │                  │
│           └──────────────┬───────────────┘                  │
└──────────────────────────┼──────────────────────────────────┘
                           │
                           │ SQL
                           │
┌──────────────────────────▼───────────────────────────────────┐
│                   PostgreSQL Database                        │
│                                                              │
│  Tables: categories, budgets, expenses, expense_types,       │
│          credit_cards, debit_cards, recurring_expenses       │
└──────────────────────────────────────────────────────────────┘
```

---

## Capas de la Aplicación

### 1. Presentation Layer (Controllers)
**Responsabilidad:** Exponer la API REST y manejar requests HTTP.

**Tecnologías:**
- NestJS Controllers
- Decorators (@Get, @Post, @Put, @Delete)
- DTOs para validación de entrada

**Características:**
- Validación de input usando class-validator
- Transformación de output usando class-transformer
- Manejo de errores HTTP
- Documentación con decorators (futuro: Swagger)

**Ejemplo:**
```typescript
@Controller('categories')
export class CategoriesController {
  constructor(private readonly categoriesService: CategoriesService) {}

  @Get()
  async findAll(@Query() query: QueryCategoriesDto) {
    return this.categoriesService.findAll(query);
  }

  @Post()
  async create(@Body() dto: CreateCategoryDto) {
    return this.categoriesService.create(dto);
  }
}
```

---

### 2. Business Logic Layer (Services)
**Responsabilidad:** Implementar la lógica de negocio y reglas de dominio.

**Tecnologías:**
- NestJS Services
- Dependency Injection
- TypeScript para type safety

**Características:**
- Lógica de negocio centralizada
- Validaciones complejas
- Orquestación de operaciones
- Cálculos específicos del dominio

**Ejemplo:**
```typescript
@Injectable()
export class ExpensesService {
  constructor(private prisma: PrismaService) {}

  async create(dto: CreateExpenseDto) {
    // Validación de negocio
    await this.validateExpenseType(dto.expense_type_id);
    
    // Cálculo de billing period si es crédito
    const billingPeriod = dto.payment_type === 'credit'
      ? this.calculateBillingPeriod(dto.date, dto.credit_card_id)
      : null;
    
    // Creación
    return this.prisma.expense.create({
      data: { ...dto, billing_period: billingPeriod }
    });
  }
  
  private calculateBillingPeriod(date: Date, cardId: string): string {
    // Lógica de cálculo...
  }
}
```

---

### 3. Data Access Layer (Prisma)
**Responsabilidad:** Abstracción de acceso a base de datos.

**Tecnologías:**
- Prisma ORM
- Prisma Client (auto-generado)
- PostgreSQL

**Características:**
- Type-safe queries
- Migrations automáticas
- Relations manejadas por Prisma
- Connection pooling
- Transacciones ACID

**Ejemplo:**
```typescript
@Injectable()
export class PrismaService extends PrismaClient {
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

---

### 4. Cross-Cutting Concerns (Common)
**Responsabilidad:** Funcionalidades compartidas entre capas.

**Componentes:**
- **Filters**: Exception handling global
- **Interceptors**: Logging, transformación de respuestas
- **Guards**: Autenticación y autorización (futuro)
- **Pipes**: Validación y transformación global
- **Decorators**: Metadata personalizada

**Ejemplo - Exception Filter:**
```typescript
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();
    
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      message: exception.message,
    });
  }
}
```

---

## Flujo de Request Completo

### Ejemplo: Crear un Gasto

```
1. HTTP Request
   POST /expenses
   Body: { expense_type_id, amount, date, payment_type, credit_card_id }
   │
   ├─> Global Validation Pipe
   │   └─> Valida formato de datos (DTOs)
   │
2. Controller Layer
   └─> ExpensesController.create()
       │
       ├─> Recibe CreateExpenseDto validado
       │
3. Service Layer
   └─> ExpensesService.create()
       │
       ├─> Valida expense_type existe
       ├─> Valida credit_card existe
       ├─> Calcula billing_period automáticamente
       │
4. Data Access Layer
   └─> PrismaService.expense.create()
       │
       ├─> Genera SQL
       ├─> Ejecuta transaction
       │
5. Database
   └─> PostgreSQL
       │
       ├─> INSERT INTO expenses (...)
       ├─> COMMIT
       │
6. Response Flow (reversa)
   │
   ├─> Prisma retorna objeto tipado
   ├─> Service aplica transformaciones si necesita
   ├─> Controller aplica DTOs de response
   ├─> Transform Interceptor formatea respuesta
   │
7. HTTP Response
   200 OK
   Body: { id, expense_type, amount, date, billing_period, ... }
```

---

## Patrones Arquitectónicos Utilizados

### 1. Domain-Driven Design (Lite)
**Qué es:**
Organización del código por dominios de negocio en lugar de por tipo técnico.

**Implementación:**
- Módulos agrupados por dominio (financial/, payment-methods/, automation/)
- Bounded contexts claros
- Sin agregados complejos ni value objects (lite)

**Beneficios:**
- Código relacionado está junto
- Fácil de entender el negocio
- Escala bien

---

### 2. Dependency Injection
**Qué es:**
Patrón donde las dependencias se inyectan en lugar de crearlas internamente.

**Implementación:**
- NestJS lo hace nativamente
- Services se inyectan en constructores
- Facilita testing con mocks

**Ejemplo:**
```typescript
@Injectable()
export class ExpensesService {
  constructor(
    private prisma: PrismaService,  // Inyectado
  ) {}
}
```

**Beneficios:**
- Desacoplamiento
- Testing simple (mock de dependencias)
- Configuración centralizada

---

### 3. Repository Pattern
**Qué es:**
Abstracción del acceso a datos.

**Implementación:**
- Prisma Service actúa como repository
- Services usan Prisma, no SQL directo
- Queries centralizadas en capa de datos

**Beneficios:**
- Cambiar ORM es más fácil
- Testing: mock de PrismaService
- Queries optimizadas en un solo lugar

---

### 4. DTO Pattern
**Qué es:**
Data Transfer Objects para validación y transformación.

**Implementación:**
- DTOs de entrada (CreateXDto, UpdateXDto)
- DTOs de salida (XResponseDto)
- Validación declarativa con decorators

**Ejemplo:**
```typescript
export class CreateExpenseDto {
  @IsUUID()
  expense_type_id: string;

  @IsNumber()
  @Min(0)
  amount: number;

  @IsDate()
  @Type(() => Date)
  date: Date;

  @IsEnum(PaymentType)
  payment_type: PaymentType;
}
```

**Beneficios:**
- Validación automática
- Type safety
- Documentación auto-descriptiva

---

### 5. Soft Delete Pattern
**Qué es:**
No borrar físicamente, marcar como eliminado.

**Implementación:**
- Campo `deleted_at` en entidades principales
- Queries filtran por `deleted_at IS NULL`
- Posibilidad de restaurar

**Beneficios:**
- Auditoría completa
- Recuperación de errores
- Integridad referencial

---

## Principios de Diseño

### SOLID Principles

**S - Single Responsibility Principle**
- Cada service tiene una responsabilidad clara
- CategoriesService solo maneja categorías
- No hay "god objects"

**O - Open/Closed Principle**
- Extender con nuevos módulos sin modificar existentes
- Agregar nuevo dominio no afecta otros

**L - Liskov Substitution Principle**
- Interfaces bien definidas
- PrismaService puede ser mockeado en tests

**I - Interface Segregation Principle**
- DTOs específicos por caso de uso
- No DTOs gigantes con campos opcionales

**D - Dependency Inversion Principle**
- Services dependen de abstracciones (PrismaService)
- No dependencia directa de implementaciones concretas

---

### Additional Principles

**DRY (Don't Repeat Yourself)**
- Código común en `common/`
- Helpers reutilizables
- Configuración centralizada

**KISS (Keep It Simple, Stupid)**
- DDD-lite en lugar de DDD completo
- No over-engineering
- Complejidad solo cuando se necesita

**YAGNI (You Aren't Gonna Need It)**
- Features solo cuando se necesitan
- No código especulativo
- MVP primero, features después

---

## Gestión de Estado

### Stateless API
La API es **stateless**: no mantiene estado de sesión entre requests.

**Implicaciones:**
- Cada request tiene toda la info necesaria
- No sessions en memoria
- Escalabilidad horizontal simple

**Excepción:**
- Connection pool de DB (manejado por Prisma)
- Caché futuro (opcional, para reportes)

---

## Manejo de Errores

### Estrategia de Errores

**1. Validation Errors (400 Bad Request)**
```typescript
// DTO validation automática
{
  "statusCode": 400,
  "message": ["amount must be a positive number"],
  "error": "Bad Request"
}
```

**2. Not Found Errors (404)**
```typescript
// Cuando recurso no existe
throw new NotFoundException('Category not found');
```

**3. Business Logic Errors (422 Unprocessable Entity)**
```typescript
// Cuando validación de negocio falla
throw new UnprocessableEntityException('Cannot delete category with active children');
```

**4. Server Errors (500)**
```typescript
// Errores inesperados
// Logged pero no exponen detalles internos
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

### Exception Filter
```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    // Log del error
    // Formateo consistente de respuesta
    // Ocultamiento de detalles sensibles en producción
  }
}
```

---

## Seguridad

### Implementado (MVP)

**1. Input Validation**
- Validación exhaustiva con class-validator
- Sanitización de inputs
- Type safety con TypeScript

**2. SQL Injection Prevention**
- Prepared statements (Prisma automático)
- No SQL concatenado
- ORM como capa de protección

**3. Sensitive Data**
- Secrets en variables de entorno (.env)
- .env en .gitignore
- No hardcodear credenciales

**4. CORS**
- Configuración apropiada de CORS
- Whitelist de orígenes permitidos

---

### Pendiente (Post-MVP)

**1. Authentication**
- JWT tokens
- Passport.js
- Refresh tokens

**2. Authorization**
- Role-based access control
- Guards de NestJS
- Permisos granulares

**3. Rate Limiting**
- Throttler de NestJS
- Límites por IP
- Prevención de DDoS

**4. Encryption**
- HTTPS en producción
- Encriptación de datos sensibles
- Hashing de passwords (si se agrega auth)

---

## Performance Considerations

### Database

**Indexing:**
```sql
-- Índices planeados
CREATE INDEX idx_expenses_date ON expenses(date);
CREATE INDEX idx_expenses_billing_period ON expenses(billing_period);
CREATE INDEX idx_budgets_period ON budgets(period);
```

**Query Optimization:**
- Uso de `select` en Prisma para traer solo campos necesarios
- `include` estratégico para evitar N+1 queries
- Paginación en listados grandes

**Connection Pooling:**
- Manejado automáticamente por Prisma
- Pool size configurado según carga esperada

---

### API

**Caching (Futuro):**
- Reportes mensuales cacheados
- TTL de 5-10 minutos
- Invalidación al crear/actualizar gastos

**Compression:**
- Gzip/Brotli en responses grandes
- Middleware de compresión

**Lazy Loading:**
- Relaciones solo cuando se necesitan
- DTOs optimizados por endpoint

---

## Escalabilidad

### Horizontal Scaling

**Stateless API permite:**
- Múltiples instancias detrás de load balancer
- Auto-scaling basado en carga
- Zero-downtime deployments

**Preparación:**
- No estado en memoria
- DB como única fuente de verdad
- Configuración externalizada

---

### Vertical Scaling

**Base de datos:**
- PostgreSQL escala bien verticalmente
- Read replicas (futuro, si se necesita)
- Particionamiento de tablas grandes (futuro)

**Aplicación:**
- Node.js multi-core con cluster mode
- PM2 para gestión de procesos

---

### Estrategia de Crecimiento

**Fase 1 (MVP):**
```
┌─────────────┐
│  NestJS API │
└──────┬──────┘
       │
┌──────▼──────┐
│ PostgreSQL  │
└─────────────┘
```

**Fase 2 (Post-MVP):**
```
┌──────────────┐
│ Load Balancer│
└──────┬───────┘
       │
    ┌──┴──┐
    │     │
┌───▼─┐ ┌─▼───┐
│API 1│ │API 2│
└──┬──┘ └──┬──┘
   └───┬───┘
┌──────▼──────┐
│ PostgreSQL  │
└─────────────┘
```

**Fase 3 (Futuro lejano):**
```
┌──────────────┐
│ Load Balancer│
└──────┬───────┘
       │
    ┌──┴────┬──────┐
┌───▼─┐ ┌───▼─┐ ┌──▼──┐
│API 1│ │API 2│ │API 3│
└──┬──┘ └──┬──┘ └──┬──┘
   │       │       │
┌──▼───────▼───────▼──┐
│   Redis Cache        │
└──────────┬───────────┘
           │
    ┌──────┴──────┐
┌───▼────┐  ┌─────▼──────┐
│ Master │  │Read Replica│
│  DB    │  │    DB      │
└────────┘  └────────────┘
```

---

## Testing Strategy

### Pirámide de Testing

```
           ┌─────┐
          /  E2E  \      ← Pocos (críticos)
         /_________\
        /           \
       / Integration \   ← Algunos (workflows)
      /______________\
     /                \
    /   Unit Tests     \  ← Muchos (lógica)
   /____________________\
```

### Unit Tests
- Cada service tiene suite de tests
- Mock de PrismaService
- Validación de lógica de negocio
- Coverage objetivo: >80%

**Ejemplo:**
```typescript
describe('CategoriesService', () => {
  let service: CategoriesService;
  let prisma: DeepMockProxy<PrismaService>;

  beforeEach(() => {
    prisma = mockDeep<PrismaService>();
    service = new CategoriesService(prisma);
  });

  it('should create category', async () => {
    const dto = { name: 'Test', description: null };
    prisma.category.create.mockResolvedValue({ id: '1', ...dto });
    
    const result = await service.create(dto);
    expect(result.name).toBe('Test');
  });
});
```

### Integration Tests
- Flujos completos de negocio
- Base de datos real (test DB)
- Validación de integraciones entre módulos

### E2E Tests
- Endpoints críticos
- Flujos de usuario completos
- Supertest para llamadas HTTP

---

## Deployment Architecture

### Development
```
Local Machine:
├─ Node.js v20.19.0
├─ PostgreSQL 15 (local o Docker)
├─ npm run start:dev (hot reload)
└─ Prisma Studio para explorar datos
```

### Production (Futuro)
```
Cloud Provider (AWS/GCP/Railway):
├─ App Server
│  ├─ Node.js runtime
│  ├─ PM2 process manager
│  └─ Environment variables
├─ Database
│  ├─ Managed PostgreSQL
│  ├─ Automated backups
│  └─ Monitoring
└─ CI/CD Pipeline
   ├─ GitHub Actions
   ├─ Automated tests
   └─ Zero-downtime deployment
```

---

## Monitoring & Observability (Futuro)

### Logging
- Winston/Pino para structured logging
- Niveles: error, warn, info, debug
- Contexto rico (request ID, user, timestamp)

### Metrics
- Prometheus para métricas
- Grafana para visualización
- Métricas clave: response time, error rate, throughput

### Tracing
- Distributed tracing con Jaeger/Zipkin
- Seguimiento de requests end-to-end

### Alerting
- Alertas en Slack/Email
- Triggers: errores 5xx, latencia alta, DB slow queries

---

## Documentation Strategy

### Code Documentation
- JSDoc en funciones complejas
- README en cada módulo
- Inline comments donde necesario

### API Documentation
- Swagger/OpenAPI (futuro)
- Auto-generado desde decorators
- Ejemplos de requests/responses

### Architecture Documentation
- Este documento (overview)
- ADRs para decisiones importantes
- Diagramas actualizados

---

## Referencias

- [NestJS Official Docs](https://docs.nestjs.com/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Domain-Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)