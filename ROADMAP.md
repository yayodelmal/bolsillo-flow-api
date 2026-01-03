# Roadmap de Desarrollo - Bolsillo Flow API POC

## Overview del POC

Este documento define el roadmap de desarrollo para construir un **Proof of Concept (POC)** funcional de Bolsillo Flow API. El objetivo es implementar las funcionalidades core que demuestren el valor del producto de manera incremental y validable.

---

## Estrategia de Desarrollo

### Principios

1. **Incremental:** Cada fase es funcional y deployable
2. **Testeable:** Cada feature tiene tests unitarios y E2E
3. **Validable:** Cada fase demuestra valor tangible
4. **Simple primero:** Implementar lo esencial, diferir lo complejo

### Metodología

- **Desarrollo por dominios:** Completar un dominio antes de pasar al siguiente
- **Tests primero:** Escribir tests antes o durante el desarrollo
- **Documentación inline:** Comentarios claros en el código
- **Migrations incrementales:** Una migration por feature

---

## Fase 0: Setup Inicial (Base del Proyecto)

**Objetivo:** Establecer la estructura base del proyecto funcional.

### 0.1 Inicializar Proyecto NestJS

```bash
nest new bolsillo-flow-api
cd bolsillo-flow-api
```

**Resultado esperado:**
- Proyecto NestJS base
- `npm run start:dev` funciona
- Health endpoint responde

---

### 0.2 Configurar Prisma

```bash
npm install prisma @prisma/client
npx prisma init
```

**Archivos a crear/modificar:**
- `prisma/schema.prisma` - Schema base
- `.env` - Database URL
- `src/prisma/prisma.module.ts` - Módulo de Prisma
- `src/prisma/prisma.service.ts` - Servicio de Prisma

**Resultado esperado:**
- Prisma conecta a PostgreSQL
- `npx prisma studio` abre correctamente

---

### 0.3 Configurar Estructura de Carpetas

**Crear estructura DDD-lite:**

```
src/
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── interceptors/
│   └── pipes/
├── financial/
│   └── (módulos se agregarán aquí)
├── payment-methods/
│   └── (módulos se agregarán aquí)
├── automation/
│   └── (módulos se agregarán aquí)
├── prisma/
│   ├── prisma.module.ts
│   └── prisma.service.ts
└── main.ts
```

**Resultado esperado:**
- Estructura de carpetas creada
- Módulos base registrados en `app.module.ts`

---

### 0.4 Configurar Testing

```bash
npm install --save-dev @nestjs/testing
```

**Archivos a configurar:**
- `jest.config.js` - Configuración de Jest
- `test/app.e2e-spec.ts` - Test E2E base

**Resultado esperado:**
- `npm run test` ejecuta tests
- `npm run test:e2e` ejecuta E2E tests

---

## Fase 1: Categorías (Financial Domain - Base)

**Objetivo:** CRUD completo de categorías con jerarquía de 2 niveles.

### 1.1 Definir Schema de Categorías

**Archivo:** `prisma/schema.prisma`

```prisma
model Category {
  id          String    @id @default(uuid())
  name        String
  description String?
  color       String?
  parentId    String?   @map("parent_id")
  isActive    Boolean   @default(true) @map("is_active")
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")
  deletedAt   DateTime? @map("deleted_at")

  parent   Category?  @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children Category[] @relation("CategoryHierarchy")

  @@map("categories")
}
```

**Comandos:**
```bash
npx prisma migrate dev --name create_categories
npx prisma generate
```

---

### 1.2 Crear Módulo de Categorías

**Generar con NestJS CLI:**
```bash
nest g module financial/categories
nest g controller financial/categories
nest g service financial/categories
```

**Archivos creados:**
- `src/financial/categories/categories.module.ts`
- `src/financial/categories/categories.controller.ts`
- `src/financial/categories/categories.service.ts`

---

### 1.3 Implementar DTOs

**Archivos a crear:**
- `src/financial/categories/dto/create-category.dto.ts`
- `src/financial/categories/dto/update-category.dto.ts`

**DTOs con validación:**

