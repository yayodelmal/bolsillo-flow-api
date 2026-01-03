# Estructura de Módulos

## Overview
Bolsillo Flow API utiliza una arquitectura **DDD-lite (Domain-Driven Design ligero)**, organizando el código por dominios de negocio en lugar de por tipo de componente técnico.

---

## Principios de Diseño

### 1. Separación por Dominio
Los módulos se agrupan según el contexto de negocio al que pertenecen, no por su función técnica.

**❌ Evitamos (organización técnica):**
```
src/
├── controllers/
│   ├── categories.controller.ts
│   ├── expenses.controller.ts
│   └── credit-cards.controller.ts
├── services/
│   ├── categories.service.ts
│   ├── expenses.service.ts
│   └── credit-cards.service.ts
└── models/
    ├── category.model.ts
    ├── expense.model.ts
    └── credit-card.model.ts
```

**✅ Preferimos (organización por dominio):**
```
src/
├── financial/
│   ├── categories/
│   ├── expenses/
│   └── budgets/
├── payment-methods/
│   ├── credit-cards/
│   └── debit-cards/
└── automation/
    └── recurring-expenses/
```

### 2. Módulos Auto-Contenidos
Cada módulo contiene todo lo necesario para su funcionalidad:
- Controller (endpoints)
- Service (lógica de negocio)
- DTOs (validación y transformación)
- Tests (unitarios e integración)

### 3. Bounded Contexts
Cada dominio tiene responsabilidades claras y bien definidas:
- **Financial**: Todo lo relacionado con dinero y gastos
- **Payment Methods**: Formas de pago y tarjetas
- **Automation**: Automatización de procesos repetitivos
- **Reports**: Análisis y reportería (futuro)

---

## Estructura Completa del Proyecto

