# Relaciones entre Entidades

## Overview
Este documento describe todas las relaciones entre las entidades del sistema Bolsillo Flow API, incluyendo cardinalidad, tipo de relación, y reglas de integridad referencial.

---

## Diagrama de Relaciones General

```
                    ┌─────────────┐
                    │  Category   │
                    │  (parent)   │
                    └──────┬──────┘
                           │ 1
                           │
                           │ self-reference
                           │
                           │ N
                    ┌──────▼──────┐
                    │  Category   │
                    │  (children) │
                    └──────┬──────┘
                           │ 1
              ┌────────────┼────────────┐
              │            │            │
              │ N          │ N          │ N
       ┌──────▼──────┐ ┌──▼────────┐ ┌─▼──────────┐
       │   Budget    │ │ExpenseType│ │   (future) │
       └─────────────┘ └─────┬─────┘ └────────────┘
                             │ 1
                             │
                             │ N
                      ┌──────▼──────┐
                      │   Expense   │
                      └──────┬──────┘
                             │ N
              ┌──────────────┼──────────────┐
              │              │              │
              │ 1            │ 1            │ 1
       ┌──────▼──────┐ ┌─────▼─────┐ ┌─────▼──────┐
       │ CreditCard  │ │ DebitCard │ │RecurringExp│
       └─────────────┘ └───────────┘ └────────────┘
                                           │ 1
                                           │
                                           │ 1
                                     ┌─────▼─────┐
                                     │ExpenseType│
                                     └───────────┘
```

---

## Relaciones Detalladas

### 1. Category ↔ Category (Self-Reference)

**Tipo:** One-to-Many (auto-referencial)  
**Cardinalidad:** 1:N  
**Descripción:** Una categoría padre puede tener múltiples hijos, pero un hijo solo tiene un padre.

#### Schema Prisma:
```prisma
model Category {
  id       String     @id @default(uuid())
  parentId String?    @map("parent_id")
  
  // Relations
  parent   Category?  @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children Category[] @relation("CategoryHierarchy")
}
```

#### Características:
- **Nullable:** `parentId` es opcional (null = categoría padre)
- **Recursiva:** Categoría se relaciona consigo misma
- **Nombre de relación:** "CategoryHierarchy" (requerido por Prisma para self-references)
- **Cascada:** Al soft-delete padre, se marcan deleted sus hijos

#### Reglas de Integridad:
1. **RI-CAT-001:** `parentId` debe referenciar a un `id` de Category existente
2. **RI-CAT-002:** Máximo 2 niveles (padre → hijo, no nieto)
3. **RI-CAT-003:** No puede haber ciclos (A padre de B, B padre de A)
4. **RI-CAT-004:** Si se elimina padre, se eliminan (soft-delete) hijos en cascada

#### Queries Comunes:
```typescript
// Obtener categoría con sus hijos
await prisma.category.findUnique({
  where: { id: categoryId },
  include: { children: true }
});

// Obtener categoría con su padre
await prisma.category.findUnique({
  where: { id: categoryId },
  include: { parent: true }
});

// Todas las categorías padre (sin parent_id)
await prisma.category.findMany({
  where: { parentId: null }
});

// Todos los hijos de una categoría
await prisma.category.findMany({
  where: { parentId: parentCategoryId }
});
```

---

### 2. Category → Budget

**Tipo:** One-to-Many  
**Cardinalidad:** 1:N  
**Descripción:** Una categoría puede tener múltiples presupuestos (diferentes meses), pero un presupuesto pertenece a una sola categoría.

#### Schema Prisma:
```prisma
model Category {
  id      String   @id @default(uuid())
  budgets Budget[]
}

model Budget {
  id         String   @id @default(uuid())
  categoryId String   @map("category_id")
  period     String   // YYYY-MM
  
  category   Category @relation(fields: [categoryId], references: [id])
  
  @@unique([categoryId, period])
}
```

#### Características:
- **Obligatorio:** Todo presupuesto debe tener categoría
- **Constraint único:** (categoryId, period) - solo un presupuesto por categoría por mes
- **Solo padre:** Los presupuestos solo se crean en categorías padre (validado en lógica, no en BD)

#### Reglas de Integridad:
1. **RI-BUD-001:** `categoryId` debe existir y ser categoría activa
2. **RI-BUD-002:** `categoryId` debe ser de categoría padre (parentId = null)
3. **RI-BUD-003:** Combinación (categoryId, period) debe ser única
4. **RI-BUD-004:** No se permite eliminar categoría con presupuestos (soft-delete ok)