```typescript
// create-category.dto.ts
import { IsString, IsOptional, IsUUID, MinLength, MaxLength, Matches } from 'class-validator';

export class CreateCategoryDto {
  @IsString()
  @MinLength(3)
  @MaxLength(50)
  name: string;

  @IsOptional()
  @IsString()
  @MaxLength(200)
  description?: string;

  @IsOptional()
  @IsString()
  @Matches(/^#[0-9A-F]{6}$/i, { message: 'Color must be a valid hex color' })
  color?: string;

  @IsOptional()
  @IsUUID()
  parentId?: string;
}
```

---

### 1.4 Implementar Service (Lógica de Negocio)

**Métodos a implementar en `categories.service.ts`:**

1. `findAll(includeDeleted = false)` - Listar todas
2. `findOne(id, includeDeleted = false)` - Obtener una
3. `create(dto)` - Crear categoría
4. `update(id, dto)` - Actualizar categoría
5. `remove(id)` - Soft delete
6. `restore(id)` - Restaurar soft deleted

**Validaciones de negocio a implementar:**
- ✅ Solo 2 niveles de jerarquía (BR-CAT-001)
- ✅ No puede tener padre si tiene hijos (BR-CAT-005)
- ✅ Validar que padre existe
- ✅ Cascade delete de hijos (BR-CAT-002)
- ✅ Solo restaurar si padre está activo (BR-CAT-003)

---

### 1.5 Implementar Controller (Endpoints)

**Endpoints a implementar:**

```typescript
@Controller('categories')
export class CategoriesController {
  @Get()
  findAll(@Query('includeDeleted') includeDeleted?: boolean) {}

  @Get(':id')
  findOne(@Param('id') id: string) {}

  @Post()
  create(@Body() dto: CreateCategoryDto) {}

  @Put(':id')
  update(@Param('id') id: string, @Body() dto: UpdateCategoryDto) {}

  @Patch(':id')
  partialUpdate(@Param('id') id: string, @Body() dto: Partial<UpdateCategoryDto>) {}

  @Delete(':id')
  remove(@Param('id') id: string) {}

  @Patch(':id/restore')
  restore(@Param('id') id: string) {}

  @Get(':id/children')
  getChildren(@Param('id') id: string) {}
}
```

---

### 1.6 Tests de Categorías

**Tests unitarios:** `categories.service.spec.ts`
- ✅ Crear categoría padre
- ✅ Crear categoría hija
- ✅ No permitir 3 niveles
- ✅ Soft delete con cascade
- ✅ Restaurar categoría

**Tests E2E:** `categories.e2e-spec.ts`
- ✅ GET /categories
- ✅ POST /categories
- ✅ GET /categories/:id
- ✅ PUT /categories/:id
- ✅ DELETE /categories/:id

---

### 1.7 Seed de Categorías

**Archivo:** `prisma/seed.ts`

```typescript
const categories = [
  { name: 'Departamento', color: '#3498DB' },
  { name: 'Transporte', color: '#E74C3C' },
  { name: 'Alimentación', color: '#2ECC71' },
  { name: 'Suscripciones', color: '#9B59B6' },
];
```

**Ejecutar:**
```bash
npx prisma db seed
```

---

**✅ Criterio de Completitud Fase 1:**
- [ ] Todas las migraciones aplicadas
- [ ] CRUD completo funcional
- [ ] Validaciones de negocio implementadas
- [ ] Tests unitarios pasan (>80% coverage)
- [ ] Tests E2E pasan
- [ ] Seeds funcionan
- [ ] Documentación inline clara

---

## Fase 2: Tarjetas de Pago (Payment Methods Domain)

**Objetivo:** Gestión de tarjetas de crédito y débito.

### 2.1 Schema de Payment Methods

**Agregar a `prisma/schema.prisma`:**

