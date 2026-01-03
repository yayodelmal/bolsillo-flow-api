# Seguimiento de Presupuestos (Budget Tracking)

## Overview
Este documento describe cómo se definen, consultan y monitorean los presupuestos mensuales por categoría, incluyendo cálculos de uso y detección de sobre-gasto.

---

## Propósito

Controlar el gasto mensual por categoría mediante:
- Definición de límites de gasto (presupuestos)
- Comparación automática de gasto real vs presupuesto
- Identificación de categorías sobre presupuesto
- Histórico de presupuestos para análisis de evolución

---

## Conceptos Clave

### Budget
Límite de gasto definido para una categoría padre en un mes específico.

**Componentes:**
- **Category:** Categoría padre (solo padre, no hijos)
- **Amount:** Monto límite del presupuesto
- **Period:** Mes específico (YYYY-MM)

### Budget Status
Estado actual del uso del presupuesto en un período.

**Métricas:**
- **Total Spent:** Cuánto se ha gastado
- **Percentage Used:** % del presupuesto usado
- **Remaining:** Cuánto queda disponible
- **Over Budget:** Si se excedió el límite

---

## Flujo Completo

### Fase 1: Definición de Presupuestos

#### Caso 1.1: Crear Presupuesto para Categoría Padre

**Request:**
```http
POST /api/budgets
Content-Type: application/json

{
  "categoryId": "uuid-departamento",
  "amount": 600000,
  "period": "2025-07"
}
```

**Validaciones:**
- `categoryId` debe ser categoría padre (parentId = null)
- `amount` > 0
- `period` formato YYYY-MM válido
- Solo un presupuesto por (categoryId, period)

**Response:**
```json
{
  "id": "uuid-budget-dept",
  "categoryId": "uuid-departamento",
  "amount": 600000,
  "period": "2025-07",
  "createdAt": "2025-07-01T10:00:00Z",
  "updatedAt": "2025-07-01T10:00:00Z"
}
```

---

#### Caso 1.2: Presupuesto Duplicado (Upsert)

**Escenario:** Ya existe presupuesto de Departamento para julio.

**Request:**
```http
POST /api/budgets
{
  "categoryId": "uuid-departamento",
  "amount": 650000,
  "period": "2025-07"
}
```

**Proceso:**
```typescript
async createOrUpdateBudget(dto: CreateBudgetDto) {
  // Buscar presupuesto existente
  const existing = await this.prisma.budget.findUnique({
    where: {
      categoryId_period: {
        categoryId: dto.categoryId,
        period: dto.period
      }
    }
  });
  
  if (existing) {
    // Actualizar existente
    return this.prisma.budget.update({
      where: { id: existing.id },
      data: { amount: dto.amount }
    });
  } else {
    // Crear nuevo
    return this.prisma.budget.create({
      data: dto
    });
  }
}
```

**Response:**
```json
{
  "id": "uuid-budget-dept",
  "categoryId": "uuid-departamento",
  "amount": 650000,
  "period": "2025-07",
  "createdAt": "2025-07-01T10:00:00Z",
  "updatedAt": "2025-07-15T14:00:00Z"
}
```

**Nota:** ID se mantiene, solo cambia amount y updatedAt.

---

#### Caso 1.3: Presupuestos de Múltiples Categorías

**Setup completo del mes:**

```http
POST /api/budgets
{"categoryId": "uuid-departamento", "amount": 600000, "period": "2025-07"}

POST /api/budgets
{"categoryId": "uuid-transporte", "amount": 150000, "period": "2025-07"}

POST /api/budgets
{"categoryId": "uuid-alimentacion", "amount": 200000, "period": "2025-07"}

POST /api/budgets
{"categoryId": "uuid-suscripciones", "amount": 50000, "period": "2025-07"}
```

**Resultado:** 4 presupuestos definidos para julio 2025.

---

### Fase 2: Consulta de Presupuestos

#### Consultar Presupuesto Específico

**Request:**
```http
GET /api/budgets/category/uuid-departamento/period/2025-07
```

**Response:**
```json
{
  "id": "uuid-budget-dept",
  "categoryId": "uuid-departamento",
  "amount": 600000,
  "period": "2025-07",
  "category": {
    "id": "uuid-departamento",
    "name": "Departamento"
  }
}
```

---

#### Listar Presupuestos de un Mes

**Request:**
```http
GET /api/budgets?period=2025-07
```

**Response:**
```json
[
  {
    "id": "uuid-budget-dept",
    "categoryId": "uuid-departamento",
    "categoryName": "Departamento",
    "amount": 600000,
    "period": "2025-07"
  },
  {
    "id": "uuid-budget-trans",
    "categoryId": "uuid-transporte",
    "categoryName": "Transporte",
    "amount": 150000,
    "period": "2025-07"
  },
  ...
]
```