#### Queries Comunes:
```typescript
// Presupuestos de una categoría
await prisma.budget.findMany({
  where: { categoryId: categoryId },
  orderBy: { period: 'desc' }
});

// Presupuesto específico de categoría en mes
await prisma.budget.findUnique({
  where: {
    categoryId_period: {
      categoryId: categoryId,
      period: '2025-07'
    }
  }
});

// Todas las categorías con sus presupuestos del mes actual
await prisma.category.findMany({
  include: {
    budgets: {
      where: { period: '2025-07' }
    }
  }
});
```

---

### 3. Category → ExpenseType

**Tipo:** One-to-Many  
**Cardinalidad:** 1:N  
**Descripción:** Una categoría puede tener múltiples tipos de gasto, pero un tipo de gasto pertenece a una sola categoría.

#### Schema Prisma:
```prisma
model Category {
  id           String        @id @default(uuid())
  expenseTypes ExpenseType[]
}

model ExpenseType {
  id         String   @id @default(uuid())
  name       String
  categoryId String   @map("category_id")
  
  category   Category @relation(fields: [categoryId], references: [id])
}
```

#### Características:
- **Obligatorio:** Todo tipo de gasto debe tener categoría
- **Inmutable (recomendado):** No se recomienda cambiar categoryId de tipos existentes
- **Puede ser hijo:** ExpenseTypes pueden pertenecer a categorías hijas o padres

#### Reglas de Integridad:
1. **RI-ET-001:** `categoryId` debe existir y estar activo
2. **RI-ET-002:** No se permite eliminar categoría con expense types activos
3. **RI-ET-003:** Al cambiar categoría de un expense type, todos sus gastos históricos mantienen la asociación original

#### Queries Comunes:
```typescript
// Tipos de gasto de una categoría
await prisma.expenseType.findMany({
  where: { 
    categoryId: categoryId,
    isActive: true,
    deletedAt: null
  }
});

// Categoría con sus tipos de gasto
await prisma.category.findUnique({
  where: { id: categoryId },
  include: {
    expenseTypes: {
      where: { isActive: true }
    }
  }
});

// Tipos de gasto por categoría padre (incluye hijos)
await prisma.expenseType.findMany({
  where: {
    category: {
      OR: [
        { id: parentCategoryId },
        { parentId: parentCategoryId }
      ]
    }
  }
});
```

---

### 4. ExpenseType → Expense

**Tipo:** One-to-Many  
**Cardinalidad:** 1:N  
**Descripción:** Un tipo de gasto puede tener múltiples gastos, pero un gasto pertenece a un solo tipo.

#### Schema Prisma:
```prisma
model ExpenseType {
  id       String    @id @default(uuid())
  name     String
  expenses Expense[]
}

model Expense {
  id            String   @id @default(uuid())
  expenseTypeId String   @map("expense_type_id")
  amount        Decimal  @db.Decimal(12, 2)
  date          DateTime
  
  expenseType   ExpenseType @relation(fields: [expenseTypeId], references: [id])
}
```

#### Características:
- **Obligatorio:** Todo gasto debe tener tipo
- **Inmutable:** Una vez creado, no se cambia el tipo (se elimina y recrea si es necesario)
- **Historial:** Gastos mantienen referencia aunque expense type se inactive

#### Reglas de Integridad:
1. **RI-EXP-001:** `expenseTypeId` debe existir
2. **RI-EXP-002:** Solo se crean gastos de expense types activos (validación en lógica)
3. **RI-EXP-003:** No se permite eliminar expense type con gastos asociados
4. **RI-EXP-004:** Si expense type se marca inactivo, gastos históricos se mantienen

#### Queries Comunes:
```typescript
// Gastos de un tipo específico
await prisma.expense.findMany({
  where: { expenseTypeId: typeId },
  orderBy: { date: 'desc' }
});

// Tipo de gasto con sus gastos del mes
await prisma.expenseType.findUnique({
  where: { id: typeId },
  include: {
    expenses: {
      where: {
        date: {
          gte: new Date('2025-07-01'),
          lt: new Date('2025-08-01')
        }
      }
    }
  }
});

// Total gastado por tipo en un período
await prisma.expense.groupBy({
  by: ['expenseTypeId'],
  where: {
    date: {
      gte: startDate,
      lt: endDate
    }
  },
  _sum: { amount: true }
});
```