```prisma
model CreditCard {
  id             String    @id @default(uuid())
  name           String
  bank           String
  lastFourDigits String?   @map("last_four_digits")
  cutoffDay      Int       @map("cutoff_day")
  paymentDueDay  Int       @map("payment_due_day")
  cupo           Decimal?  @db.Decimal(12, 2)
  isActive       Boolean   @default(true) @map("is_active")
  createdAt      DateTime  @default(now()) @map("created_at")
  updatedAt      DateTime  @updatedAt @map("updated_at")
  deletedAt      DateTime? @map("deleted_at")

  @@map("credit_cards")
}

model DebitCard {
  id             String    @id @default(uuid())
  name           String
  bank           String
  currentBalance Decimal?  @db.Decimal(12, 2) @map("current_balance")
  isActive       Boolean   @default(true) @map("is_active")
  createdAt      DateTime  @default(now()) @map("created_at")
  updatedAt      DateTime  @updatedAt @map("updated_at")
  deletedAt      DateTime? @map("deleted_at")

  @@map("debit_cards")
}
```

**Migración:**
```bash
npx prisma migrate dev --name create_payment_methods
```

---

### 2.2 Módulos de Tarjetas

```bash
nest g module payment-methods/credit-cards
nest g controller payment-methods/credit-cards
nest g service payment-methods/credit-cards

nest g module payment-methods/debit-cards
nest g controller payment-methods/debit-cards
nest g service payment-methods/debit-cards
```

---

### 2.3 Implementar Credit Cards

**DTOs:**
- `CreateCreditCardDto`
- `UpdateCreditCardDto`

**Validaciones:**
- `cutoffDay` entre 1-31
- `paymentDueDay` entre 1-31
- `cupo` > 0 si se proporciona

**Endpoints:**
- GET /credit-cards
- POST /credit-cards
- PUT /credit-cards/:id
- DELETE /credit-cards/:id

---

### 2.4 Implementar Debit Cards

**Similar a Credit Cards pero más simple (no tiene cutoff/payment days)**

---

### 2.5 Seeds de Tarjetas

```typescript
const creditCards = [
  { name: 'BCI Crédito', bank: 'BCI', cutoffDay: 15, paymentDueDay: 5 },
];

const debitCards = [
  { name: 'Cuenta RUT', bank: 'Banco Estado' },
  { name: 'BCI Débito', bank: 'BCI' },
];
```

---

**✅ Criterio de Completitud Fase 2:**
- [ ] Schema de tarjetas migrado
- [ ] CRUD de credit cards funcional
- [ ] CRUD de debit cards funcional
- [ ] Validaciones implementadas
- [ ] Tests pasan
- [ ] Seeds funcionan

---

## Fase 3: Expense Types (Financial Domain)

**Objetivo:** Gestión de tipos de gasto con soporte para cuotas y recurrentes.

### 3.1 Schema de Expense Types

```prisma
model ExpenseType {
  id                String    @id @default(uuid())
  name              String
  categoryId        String    @map("category_id")
  isRecurring       Boolean   @default(false) @map("is_recurring")
  isInstallment     Boolean   @default(false) @map("is_installment")
  installmentConfig Json?     @map("installment_config")
  isCompleted       Boolean   @default(false) @map("is_completed")
  completedAt       DateTime? @map("completed_at")
  isActive          Boolean   @default(true) @map("is_active")
  createdAt         DateTime  @default(now()) @map("created_at")
  updatedAt         DateTime  @updatedAt @map("updated_at")
  deletedAt         DateTime? @map("deleted_at")

  category Category @relation(fields: [categoryId], references: [id])

  @@map("expense_types")
}
```

**Migración:**
```bash
npx prisma migrate dev --name create_expense_types
```

---

### 3.2 Módulo de Expense Types

```bash
nest g module financial/expense-types
nest g controller financial/expense-types
nest g service financial/expense-types
```

---

### 3.3 Implementar DTOs

**InstallmentConfig Interface:**
```typescript
interface InstallmentConfig {
  totalInstallments: number;
  amountPerInstallment: number;
  currentInstallment: number;
  remainingInstallments: number;
  startDate: string;
}
```

**CreateExpenseTypeDto:**
```typescript
export class CreateExpenseTypeDto {
  @IsString()
  name: string;

  @IsUUID()
  categoryId: string;

  @IsBoolean()
  @IsOptional()
  isRecurring?: boolean;

  @IsBoolean()
  @IsOptional()
  isInstallment?: boolean;

  @IsOptional()
  @ValidateNested()
  installmentConfig?: InstallmentConfig;
}
```