---

#### Historial de Presupuesto de una Categoría

**Request:**
```http
GET /api/budgets/category/uuid-departamento/history
```

**Response:**
```json
[
  {
    "id": "uuid-budget-1",
    "period": "2025-07",
    "amount": 600000,
    "createdAt": "2025-07-01T10:00:00Z"
  },
  {
    "id": "uuid-budget-2",
    "period": "2025-06",
    "amount": 550000,
    "createdAt": "2025-06-01T10:00:00Z"
  },
  {
    "id": "uuid-budget-3",
    "period": "2025-05",
    "amount": 550000,
    "createdAt": "2025-05-01T10:00:00Z"
  }
]
```

**Análisis:** Presupuesto aumentó de $550k a $600k en julio.

---

### Fase 3: Seguimiento de Gasto vs Presupuesto

#### Cálculo de Budget Status

**Algoritmo:**
```typescript
async getBudgetStatus(categoryId: string, period: string) {
  // 1. Obtener presupuesto
  const budget = await this.prisma.budget.findUnique({
    where: {
      categoryId_period: { categoryId, period }
    },
    include: { category: true }
  });
  
  if (!budget) {
    return {
      categoryId,
      period,
      budgetDefined: false,
      message: 'No budget defined for this category in this period'
    };
  }
  
  // 2. Calcular gasto total del período
  const [year, month] = period.split('-').map(Number);
  const startDate = new Date(year, month - 1, 1);
  const endDate = new Date(year, month, 1);
  
  // Obtener todos los gastos de la categoría (padre e hijos)
  const categoryIds = await this.getCategoryAndChildren(categoryId);
  
  const expenses = await this.prisma.expense.findMany({
    where: {
      expenseType: {
        categoryId: { in: categoryIds }
      },
      date: {
        gte: startDate,
        lt: endDate
      }
    }
  });
  
  const totalSpent = expenses.reduce((sum, exp) => sum + exp.amount, 0);
  
  // 3. Calcular métricas
  const percentageUsed = (totalSpent / budget.amount) * 100;
  const remaining = budget.amount - totalSpent;
  const overBudget = totalSpent > budget.amount;
  
  return {
    categoryId,
    categoryName: budget.category.name,
    period,
    budgetDefined: true,
    budget: budget.amount,
    totalSpent,
    remaining,
    percentageUsed: Math.round(percentageUsed * 10) / 10, // 1 decimal
    overBudget,
    overBudgetAmount: overBudget ? totalSpent - budget.amount : 0,
    expensesCount: expenses.length
  };
}
```

---

#### Consultar Status de Presupuesto

**Request:**
```http
GET /api/categories/uuid-departamento/budget-status?period=2025-07
```

**Response (Sin sobre-gasto):**
```json
{
  "categoryId": "uuid-departamento",
  "categoryName": "Departamento",
  "period": "2025-07",
  "budgetDefined": true,
  "budget": 600000,
  "totalSpent": 598205,
  "remaining": 1795,
  "percentageUsed": 99.7,
  "overBudget": false,
  "overBudgetAmount": 0,
  "expensesCount": 5
}
```

---

**Response (Sobre presupuesto):**
```json
{
  "categoryId": "uuid-alimentacion",
  "categoryName": "Alimentación",
  "period": "2025-07",
  "budgetDefined": true,
  "budget": 200000,
  "totalSpent": 235000,
  "remaining": -35000,
  "percentageUsed": 117.5,
  "overBudget": true,
  "overBudgetAmount": 35000,
  "expensesCount": 18
}
```

---

**Response (Sin presupuesto definido):**
```json
{
  "categoryId": "uuid-educacion",
  "categoryName": "Educación",
  "period": "2025-07",
  "budgetDefined": false,
  "message": "No budget defined for this category in this period"
}
```

---

### Fase 4: Reporte Mensual Consolidado

#### Todas las Categorías con Budget Status

**Request:**
```http
GET /api/expenses/monthly-report?month=2025-07
```