```
bolsillo-flow-api/
│
├── src/
│   ├── main.ts                          # Entry point de la aplicación
│   ├── app.module.ts                    # Módulo raíz
│   │
│   ├── common/                          # Código compartido entre módulos
│   │   ├── decorators/                  # Decorators personalizados
│   │   │   └── soft-delete.decorator.ts
│   │   ├── filters/                     # Exception filters
│   │   │   ├── http-exception.filter.ts
│   │   │   └── prisma-exception.filter.ts
│   │   ├── guards/                      # Guards (auth, roles, etc)
│   │   │   └── auth.guard.ts            # Futuro
│   │   ├── interceptors/                # Interceptors (logging, transform)
│   │   │   ├── logging.interceptor.ts
│   │   │   └── transform.interceptor.ts
│   │   ├── pipes/                       # Validation pipes
│   │   │   └── validation.pipe.ts
│   │   └── types/                       # Tipos TypeScript compartidos
│   │       ├── payment-type.enum.ts
│   │       └── recurring-frequency.enum.ts
│   │
│   ├── config/                          # Configuración
│   │   ├── app.config.ts                # Config general de la app
│   │   └── database.config.ts           # Config de BD
│   │
│   ├── prisma/                          # Todo lo relacionado con Prisma
│   │   ├── prisma.module.ts             # Módulo de Prisma
│   │   ├── prisma.service.ts            # Servicio singleton de Prisma
│   │   ├── schema.prisma                # Schema de BD
│   │   ├── migrations/                  # Migraciones versionadas
│   │   │   ├── 20250102_init/
│   │   │   └── ...
│   │   └── seeds/                       # Seeds de datos iniciales
│   │       ├── seed.ts                  # Script principal de seed
│   │       ├── categories.seed.ts       # Seed de categorías
│   │       └── cards.seed.ts            # Seed de tarjetas iniciales
│   │
│   ├── financial/                       # DOMINIO: Gestión financiera
│   │   │
│   │   ├── categories/                  # Módulo: Categorías
│   │   │   ├── categories.module.ts
│   │   │   ├── categories.controller.ts
│   │   │   ├── categories.service.ts
│   │   │   ├── dto/
│   │   │   │   ├── create-category.dto.ts
│   │   │   │   ├── update-category.dto.ts
│   │   │   │   ├── category-response.dto.ts
│   │   │   │   └── query-categories.dto.ts
│   │   │   ├── entities/                # Opcional: interfaces adicionales
│   │   │   │   └── category.entity.ts
│   │   │   └── tests/
│   │   │       ├── categories.controller.spec.ts
│   │   │       └── categories.service.spec.ts
│   │   │
│   │   ├── budgets/                     # Módulo: Presupuestos
│   │   │   ├── budgets.module.ts
│   │   │   ├── budgets.controller.ts
│   │   │   ├── budgets.service.ts
│   │   │   ├── dto/
│   │   │   │   ├── create-budget.dto.ts
│   │   │   │   ├── update-budget.dto.ts
│   │   │   │   └── budget-status.dto.ts
│   │   │   └── tests/
│   │   │
│   │   ├── expenses/                    # Módulo: Gastos
│   │   │   ├── expenses.module.ts
│   │   │   ├── expenses.controller.ts
│   │   │   ├── expenses.service.ts
│   │   │   ├── dto/
│   │   │   │   ├── create-expense.dto.ts
│   │   │   │   ├── update-expense.dto.ts
│   │   │   │   ├── expense-response.dto.ts
│   │   │   │   └── query-expenses.dto.ts
│   │   │   └── tests/
│   │   │
│   │   └── expense-types/               # Módulo: Tipos de Gasto
│   │       ├── expense-types.module.ts
│   │       ├── expense-types.controller.ts
│   │       ├── expense-types.service.ts
│   │       ├── dto/
│   │       │   ├── create-expense-type.dto.ts
│   │       │   ├── update-expense-type.dto.ts
│   │       │   └── installment-config.dto.ts
│   │       └── tests/
│   │
│   ├── payment-methods/                 # DOMINIO: Métodos de pago
│   │   │
│   │   ├── credit-cards/                # Módulo: Tarjetas de crédito
│   │   │   ├── credit-cards.module.ts
│   │   │   ├── credit-cards.controller.ts
│   │   │   ├── credit-cards.service.ts
│   │   │   ├── dto/
│   │   │   │   ├── create-credit-card.dto.ts
│   │   │   │   ├── update-credit-card.dto.ts
│   │   │   │   └── next-payment.dto.ts
│   │   │   └── tests/
│   │   │
│   │   └── debit-cards/                 # Módulo: Tarjetas de débito
│   │       ├── debit-cards.module.ts
│   │       ├── debit-cards.controller.ts
│   │       ├── debit-cards.service.ts
│   │       ├── dto/
│   │       │   ├── create-debit-card.dto.ts
│   │       │   └── update-debit-card.dto.ts
│   │       └── tests/
│   │
│   ├── automation/                      # DOMINIO: Automatización
│   │   │
│   │   └── recurring-expenses/          # Módulo: Gastos recurrentes
│   │       ├── recurring-expenses.module.ts
│   │       ├── recurring-expenses.controller.ts
│   │       ├── recurring-expenses.service.ts
│   │       ├── dto/
│   │       │   ├── create-recurring-expense.dto.ts
│   │       │   ├── update-recurring-expense.dto.ts
│   │       │   ├── generate-expenses.dto.ts
│   │       │   └── reactivate-recurring.dto.ts
│   │       └── tests/
│   │
│   └── reports/                         # DOMINIO: Reportes (futuro)
│       ├── reports.module.ts
│       ├── reports.controller.ts
│       └── reports.service.ts
│
├── test/                                # E2E tests
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
│
├── .env.example                         # Template de variables de entorno
├── .env                                 # Variables de entorno (no commitear)
├── .gitignore
├── package.json
├── tsconfig.json
├── nest-cli.json
└── README.md
```

---