---

### 3.4 Validaciones de Negocio

- ✅ Si `isInstallment = true`, `installmentConfig` es obligatorio
- ✅ Auto-calcular `currentInstallment = 0` y `remainingInstallments = totalInstallments`
- ✅ Categoría debe existir y estar activa
- ✅ No permitir eliminar si tiene expenses asociados (se agregará en Fase 4)

---

### 3.5 Seeds de Expense Types

```typescript
const expenseTypes = [
  { name: 'Arriendo', categoryId: 'departamento-id', isRecurring: true },
  { name: 'Luz', categoryId: 'departamento-id' },
  { name: 'Uber', categoryId: 'transporte-id' },
];
```

---

**✅ Criterio de Completitud Fase 3:**
- [ ] Schema migrado con relación a categories
- [ ] CRUD completo
- [ ] Validación de installmentConfig
- [ ] Tests pasan
- [ ] Seeds funcionan

---

## Fase 4: Expenses (Core del Sistema)

**Objetivo:** Registro de gastos con cálculo automático de billing period.

### 4.1 Schema de Expenses

```prisma
model Expense {
  id               String    @id @default(uuid())
  expenseTypeId    String    @map("expense_type_id")
  amount           Decimal   @db.Decimal(12, 2)
  date             DateTime
  paymentType      String    @map("payment_type") // 'credit', 'debit', 'transfer', 'cash'
  creditCardId     String?   @map("credit_card_id")
  debitCardId      String?   @map("debit_card_id")
  billingPeriod    String?   @map("billing_period") // 'YYYY-MM'
  isRecurring      Boolean   @default(false) @map("is_recurring")
  installmentNumber String?  @map("installment_number") // 'N/M'
  notes            String?
  createdAt        DateTime  @default(now()) @map("created_at")
  updatedAt        DateTime  @updatedAt @map("updated_at")
  deletedAt        DateTime? @map("deleted_at")

  expenseType ExpenseType @relation(fields: [expenseTypeId], references: [id])
  creditCard  CreditCard? @relation(fields: [creditCardId], references: [id])
  debitCard   DebitCard?  @relation(fields: [debitCardId], references: [id])

  @@index([date])
  @@index([billingPeriod])
  @@index([expenseTypeId])
  @@map("expenses")
}
```

**Actualizar relaciones en otros modelos:**
- Category → ExpenseType (1:N)
- ExpenseType → Expense (1:N)
- CreditCard → Expense (1:N)
- DebitCard → Expense (1:N)

**Migración:**
```bash
npx prisma migrate dev --name create_expenses
```

---

### 4.2 Módulo de Expenses

```bash
nest g module financial/expenses
nest g controller financial/expenses
nest g service financial/expenses
```

---

### 4.3 Implementar Billing Cycle Calculator

**Archivo:** `src/financial/expenses/utils/billing-cycle.calculator.ts`

```typescript
export class BillingCycleCalculator {
  static calculateBillingPeriod(transactionDate: Date, cutoffDay: number): string {
    const day = transactionDate.getDate();
    const month = transactionDate.getMonth();
    const year = transactionDate.getFullYear();

    if (day <= cutoffDay) {
      return `${year}-${String(month + 1).padStart(2, '0')}`;
    } else {
      const nextMonth = new Date(year, month + 1, 1);
      return `${nextMonth.getFullYear()}-${String(nextMonth.getMonth() + 1).padStart(2, '0')}`;
    }
  }

  static calculatePaymentDueDate(billingPeriod: string, paymentDueDay: number): Date {
    const [year, month] = billingPeriod.split('-').map(Number);
    let paymentMonth = month + 1;
    let paymentYear = year;

    if (paymentMonth > 12) {
      paymentMonth = 1;
      paymentYear++;
    }

    return new Date(paymentYear, paymentMonth - 1, paymentDueDay);
  }
}
```

**Tests unitarios del calculator son CRÍTICOS.**

---

### 4.4 Implementar DTOs