**Response:**
```json
{
  "month": "2025-07",
  "summary": {
    "totalSpent": 1009209,
    "totalBudget": 1200000,
    "budgetUsedPercentage": 84.1,
    "categoriesWithBudget": 4,
    "categoriesOverBudget": 1
  },
  "categories": [
    {
      "categoryId": "uuid-departamento",
      "categoryName": "Departamento",
      "total": 598205,
      "budget": 600000,
      "percentageUsed": 99.7,
      "overBudget": false,
      "expensesCount": 5,
      "breakdown": [
        {
          "expenseTypeName": "Arriendo",
          "amount": 400000
        },
        {
          "expenseTypeName": "Luz",
          "amount": 95400
        },
        {
          "expenseTypeName": "Agua",
          "amount": 18950
        },
        {
          "expenseTypeName": "Internet",
          "amount": 11604
        },
        {
          "expenseTypeName": "Gastos Comunes",
          "amount": 72251
        }
      ]
    },
    {
      "categoryId": "uuid-alimentacion",
      "categoryName": "Alimentación",
      "total": 235000,
      "budget": 200000,
      "percentageUsed": 117.5,
      "overBudget": true,
      "overBudgetAmount": 35000,
      "expensesCount": 18,
      "breakdown": [...]
    },
    {
      "categoryId": "uuid-transporte",
      "categoryName": "Transporte",
      "total": 82000,
      "budget": 150000,
      "percentageUsed": 54.7,
      "overBudget": false,
      "expensesCount": 12,
      "breakdown": [...]
    },
    {
      "categoryId": "uuid-suscripciones",
      "categoryName": "Suscripciones",
      "total": 35868,
      "budget": 50000,
      "percentageUsed": 71.7,
      "overBudget": false,
      "expensesCount": 6,
      "breakdown": [...]
    },
    {
      "categoryId": "uuid-educacion",
      "categoryName": "Educación",
      "total": 53000,
      "budget": null,
      "percentageUsed": null,
      "overBudget": false,
      "expensesCount": 1,
      "breakdown": [...]
    }
  ]
}
```

---

### Fase 5: Análisis de Evolución

#### Comparar Presupuestos Mes a Mes

**Request:**
```http
GET /api/budgets/category/uuid-departamento/history?startPeriod=2025-01&endPeriod=2025-07
```

**Response:**
```json
[
  {"period": "2025-07", "amount": 600000},
  {"period": "2025-06", "amount": 550000},
  {"period": "2025-05", "amount": 550000},
  {"period": "2025-04", "amount": 550000},
  {"period": "2025-03", "amount": 520000},
  {"period": "2025-02", "amount": 520000},
  {"period": "2025-01", "amount": 500000}
]
```

**Análisis:**
- Aumentó $100k de enero a julio (20% de incremento)
- Ajustes en marzo (+$30k) y julio (+$50k)

---

#### Comparar Gasto Real vs Presupuesto Histórico

**Algoritmo:**
```typescript
async getBudgetTrend(categoryId: string, periods: string[]) {
  const results = [];
  
  for (const period of periods) {
    const status = await this.getBudgetStatus(categoryId, period);
    results.push({
      period,
      budget: status.budget,
      spent: status.totalSpent,
      difference: status.remaining,
      percentageUsed: status.percentageUsed,
      overBudget: status.overBudget
    });
  }
  
  return results;
}
```

**Request:**
```http
GET /api/categories/uuid-departamento/budget-trend?periods=2025-01,2025-02,2025-03,2025-04,2025-05,2025-06,2025-07
```

**Response:**
```json
[
  {
    "period": "2025-01",
    "budget": 500000,
    "spent": 478913,
    "difference": 21087,
    "percentageUsed": 95.8,
    "overBudget": false
  },
  {
    "period": "2025-02",
    "budget": 520000,
    "spent": 633761,
    "difference": -113761,
    "percentageUsed": 121.9,
    "overBudget": true
  },
  ...
  {
    "period": "2025-07",
    "budget": 600000,
    "spent": 598205,
    "difference": 1795,
    "percentageUsed": 99.7,
    "overBudget": false
  }
]
```

**Insights:**
- Febrero: Sobre presupuesto (gastos comunes anormales)
- Marzo-Junio: Ajustado bien
- Julio: Muy cerca del límite (99.7%)

---

## Helper Functions

### Obtener Categoría y sus Hijos

```typescript
async getCategoryAndChildren(categoryId: string): Promise<string[]> {
  const category = await this.prisma.category.findUnique({
    where: { id: categoryId },
    include: { children: true }
  });
  
  const ids = [categoryId];
  
  if (category.children) {
    ids.push(...category.children.map(c => c.id));
  }
  
  return ids;
}
```

**Uso:** Sumar gastos de categoría padre + hijos.

---

### Calcular Promedio de Gasto

```typescript
async getAverageSpending(
  categoryId: string,
  startPeriod: string,
  endPeriod: string
): Promise<number> {
  const periods = this.generatePeriods(startPeriod, endPeriod);
  const totals = [];
  
  for (const period of periods) {
    const status = await this.getBudgetStatus(categoryId, period);
    totals.push(status.totalSpent);
  }
  
  return totals.reduce((sum, t) => sum + t, 0) / totals.length;
}
```

---

### Proyectar Gasto del Mes