## Descripción de Dominios

### 1. Financial Domain
**Responsabilidad:** Gestión de todo lo relacionado con dinero y finanzas personales.

**Módulos incluidos:**
- **Categories**: Organización jerárquica de gastos
- **Budgets**: Control de límites de gasto
- **Expenses**: Registro de transacciones individuales
- **Expense Types**: Definición de conceptos de gasto

**Razón de agrupación:**
Todos estos módulos trabajan juntos para responder: "¿En qué gasté mi dinero?"

**Interacciones principales:**
- Expense depende de ExpenseType
- ExpenseType depende de Category
- Budget depende de Category
- Expense puede usar datos de Budget para validaciones

---

### 2. Payment Methods Domain
**Responsabilidad:** Gestión de tarjetas y formas de pago.

**Módulos incluidos:**
- **Credit Cards**: Tarjetas de crédito con billing cycles
- **Debit Cards**: Tarjetas de débito y cuentas

**Razón de agrupación:**
Ambos son métodos de pago pero tienen lógicas diferentes (crédito tiene billing period).

**Interacciones principales:**
- Expense usa CreditCard o DebitCard según payment_type
- CreditCard calcula billing_period para Expenses
- RecurringExpense puede usar cualquier tarjeta

**Por qué NO están en Financial:**
Son infraestructura de pago, no el concepto financiero en sí. Un gasto ES financiero, pero CÓMO lo pagas es un método.

---

### 3. Automation Domain
**Responsabilidad:** Automatización de procesos repetitivos.

**Módulos incluidos:**
- **Recurring Expenses**: Generación automática de gastos recurrentes

**Razón de agrupación:**
Todo lo que "se hace solo" o tiene lógica programada.

**Interacciones principales:**
- RecurringExpense genera Expense automáticamente
- Usa ExpenseType como template
- Puede usar CreditCard o DebitCard

**Futuras adiciones al dominio:**
- Scheduled Reports (reportes automáticos)
- Auto-categorization (ML para categorizar gastos)
- Alerts & Notifications

**Por qué NO está en Financial:**
Es un mecanismo de soporte, no la operación financiera en sí.

---

### 4. Reports Domain (Futuro)
**Responsabilidad:** Análisis, reportería y visualización de datos.

**Módulos incluidos:**
- Reports: Agregaciones y análisis
- Analytics: Tendencias y proyecciones (backlog)
- Exports: Exportación de datos (backlog)

**Razón de agrupación:**
Todo lo relacionado con "entender los datos" vs "capturar los datos".

**Interacciones principales:**
- Lee de todos los demás dominios
- No modifica datos (read-only en su mayoría)
- Puede cachear resultados

---

## Módulo Common

### ¿Qué va en Common?
Código que es **verdaderamente compartido** entre múltiples dominios.

**✅ Sí va en Common:**
- Decorators genéricos (@SoftDelete)
- Exception filters (errores HTTP, Prisma)
- Interceptors (logging, transformación)
- Pipes de validación global
- Tipos/enums compartidos (PaymentType, RecurringFrequency)
- Utilidades genéricas (date helpers, formatters)

**❌ No va en Common:**
- Lógica de negocio específica de un dominio
- DTOs específicos de un módulo
- Servicios que solo usa un módulo
- Validaciones específicas de un caso de uso

**Regla de oro:**
Si algo se usa en 2+ dominios diferentes → Common  
Si solo se usa en un dominio → Dentro del dominio

---

## Módulo Prisma

### Responsabilidad
- Conexión con la base de datos
- Cliente de Prisma como singleton
- Gestión de transacciones
- No contiene lógica de negocio

### ¿Por qué separado?
- Prisma es infraestructura, no dominio
- Se inyecta en servicios de todos los dominios
- Facilita testing (mock de PrismaService)
- Configuración centralizada

### Uso en otros módulos
```typescript
// En cualquier service
constructor(private prisma: PrismaService) {}

async findAll() {
  return this.prisma.category.findMany();
}
```