```typescript
export class CreateExpenseDto {
  @IsUUID()
  expenseTypeId: string;

  @IsNumber()
  @Min(0)
  amount: number;

  @IsDateString()
  date: string;

  @IsIn(['credit', 'debit', 'transfer', 'cash'])
  paymentType: string;

  @IsUUID()
  @IsOptional()
  creditCardId?: string;

  @IsUUID()
  @IsOptional()
  debitCardId?: string;

  @IsString()
  @IsOptional()
  @Matches(/^\d+\/\d+$/)
  installmentNumber?: string;

  @IsString()
  @IsOptional()
  notes?: string;
}
```

---

### 4.5 Implementar Service

**Método crítico: `create(dto)`**

```typescript
async create(dto: CreateExpenseDto) {
  // 1. Validar expense type existe
  const expenseType = await this.prisma.expenseType.findUnique({
    where: { id: dto.expenseTypeId }
  });

  // 2. Si es crédito, calcular billing period
  let billingPeriod = null;
  if (dto.paymentType === 'credit' && dto.creditCardId) {
    const card = await this.prisma.creditCard.findUnique({
      where: { id: dto.creditCardId }
    });
    
    billingPeriod = BillingCycleCalculator.calculateBillingPeriod(
      new Date(dto.date),
      card.cutoffDay
    );
  }

  // 3. Crear expense
  const expense = await this.prisma.expense.create({
    data: {
      ...dto,
      billingPeriod,
      amount: new Decimal(dto.amount),
      date: new Date(dto.date)
    }
  });

  // 4. Si es cuota, actualizar expense type
  if (dto.installmentNumber) {
    await this.updateInstallmentProgress(expenseType, dto.installmentNumber);
  }

  return expense;
}
```

---

### 4.6 Endpoints Básicos

**Para el POC, implementar:**
- POST /expenses - Crear gasto
- GET /expenses - Listar con filtros (month, categoryId, paymentType)
- GET /expenses/:id - Obtener uno
- PUT /expenses/:id - Actualizar
- DELETE /expenses/:id - Eliminar

**Diferir para después del POC:**
- GET /expenses/summary
- GET /expenses/monthly-report
- GET /expenses/annual-installments

---

### 4.7 Seeds de Expenses

```typescript
const expenses = [
  {
    expenseTypeId: 'arriendo-id',
    amount: 400000,
    date: new Date('2025-07-01'),
    paymentType: 'debit',
    debitCardId: 'cuenta-rut-id',
  },
  {
    expenseTypeId: 'uber-id',
    amount: 15000,
    date: new Date('2025-07-18'),
    paymentType: 'credit',
    creditCardId: 'bci-credito-id',
  },
];
```

---

**✅ Criterio de Completitud Fase 4:**
- [ ] Schema completo con relaciones
- [ ] Billing cycle calculator con tests
- [ ] Crear expense funciona
- [ ] Cálculo automático de billing period
- [ ] Filtros básicos funcionan
- [ ] Tests unitarios y E2E pasan
- [ ] Seeds funcionan

---

## Fase 5: Budgets (Control de Gastos)

**Objetivo:** Definir y consultar presupuestos mensuales.

### 5.1 Schema de Budgets

```prisma
model Budget {
  id         String   @id @default(uuid())
  categoryId String   @map("category_id")
  amount     Decimal  @db.Decimal(12, 2)
  period     String   // 'YYYY-MM'
  createdAt  DateTime @default(now()) @map("created_at")
  updatedAt  DateTime @updatedAt @map("updated_at")

  category Category @relation(fields: [categoryId], references: [id])

  @@unique([categoryId, period])
  @@index([period])
  @@map("budgets")
}
```

**Migración:**
```bash
npx prisma migrate dev --name create_budgets
```

---

### 5.2 Módulo de Budgets

```bash
nest g module financial/budgets
nest g controller financial/budgets
nest g service financial/budgets
```

---

### 5.3 Implementar Service

**Métodos clave:**