---

### 5. Expense → CreditCard

**Tipo:** Many-to-One (opcional)  
**Cardinalidad:** N:1  
**Descripción:** Múltiples gastos pueden usar la misma tarjeta de crédito, pero un gasto solo usa una tarjeta (si aplica).

#### Schema Prisma:
```prisma
model Expense {
  id           String      @id @default(uuid())
  paymentType  PaymentType
  creditCardId String?     @map("credit_card_id")
  
  creditCard   CreditCard? @relation(fields: [creditCardId], references: [id])
}

model CreditCard {
  id       String    @id @default(uuid())
  name     String
  expenses Expense[]
}

enum PaymentType {
  CREDIT
  DEBIT
  TRANSFER
  CASH
}
```

#### Características:
- **Opcional:** Solo si `paymentType = CREDIT`
- **Validación condicional:** Si payment type es crédito, credit card es obligatoria
- **Puede ser null:** Si payment type no es crédito

#### Reglas de Integridad:
1. **RI-EXP-CC-001:** Si `paymentType = 'CREDIT'`, entonces `creditCardId` debe existir
2. **RI-EXP-CC-002:** Si `paymentType != 'CREDIT'`, entonces `creditCardId` debe ser null
3. **RI-EXP-CC-003:** `creditCardId` debe referenciar tarjeta activa (al momento de crear)
4. **RI-EXP-CC-004:** No se permite eliminar tarjeta con gastos asociados

#### Queries Comunes:
```typescript
// Gastos de una tarjeta
await prisma.expense.findMany({
  where: { creditCardId: cardId },
  orderBy: { date: 'desc' }
});

// Tarjeta con gastos del período de facturación
await prisma.creditCard.findUnique({
  where: { id: cardId },
  include: {
    expenses: {
      where: { billingPeriod: '2025-07' }
    }
  }
});

// Total por pagar en próximo vencimiento
const total = await prisma.expense.aggregate({
  where: {
    creditCardId: cardId,
    billingPeriod: currentPeriod
  },
  _sum: { amount: true }
});
```

---

### 6. Expense → DebitCard

**Tipo:** Many-to-One (opcional)  
**Cardinalidad:** N:1  
**Descripción:** Múltiples gastos pueden usar la misma tarjeta de débito, pero un gasto solo usa una tarjeta (si aplica).

#### Schema Prisma:
```prisma
model Expense {
  id          String      @id @default(uuid())
  paymentType PaymentType
  debitCardId String?     @map("debit_card_id")
  
  debitCard   DebitCard?  @relation(fields: [debitCardId], references: [id])
}

model DebitCard {
  id       String    @id @default(uuid())
  name     String
  expenses Expense[]
}
```

#### Características:
- **Opcional:** Solo si `paymentType = DEBIT`
- **Validación condicional:** Similar a credit card
- **Sin billing period:** Gastos con débito no tienen billing period

#### Reglas de Integridad:
1. **RI-EXP-DC-001:** Si `paymentType = 'DEBIT'`, entonces `debitCardId` debe existir
2. **RI-EXP-DC-002:** Si `paymentType != 'DEBIT'`, entonces `debitCardId` debe ser null
3. **RI-EXP-DC-003:** `debitCardId` debe referenciar tarjeta activa (al momento de crear)
4. **RI-EXP-DC-004:** No se permite eliminar tarjeta con gastos asociados

#### Queries Comunes:
```typescript
// Gastos de una tarjeta de débito
await prisma.expense.findMany({
  where: { debitCardId: cardId },
  orderBy: { date: 'desc' }
});

// Tarjeta con gastos del mes
await prisma.debitCard.findUnique({
  where: { id: cardId },
  include: {
    expenses: {
      where: {
        date: {
          gte: new Date('2025-07-01'),
          lt: new Date('2025-08-01')
        }
      }
    }
  }
});
```

---

### 7. ExpenseType → RecurringExpense

**Tipo:** One-to-One (opcional)  
**Cardinalidad:** 1:1  
**Descripción:** Un tipo de gasto puede tener una configuración recurrente, y una configuración recurrente pertenece a un solo tipo.

#### Schema Prisma:
```prisma
model ExpenseType {
  id               String            @id @default(uuid())
  name             String
  recurringExpense RecurringExpense?
}

model RecurringExpense {
  id            String      @id @default(uuid())
  expenseTypeId String      @unique @map("expense_type_id")
  amount        Decimal     @db.Decimal(12, 2)
  dayOfMonth    Int         @map("day_of_month")
  
  expenseType   ExpenseType @relation(fields: [expenseTypeId], references: [id])
}
```