---

## Dependencias entre Módulos

### Reglas de Dependencia

1. **Módulos de dominio NO deben depender entre sí directamente**
   - ❌ `CategoriesService` no importa `ExpenseService`
   - ✅ Ambos usan `PrismaService` independientemente

2. **Comunicación vía base de datos**
   - Las relaciones están en Prisma schema
   - Los servicios consultan via Prisma
   - No hay llamadas directas entre servicios de dominios diferentes

3. **Common puede ser usado por todos**
   - ✅ Cualquier módulo puede importar de `common/`
   - ❌ `common/` no importa de módulos de dominio

4. **Prisma puede ser usado por todos**
   - ✅ Todo módulo puede inyectar `PrismaService`
   - ❌ `PrismaService` no tiene lógica de negocio

### Diagrama de Dependencias

```
┌─────────────────────────────────────────────┐
│              app.module.ts                  │
└─────────────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
┌───────▼───────┐ ┌──▼──────┐ ┌──▼──────────┐
│   Financial   │ │ Payment │ │ Automation  │
│    Domain     │ │ Methods │ │   Domain    │
└───────┬───────┘ └────┬────┘ └──────┬──────┘
        │              │              │
        └──────────────┼──────────────┘
                       │
                  ┌────▼─────┐
                  │  Prisma  │
                  │  Module  │
                  └──────────┘
                       │
                  ┌────▼─────┐
                  │ PostgreSQL│
                  └──────────┘

   ┌─────────────────┐
   │  Common Module  │  ← Usado por todos
   └─────────────────┘
```

---

## Ejemplo: Flujo de Creación de Gasto

Para entender cómo interactúan los módulos:

```typescript
// 1. Request llega al controller
POST /expenses
{
  "expense_type_id": "uuid-arriendo",
  "amount": 400000,
  "date": "2025-07-01",
  "payment_type": "debit",
  "debit_card_id": "uuid-cuenta-rut"
}

// 2. ExpenseController → ExpenseService
@Controller('expenses')
export class ExpensesController {
  async create(@Body() dto: CreateExpenseDto) {
    return this.expensesService.create(dto);
  }
}

// 3. ExpenseService valida y crea
@Injectable()
export class ExpensesService {
  constructor(private prisma: PrismaService) {}
  
  async create(dto: CreateExpenseDto) {
    // Valida que expense_type existe
    const expenseType = await this.prisma.expenseType.findUnique({
      where: { id: dto.expense_type_id }
    });
    
    // Valida que debit_card existe (si aplica)
    if (dto.payment_type === 'debit') {
      const card = await this.prisma.debitCard.findUnique({
        where: { id: dto.debit_card_id }
      });
    }
    
    // Crea el gasto
    return this.prisma.expense.create({
      data: {
        ...dto,
        billing_period: null // no aplica para débito
      },
      include: {
        expenseType: {
          include: { category: true }
        }
      }
    });
  }
}
```

**Nota:** No hay llamadas entre `ExpenseService` y `DebitCardService`. Todo se maneja vía Prisma y las relaciones de BD.

---

## Ventajas de esta Estructura

### 1. Escalabilidad
- Agregar nuevo dominio es fácil (nueva carpeta)
- No "contamina" otros dominios
- Bounded contexts claros

### 2. Mantenibilidad
- Código relacionado está junto
- Fácil de encontrar (busco por dominio, no por tipo técnico)
- Cambios localizados

### 3. Testing
- Módulos auto-contenidos facilitan testing
- Mock de PrismaService es suficiente
- No hay dependencias cruzadas complejas

### 4. Onboarding
- Nuevo dev entiende rápido la estructura
- Dominios son conceptos de negocio, no técnicos
- Navegación intuitiva

### 5. Evolución
- Fácil refactorizar dentro de un dominio
- Fácil mover módulos entre dominios si es necesario
- Preparado para microservicios (cada dominio podría ser un servicio)