```typescript
async createOrUpdate(dto: CreateBudgetDto) {
  // Upsert
  return this.prisma.budget.upsert({
    where: {
      categoryId_period: {
        categoryId: dto.categoryId,
        period: dto.period
      }
    },
    update: { amount: dto.amount },
    create: dto
  });
}

async getBudgetStatus(categoryId: string, period: string) {
  const budget = await this.findOne(categoryId, period);
  
  if (!budget) {
    return { budgetDefined: false };
  }

  // Calcular gasto total del período
  const totalSpent = await this.calculateTotalSpent(categoryId, period);

  return {
    budget: budget.amount,
    totalSpent,
    remaining: budget.amount - totalSpent,
    percentageUsed: (totalSpent / budget.amount) * 100,
    overBudget: totalSpent > budget.amount
  };
}
```

---

### 5.4 Endpoints

- POST /budgets - Crear/actualizar (upsert)
- GET /budgets?period=YYYY-MM - Listar del período
- GET /budgets/category/:categoryId/period/:period - Obtener específico
- GET /budgets/category/:categoryId/status?period=YYYY-MM - Status

---

### 5.5 Seeds de Budgets

```typescript
const budgets = [
  { categoryId: 'departamento-id', amount: 600000, period: '2025-07' },
  { categoryId: 'transporte-id', amount: 150000, period: '2025-07' },
  { categoryId: 'alimentacion-id', amount: 200000, period: '2025-07' },
];
```

---

**✅ Criterio de Completitud Fase 5:**
- [ ] Schema migrado
- [ ] Upsert funciona
- [ ] Cálculo de budget status correcto
- [ ] Tests pasan
- [ ] Seeds funcionan

---

## Fase 6: Recurring Expenses (Automatización)

**Objetivo:** Configurar y generar gastos recurrentes.

### 6.1 Schema de Recurring Expenses

```prisma
model RecurringExpense {
  id                    String    @id @default(uuid())
  expenseTypeId         String    @unique @map("expense_type_id")
  amount                Decimal   @db.Decimal(12, 2)
  dayOfMonth            Int       @map("day_of_month")
  paymentType           String    @map("payment_type")
  creditCardId          String?   @map("credit_card_id")
  debitCardId           String?   @map("debit_card_id")
  frequency             String    @default("monthly") // 'monthly' | 'annual'
  isFixedAmount         Boolean   @default(true) @map("is_fixed_amount")
  usePreviousMonthAmount Boolean  @default(false) @map("use_previous_month_amount")
  isActive              Boolean   @default(true) @map("is_active")
  lastGeneratedDate     DateTime? @map("last_generated_date")
  createdAt             DateTime  @default(now()) @map("created_at")
  updatedAt             DateTime  @updatedAt @map("updated_at")

  expenseType ExpenseType @relation(fields: [expenseTypeId], references: [id])
  creditCard  CreditCard? @relation(fields: [creditCardId], references: [id])
  debitCard   DebitCard?  @relation(fields: [debitCardId], references: [id])

  @@map("recurring_expenses")
}
```

**Migración:**
```bash
npx prisma migrate dev --name create_recurring_expenses
```

---

### 6.2 Módulo de Recurring Expenses

```bash
nest g module automation/recurring-expenses
nest g controller automation/recurring-expenses
nest g service automation/recurring-expenses
```

---

### 6.3 Implementar Service

**Método crítico: `generateMonthlyExpenses(year, month)`**

```typescript
async generateMonthlyExpenses(year: number, month: number) {
  const recurrings = await this.prisma.recurringExpense.findMany({
    where: { isActive: true, frequency: 'monthly' }
  });

  const results = [];

  for (const rec of recurrings) {
    // Verificar si ya fue generado
    const alreadyGenerated = await this.wasAlreadyGenerated(rec, year, month);
    if (alreadyGenerated) {
      results.push({ ...rec, status: 'skipped' });
      continue;
    }

    // Determinar monto
    let amount = rec.amount;
    if (!rec.isFixedAmount && rec.usePreviousMonthAmount) {
      amount = await this.getPreviousMonthAmount(rec.expenseTypeId, year, month);
    }

    // Crear expense
    const expense = await this.expensesService.create({
      expenseTypeId: rec.expenseTypeId,
      amount,
      date: new Date(year, month - 1, rec.dayOfMonth),
      paymentType: rec.paymentType,
      creditCardId: rec.creditCardId,
      debitCardId: rec.debitCardId,
      isRecurring: true
    });

    // Actualizar last generated
    await this.prisma.recurringExpense.update({
      where: { id: rec.id },
      data: { lastGeneratedDate: new Date() }
    });

    results.push({ ...rec, status: 'generated', expenseId: expense.id });
  }

  return results;
}
```