#### Características:
- **Opcional:** No todos los expense types son recurrentes
- **Único:** `expenseTypeId` es unique (solo una config recurrente por tipo)
- **One-to-One:** Relación 1:1

#### Reglas de Integridad:
1. **RI-REC-001:** `expenseTypeId` debe existir y ser único
2. **RI-REC-002:** Solo puede haber una config recurrente por expense type
3. **RI-REC-003:** Al eliminar expense type, se elimina su config recurrente (cascada)
4. **RI-REC-004:** Expense type puede existir sin config recurrente

#### Queries Comunes:
```typescript
// Tipo de gasto con su configuración recurrente
await prisma.expenseType.findUnique({
  where: { id: typeId },
  include: { recurringExpense: true }
});

// Todos los gastos recurrentes activos
await prisma.recurringExpense.findMany({
  where: { isActive: true },
  include: { expenseType: true }
});

// Verificar si tipo tiene config recurrente
const hasRecurring = await prisma.recurringExpense.findUnique({
  where: { expenseTypeId: typeId }
});
```

---

### 8. RecurringExpense → CreditCard

**Tipo:** Many-to-One (opcional)  
**Cardinalidad:** N:1  
**Descripción:** Múltiples gastos recurrentes pueden usar la misma tarjeta, pero un recurrente solo usa una tarjeta (si aplica).

#### Schema Prisma:
```prisma
model RecurringExpense {
  id           String      @id @default(uuid())
  paymentType  PaymentType
  creditCardId String?     @map("credit_card_id")
  
  creditCard   CreditCard? @relation(fields: [creditCardId], references: [id])
}

model CreditCard {
  id                String             @id @default(uuid())
  recurringExpenses RecurringExpense[]
}
```

#### Características:
- **Opcional:** Solo si `paymentType = CREDIT`
- **Similar a Expense:** Mismas reglas de validación

#### Reglas de Integridad:
1. **RI-REC-CC-001:** Si `paymentType = 'CREDIT'`, entonces `creditCardId` obligatorio
2. **RI-REC-CC-002:** `creditCardId` debe referenciar tarjeta activa
3. **RI-REC-CC-003:** Al generar expense, hereda la tarjeta configurada

---

### 9. RecurringExpense → DebitCard

**Tipo:** Many-to-One (opcional)  
**Cardinalidad:** N:1  
**Descripción:** Múltiples gastos recurrentes pueden usar la misma tarjeta de débito, pero un recurrente solo usa una tarjeta (si aplica).

#### Schema Prisma:
```prisma
model RecurringExpense {
  id          String      @id @default(uuid())
  paymentType PaymentType
  debitCardId String?     @map("debit_card_id")
  
  debitCard   DebitCard?  @relation(fields: [debitCardId], references: [id])
}

model DebitCard {
  id                String             @id @default(uuid())
  recurringExpenses RecurringExpense[]
}
```

#### Características:
- **Opcional:** Solo si `paymentType = DEBIT`
- **Similar a Expense:** Mismas reglas de validación

#### Reglas de Integridad:
1. **RI-REC-DC-001:** Si `paymentType = 'DEBIT'`, entonces `debitCardId` obligatorio
2. **RI-REC-DC-002:** `debitCardId` debe referenciar tarjeta activa
3. **RI-REC-DC-003:** Al generar expense, hereda la tarjeta configurada

---

## Resumen de Cardinalidades

| Relación | Tipo | Cardinalidad | Opcional |
|----------|------|--------------|----------|
| Category → Category | Self-Reference | 1:N | Sí (padre) |
| Category → Budget | One-to-Many | 1:N | No |
| Category → ExpenseType | One-to-Many | 1:N | No |
| ExpenseType → Expense | One-to-Many | 1:N | No |
| ExpenseType → RecurringExpense | One-to-One | 1:1 | Sí |
| Expense → CreditCard | Many-to-One | N:1 | Sí (condicional) |
| Expense → DebitCard | Many-to-One | N:1 | Sí (condicional) |
| RecurringExpense → CreditCard | Many-to-One | N:1 | Sí (condicional) |
| RecurringExpense → DebitCard | Many-to-One | N:1 | Sí (condicional) |

---

## Integridad Referencial

### On Delete Behavior