```typescript
async projectMonthlySpending(
  categoryId: string,
  currentPeriod: string
): Promise<number> {
  const [year, month] = currentPeriod.split('-').map(Number);
  const today = new Date();
  const dayOfMonth = today.getDate();
  const daysInMonth = new Date(year, month, 0).getDate();
  
  // Calcular gasto hasta hoy
  const status = await this.getBudgetStatus(categoryId, currentPeriod);
  const spentSoFar = status.totalSpent;
  
  // Proyectar para el mes completo
  const dailyAverage = spentSoFar / dayOfMonth;
  const projected = dailyAverage * daysInMonth;
  
  return Math.round(projected);
}
```

**Ejemplo:**
```
Hoy: 15 de julio
Gastado hasta ahora: $300,000
Días transcurridos: 15
Días en julio: 31

Promedio diario: $300,000 / 15 = $20,000
Proyección mensual: $20,000 * 31 = $620,000

Si presupuesto es $600,000:
  → Proyección indica que se pasará del presupuesto
```

---

## Alertas (Futuro - Backlog)

### Umbrales de Alerta

**Configuración:**
```json
{
  "categoryId": "uuid-departamento",
  "alerts": [
    {
      "threshold": 75,
      "type": "warning",
      "message": "Has usado el 75% del presupuesto"
    },
    {
      "threshold": 90,
      "type": "caution",
      "message": "Cuidado: 90% del presupuesto usado"
    },
    {
      "threshold": 100,
      "type": "danger",
      "message": "¡Presupuesto excedido!"
    }
  ]
}
```

**Trigger:**
Al crear expense, verificar si se cruza un umbral y enviar alerta.

---

## Validaciones

### VL-BUD-001: Solo Categorías Padre

```typescript
async validateCategoryIsParent(categoryId: string): Promise<void> {
  const category = await this.prisma.category.findUnique({
    where: { id: categoryId }
  });
  
  if (category.parentId !== null) {
    throw new UnprocessableEntityException(
      'Budgets can only be defined for parent categories'
    );
  }
}
```

---

### VL-BUD-002: Período Válido

```typescript
function validatePeriod(period: string): void {
  const regex = /^\d{4}-(0[1-9]|1[0-2])$/;
  
  if (!regex.test(period)) {
    throw new BadRequestException(
      'Period must be in YYYY-MM format'
    );
  }
  
  const [year, month] = period.split('-').map(Number);
  
  if (year < 2020 || year > 2099) {
    throw new BadRequestException('Year must be between 2020 and 2099');
  }
}
```

---

## Testing

### Unit Tests

```typescript
describe('Budget Tracking', () => {
  describe('getBudgetStatus', () => {
    it('should calculate percentage used correctly', async () => {
      // Setup: budget = 100000, spent = 75000
      const status = await service.getBudgetStatus(categoryId, '2025-07');
      
      expect(status.percentageUsed).toBe(75.0);
      expect(status.overBudget).toBe(false);
      expect(status.remaining).toBe(25000);
    });
    
    it('should detect over budget', async () => {
      // Setup: budget = 100000, spent = 120000
      const status = await service.getBudgetStatus(categoryId, '2025-07');
      
      expect(status.overBudget).toBe(true);
      expect(status.overBudgetAmount).toBe(20000);
      expect(status.remaining).toBe(-20000);
    });
    
    it('should include children expenses', async () => {
      // Setup: parent + 2 children with expenses
      const status = await service.getBudgetStatus(parentId, '2025-07');
      
      // Should sum expenses from parent and both children
      expect(status.expensesCount).toBeGreaterThan(0);
    });
  });
  
  describe('projectMonthlySpending', () => {
    it('should project based on daily average', async () => {
      const projected = await service.projectMonthlySpending(
        categoryId,
        '2025-07'
      );
      
      expect(projected).toBeGreaterThan(0);
    });
  });
});
```

---

## UI Considerations

### Indicadores Visuales

**Verde (< 75%):**
```
Departamento: 54% usado ($324k / $600k)
█████████░░░░░░░ 54%
```

**Amarillo (75-90%):**
```
Alimentación: 85% usado ($170k / $200k)
██████████████▓░ 85%
```

**Naranja (90-100%):**
```
Departamento: 99.7% usado ($598k / $600k)
███████████████▓ 99.7%
```

**Rojo (> 100%):**
```
Alimentación: 117% usado ($235k / $200k)
█████████████████ 117% ⚠️ Sobre presupuesto
```

---

## Referencias

- [Business Rules - BR-BUD-001 to BR-BUD-005](../../03-data-model/business-rules.md#presupuestos)
- [Endpoints - Budgets](../../04-api/endpoints.md#budgets)
- [Requirements - RF-008 to RF-010](../../01-product/requirements.md#gestión-de-presupuestos)
- [Backlog - BL-001: Alertas de Presupuesto](../../01-product/backlog.md#bl-001-alertas-de-presupuesto)