---

### 6.4 Endpoints

- GET /recurring-expenses - Listar
- POST /recurring-expenses - Crear config
- PUT /recurring-expenses/:id - Actualizar
- DELETE /recurring-expenses/:id - Eliminar
- PATCH /recurring-expenses/:id/pause - Pausar
- PATCH /recurring-expenses/:id/reactivate - Reactivar
- POST /recurring-expenses/generate - Generar gastos (año, mes)

---

### 6.5 Seeds de Recurring Expenses

```typescript
const recurrings = [
  {
    expenseTypeId: 'arriendo-id',
    amount: 400000,
    dayOfMonth: 1,
    paymentType: 'debit',
    debitCardId: 'cuenta-rut-id',
    isFixedAmount: true
  },
];
```

---

**✅ Criterio de Completitud Fase 6:**
- [ ] Schema migrado
- [ ] CRUD completo
- [ ] Generación mensual funciona
- [ ] Previene duplicados
- [ ] Usa monto anterior si es variable
- [ ] Tests pasan

---

## Fase 7: Polish & Testing (Final del POC)

**Objetivo:** Pulir detalles, completar tests y preparar para demo.

### 7.1 Global Exception Filters

**Archivo:** `src/common/filters/http-exception.filter.ts`

Manejo centralizado de errores con responses consistentes.

---

### 7.2 Validation Pipe Global

**En `main.ts`:**
```typescript
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true
}));
```

---

### 7.3 Completar Coverage de Tests

**Target: >80% coverage**

```bash
npm run test:cov
```

Asegurar cobertura en:
- Services (lógica de negocio)
- DTOs (validaciones)
- Calculators (billing cycle)

---

### 7.4 E2E Tests Completos

**Archivo:** `test/app.e2e-spec.ts`

Flujos completos end-to-end:
- ✅ Crear categoría → expense type → expense
- ✅ Crear presupuesto → verificar status
- ✅ Crear recurrente → generar expenses
- ✅ Calcular billing period correcto

---

### 7.5 Seeds Completos y Realistas

**Crear escenario completo:**
- 4 categorías padre con 2-3 hijas cada una
- 3 tarjetas (2 crédito, 1 débito)
- 15+ expense types
- 50+ expenses (julio 2025)
- 4 presupuestos
- 10 recurrentes

---

### 7.6 Health & Info Endpoints

```typescript
@Get('health')
async health() {
  return {
    status: 'ok',
    timestamp: new Date(),
    database: await this.prisma.$queryRaw`SELECT 1`
  };
}

@Get('api-info')
async info() {
  return {
    name: 'Bolsillo Flow API',
    version: '1.0.0',
    environment: process.env.NODE_ENV
  };
}
```

---

### 7.7 README del Proyecto

Actualizar README.md con:
- ✅ Instrucciones de setup
- ✅ Comandos principales
- ✅ Endpoints disponibles
- ✅ Estructura del proyecto

---

**✅ Criterio de Completitud Fase 7:**
- [ ] Exception filters implementados
- [ ] Validation pipe global
- [ ] >80% test coverage
- [ ] E2E tests de flujos completos
- [ ] Seeds completos y realistas
- [ ] Health endpoints
- [ ] README completo

---

## Criterios de Éxito del POC

### Funcional
- [x] Usuario puede crear categorías jerárquicas
- [x] Usuario puede registrar gastos
- [x] Sistema calcula automáticamente período de facturación
- [x] Usuario puede definir presupuestos y ver status
- [x] Usuario puede configurar gastos recurrentes
- [x] Sistema puede generar gastos recurrentes

### Técnico
- [x] Todas las migraciones aplicadas
- [x] Tests >80% coverage
- [x] E2E tests pasan
- [x] Seeds funcionan
- [x] API deployable
- [x] Documentación completa