#### Soft Delete (Mayoría de casos)
```prisma
// No hay ON DELETE en BD, manejado en lógica
// Se marca deleted_at en lugar de eliminar
```

**Entidades con soft delete:**
- Category
- ExpenseType
- CreditCard
- DebitCard
- Expense (opcional)

**Reglas:**
- Al soft-delete padre, verificar hijos/dependencias
- Posibilidad de restaurar
- Queries por defecto filtran `deleted_at IS NULL`

---

#### Restrict (No permitir eliminar)
Validado en lógica de negocio, no en BD.

**Casos:**
- No eliminar categoría con expense types activos
- No eliminar expense type con expenses
- No eliminar tarjetas con gastos
- No eliminar categoría padre con hijos activos

---

#### Cascade (Eliminar en cascada)
Algunas relaciones tienen cascada automática.

**Casos:**
- Eliminar categoría padre → eliminar hijos (soft-delete)
- Eliminar expense type → eliminar recurring expense (si existe)

---

### Validaciones de Consistencia

#### Al Crear

**Expense:**
1. Verificar expense type existe y está activo
2. Verificar tarjeta existe y está activa (si aplica)
3. Validar consistencia payment type - card
4. Calcular billing period si es crédito

**Budget:**
1. Verificar categoría existe y es padre
2. Verificar no existe presupuesto para ese período

**ExpenseType:**
1. Verificar categoría existe y está activa
2. Validar installment_config si is_installment = true

---

#### Al Actualizar

**Category parentId:**
1. Validar no crea ciclos
2. Validar no excede 2 niveles
3. Si tiene hijos, no puede moverse bajo otra

**ExpenseType categoryId:**
1. Nueva categoría debe existir y estar activa
2. Gastos históricos mantienen asociación original (no se actualizan)

**CreditCard cutoff_day:**
1. Solo afecta gastos futuros
2. No recalcular billing period de gastos históricos

---

#### Al Eliminar (Soft Delete)

**Category:**
1. Verificar no tiene expense types activos
2. Si tiene hijos, eliminar en cascada
3. Marcar deleted_at en padre e hijos

**ExpenseType:**
1. Verificar no tiene expenses (o permitir si solo se inactiva)
2. Eliminar recurring expense asociado

**CreditCard/DebitCard:**
1. Verificar no tiene gastos (o solo marcar inactiva)

---

## Consultas Cross-Domain

### Reporte Mensual por Categoría Padre

```typescript
// Todos los gastos del mes agrupados por categoría padre
const report = await prisma.expense.findMany({
  where: {
    date: {
      gte: new Date('2025-07-01'),
      lt: new Date('2025-08-01')
    }
  },
  include: {
    expenseType: {
      include: {
        category: {
          include: {
            parent: true  // Para obtener categoría padre si es hijo
          }
        }
      }
    }
  }
});

// Agrupar por categoría padre en código
const grouped = report.reduce((acc, expense) => {
  const category = expense.expenseType.category;
  const parentCategory = category.parent || category;
  
  if (!acc[parentCategory.id]) {
    acc[parentCategory.id] = {
      name: parentCategory.name,
      total: 0,
      expenses: []
    };
  }
  
  acc[parentCategory.id].total += expense.amount;
  acc[parentCategory.id].expenses.push(expense);
  
  return acc;
}, {});
```

---

### Próximo Pago de Tarjeta

```typescript
// Total a pagar en próximo vencimiento
const nextPayment = await prisma.expense.aggregate({
  where: {
    creditCardId: cardId,
    billingPeriod: currentPeriod
  },
  _sum: { amount: true },
  _count: true
});

// Con desglose
const breakdown = await prisma.expense.findMany({
  where: {
    creditCardId: cardId,
    billingPeriod: currentPeriod
  },
  include: {
    expenseType: {
      include: { category: true }
    }
  },
  orderBy: { amount: 'desc' }
});
```

---

### Presupuesto vs Gastado

```typescript
// Categoría con presupuesto y gastos del mes
const categoryData = await prisma.category.findUnique({
  where: { id: categoryId },
  include: {
    budgets: {
      where: { period: '2025-07' }
    },
    expenseTypes: {
      include: {
        expenses: {
          where: {
            date: {
              gte: new Date('2025-07-01'),
              lt: new Date('2025-08-01')
            }
          }
        }
      }
    }
  }
});

// Calcular total gastado
const totalSpent = categoryData.expenseTypes
  .flatMap(type => type.expenses)
  .reduce((sum, exp) => sum + exp.amount, 0);

const budget = categoryData.budgets[0]?.amount || null;
const percentageUsed = budget ? (totalSpent / budget) * 100 : null;
```