---

## Comparación con Alternativas

### vs. Estructura Plana (por entidad)
```
src/
├── categories/
├── budgets/
├── expenses/
├── expense-types/
├── credit-cards/
├── debit-cards/
└── recurring-expenses/
```

**Problemas:**
- No hay agrupación lógica
- Difícil ver relaciones de negocio
- Escala mal (> 20 módulos es caos)

---

### vs. MVC Tradicional
```
src/
├── controllers/
├── services/
├── models/
└── dto/
```

**Problemas:**
- Saltos constantes entre carpetas
- Difícil encontrar código relacionado
- No refleja la arquitectura de negocio

---

### vs. DDD Completo
Con agregados, value objects, domain events, etc.

**Por qué DDD-lite:**
- MVP no requiere complejidad completa de DDD
- Mantiene beneficios de organización por dominio
- Más simple de entender y mantener
- Puede evolucionar a DDD completo si se necesita

---

## Convenciones de Naming

### Archivos
- `*.module.ts`: Módulo NestJS
- `*.controller.ts`: Controller (endpoints)
- `*.service.ts`: Lógica de negocio
- `*.dto.ts`: Data Transfer Objects
- `*.entity.ts`: Opcional, interfaces adicionales
- `*.spec.ts`: Tests unitarios
- `*.e2e-spec.ts`: Tests E2E

### Clases
- `CategoriesModule`
- `CategoriesController`
- `CategoriesService`
- `CreateCategoryDto`
- `UpdateCategoryDto`

### Rutas
- `/categories`
- `/categories/:id`
- `/expenses`
- `/credit-cards/:id/next-payment`

**Regla:** Plural para colecciones, descriptivo para acciones.

---

## Guía de Decisión: ¿Dónde va mi código?

### Pregunta 1: ¿Es específico de un módulo?
- **Sí** → Va dentro del módulo
- **No** → Pregunta 2

### Pregunta 2: ¿Es lógica de negocio compartida?
- **Sí** → Crear módulo compartido en el dominio apropiado
- **No** → Pregunta 3

### Pregunta 3: ¿Es infraestructura técnica?
- **Sí (DB, config, etc)** → Nivel raíz (prisma/, config/)
- **Sí (helpers, utils)** → common/
- **No** → Probablemente debería ser un nuevo módulo

### Ejemplos

**"Necesito un helper para formatear montos en CLP"**
→ `common/utils/currency.helper.ts` (se usa en múltiples módulos)

**"Necesito validar que un gasto no sea negativo"**
→ `financial/expenses/dto/create-expense.dto.ts` (específico de expenses)

**"Necesito calcular billing period de una tarjeta"**
→ `payment-methods/credit-cards/credit-cards.service.ts` (lógica de crédito)

**"Necesito un servicio para generar reportes mensuales"**
→ `reports/reports.service.ts` (nuevo dominio Reports)

---

## Evolución Futura

### Cuando el proyecto crezca:

1. **Más módulos en dominios existentes**
   - financial/transactions
   - financial/investments
   - automation/alerts

2. **Nuevos dominios**
   - users/ (si se hace multi-usuario)
   - notifications/
   - integrations/

3. **Subdominios**
   - Si Financial crece mucho, podría tener subdominios
   - financial/core/
   - financial/analytics/

4. **Microservicios** (muy futuro)
   - Cada dominio podría ser un servicio independiente
   - Esta estructura facilita esa migración

---

## Referencias

- [NestJS Modules](https://docs.nestjs.com/modules)
- [Domain-Driven Design (Eric Evans)](https://www.domainlanguage.com/ddd/)
- [NestJS Architecture Best Practices](https://docs.nestjs.com/fundamentals/custom-providers)
- [Bounded Context (Martin Fowler)](https://martinfowler.com/bliki/BoundedContext.html)