### Demo
- [x] Crear estructura de categorías
- [x] Registrar 5+ gastos de diferentes tipos
- [x] Mostrar cálculo de billing period
- [x] Definir presupuestos y ver status
- [x] Crear recurrente y generar expenses
- [x] Mostrar que gastos variables usan monto anterior

---

## Comandos Útiles Durante Desarrollo

```bash
# Desarrollo
npm run start:dev

# Tests
npm run test                  # Unit tests
npm run test:watch           # Watch mode
npm run test:cov             # Coverage
npm run test:e2e             # E2E tests

# Database
npx prisma migrate dev       # Crear y aplicar migración
npx prisma generate          # Generar Prisma Client
npx prisma studio            # GUI de base de datos
npx prisma db seed           # Ejecutar seeds
npx prisma migrate reset     # Reset completo (⚠️ borra datos)

# Generar código
nest g module <name>         # Nuevo módulo
nest g controller <name>     # Nuevo controller
nest g service <name>        # Nuevo service
```

---

## Estructura de Commits

**Seguir convención:**
```
Add: nueva funcionalidad
Fix: corrección de bug
Update: actualización
Refactor: refactorización
Test: tests
Docs: documentación
```

**Ejemplo:**
```bash
git commit -m "Add: categories CRUD with hierarchy validation"
git commit -m "Fix: billing cycle calculation for months with 30 days"
git commit -m "Test: add E2E tests for expenses"
```

---

## Notas Importantes para Claude Code

### Orden de Implementación

**Siempre en este orden:**
1. Schema (Prisma)
2. Migration
3. Generate Prisma Client
4. DTOs
5. Service (lógica)
6. Controller (endpoints)
7. Tests
8. Seeds

### No Saltarse Tests

**Cada feature DEBE tener:**
- Unit tests del service
- Tests de validación de DTOs
- E2E test de al menos un happy path

### Validaciones de Negocio

**Implementar en el Service, NO en el controller:**
```typescript
// ✅ CORRECTO
async create(dto: CreateCategoryDto) {
  if (dto.parentId) {
    const parent = await this.validateParent(dto.parentId);
    if (parent.parentId) {
      throw new UnprocessableEntityException('Cannot exceed 2 levels');
    }
  }
}

// ❌ INCORRECTO
@Post()
create(@Body() dto: CreateCategoryDto) {
  if (dto.parentId) { ... } // Validación en controller
}
```

### Prisma Best Practices

**Siempre incluir relaciones necesarias:**
```typescript
// ✅ CORRECTO
const expense = await this.prisma.expense.findUnique({
  where: { id },
  include: {
    expenseType: {
      include: {
        category: true
      }
    },
    creditCard: true
  }
});

// ❌ INCORRECTO - Requiere múltiples queries
const expense = await this.prisma.expense.findUnique({ where: { id } });
const expenseType = await this.prisma.expenseType.findUnique({ where: { id: expense.expenseTypeId } });
```

---

## Recursos de Referencia

Durante el desarrollo, consultar:

1. **Documentación del proyecto:** `/docs`
   - Business Rules: `/docs/03-data-model/business-rules.md`
   - Workflows: `/docs/05-workflows/`
   - Endpoints: `/docs/04-api/endpoints.md`

2. **Documentación externa:**
   - [NestJS Docs](https://docs.nestjs.com/)
   - [Prisma Docs](https://www.prisma.io/docs)
   - [Class Validator](https://github.com/typestack/class-validator)

3. **Ejemplos en el proyecto:**
   - `/docs/04-api/examples/categories.md`

---

## Próximos Pasos Post-POC

**NO implementar durante el POC, diferir para después:**

- [ ] Autenticación (JWT)
- [ ] Multi-usuario
- [ ] Reportes avanzados (summary, annual installments)
- [ ] Exportación de datos
- [ ] Rate limiting
- [ ] Caching
- [ ] Webhooks
- [ ] Notificaciones
- [ ] Dashboard web

**Enfoque del POC:** Core funcional, bien testeado, deployable.