---

### Gastos Recurrentes del Mes

```typescript
// Generar todos los recurrentes activos
const recurrents = await prisma.recurringExpense.findMany({
  where: { isActive: true },
  include: {
    expenseType: true,
    creditCard: true,
    debitCard: true
  }
});

// Para cada uno, crear expense si no existe
for (const rec of recurrents) {
  const exists = await prisma.expense.findFirst({
    where: {
      expenseTypeId: rec.expenseTypeId,
      date: {
        gte: new Date('2025-07-01'),
        lt: new Date('2025-08-01')
      },
      isRecurring: true
    }
  });
  
  if (!exists) {
    await prisma.expense.create({
      data: {
        expenseTypeId: rec.expenseTypeId,
        amount: rec.amount,
        date: new Date(`2025-07-${rec.dayOfMonth}`),
        paymentType: rec.paymentType,
        creditCardId: rec.creditCardId,
        debitCardId: rec.debitCardId,
        isRecurring: true
      }
    });
  }
}
```

---

## Diagramas de Flujo de Datos

### Crear Gasto (Data Flow)

```
User Input (CreateExpenseDto)
    │
    ├─> expense_type_id
    │       │
    │       ├─> Query ExpenseType
    │       │       │
    │       │       └─> Include Category
    │       │
    │       └─> Validate is_active
    │
    ├─> credit_card_id (if payment_type = credit)
    │       │
    │       ├─> Query CreditCard
    │       │       │
    │       │       └─> Get cutoff_day
    │       │
    │       ├─> Validate is_active
    │       │
    │       └─> Calculate billing_period
    │
    └─> Create Expense
            │
            └─> Store in DB with all relations
```

---

### Reporte Mensual (Data Flow)

```
Request: GET /reports/monthly?month=2025-07
    │
    ├─> Query all Expenses in date range
    │       │
    │       └─> Include: ExpenseType → Category → Parent
    │
    ├─> Group by Parent Category
    │       │
    │       └─> Sum amounts per category
    │
    ├─> Query Budgets for month
    │       │
    │       └─> Match with categories
    │
    └─> Calculate:
            ├─> Total spent per category
            ├─> Budget vs spent
            ├─> Percentage used
            └─> Over budget flag
```

---

## Optimización de Queries

### Índices Recomendados

```sql
-- Expenses: búsqueda por fecha y billing period
CREATE INDEX idx_expenses_date ON expenses(date);
CREATE INDEX idx_expenses_billing_period ON expenses(billing_period);
CREATE INDEX idx_expenses_expense_type_id ON expenses(expense_type_id);
CREATE INDEX idx_expenses_credit_card_id ON expenses(credit_card_id);
CREATE INDEX idx_expenses_debit_card_id ON expenses(debit_card_id);

-- Budgets: búsqueda por período
CREATE INDEX idx_budgets_period ON budgets(period);
CREATE INDEX idx_budgets_category_id ON budgets(category_id);

-- Categories: jerarquía
CREATE INDEX idx_categories_parent_id ON categories(parent_id);

-- ExpenseTypes: búsqueda por categoría
CREATE INDEX idx_expense_types_category_id ON expense_types(category_id);

-- Soft delete: filtro común
CREATE INDEX idx_categories_deleted_at ON categories(deleted_at);
CREATE INDEX idx_expense_types_deleted_at ON expense_types(deleted_at);
CREATE INDEX idx_credit_cards_deleted_at ON credit_cards(deleted_at);
CREATE INDEX idx_debit_cards_deleted_at ON debit_cards(deleted_at);
```

---

### N+1 Query Prevention

**Problema:** Query por cada relación.

**Solución:** Usar `include` estratégicamente.

```typescript
// ❌ Malo: N+1 query
const expenses = await prisma.expense.findMany();
for (const exp of expenses) {
  const type = await prisma.expenseType.findUnique({
    where: { id: exp.expenseTypeId }
  });
}

// ✅ Bueno: Single query con include
const expenses = await prisma.expense.findMany({
  include: {
    expenseType: {
      include: { category: true }
    }
  }
});
```

---

## Referencias

- [Entities](./entities.md)
- [Business Rules](./business-rules.md)
- [Prisma Relations](https://www.prisma.io/docs/concepts/components/prisma-schema/relations)