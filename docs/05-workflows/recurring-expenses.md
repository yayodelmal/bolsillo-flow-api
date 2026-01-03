# Gastos Recurrentes (Recurring Expenses)

## Overview
Este documento describe el flujo completo de gestión de gastos recurrentes, desde su configuración hasta la generación automática de gastos mensuales.

---

## Propósito

Automatizar la creación de gastos que se repiten mes a mes, evitando:
- Registro manual mensual de gastos fijos
- Olvidos de registrar gastos recurrentes
- Trabajo repetitivo

**Tipos de gastos recurrentes:**
- **Fijos:** Monto no cambia (ej: Arriendo $400,000)
- **Variables:** Monto puede cambiar (ej: Gastos Comunes ~$71,000)

---

## Conceptos Clave

### Recurring Expense Configuration
Configuración que define cómo y cuándo se debe generar un gasto automáticamente.

**Componentes:**
- **ExpenseType:** Qué tipo de gasto es (Arriendo, Internet, etc.)
- **Amount:** Monto del gasto
- **Day of Month:** Día del mes en que se genera (1-31)
- **Payment Type:** Método de pago (credit/debit/transfer/cash)
- **Card:** Tarjeta asociada si aplica
- **Frequency:** Mensual o anual
- **Is Fixed Amount:** Si el monto es fijo o variable
- **Is Active:** Si está activo para generación

### Fixed Amount vs Variable Amount

**Fixed Amount (isFixedAmount = true):**
- Monto no cambia mes a mes
- Se usa el `amount` configurado siempre
- Ejemplo: Arriendo, Internet, Suscripciones

**Variable Amount (isFixedAmount = false):**
- Monto puede variar mes a mes
- Se genera con `amount` como estimado o $0
- Usuario edita después con el monto real
- Opcionalmente puede usar monto del mes anterior
- Ejemplo: Gastos Comunes, CAE

---

## Flujo Completo

### Fase 1: Configuración de Recurrentes

#### Caso 1.1: Gasto Fijo (Arriendo)

**Request:**
```http
POST /api/recurring-expenses
Content-Type: application/json

{
  "expenseTypeId": "uuid-arriendo",
  "amount": 400000,
  "dayOfMonth": 1,
  "paymentType": "debit",
  "debitCardId": "uuid-cuenta-rut",
  "frequency": "monthly",
  "isFixedAmount": true
}
```

**Validaciones:**
- `expenseTypeId` debe existir y no tener otra config recurrente
- `amount` > 0 si isFixedAmount = true
- `dayOfMonth` entre 1 y 31
- Consistencia payment type - card

**Response:**
```json
{
  "id": "uuid-rec-arriendo",
  "expenseTypeId": "uuid-arriendo",
  "amount": 400000,
  "dayOfMonth": 1,
  "paymentType": "debit",
  "debitCardId": "uuid-cuenta-rut",
  "creditCardId": null,
  "frequency": "monthly",
  "isFixedAmount": true,
  "isActive": true,
  "lastGeneratedDate": null,
  "createdAt": "2025-01-02T10:00:00Z",
  "updatedAt": "2025-01-02T10:00:00Z"
}
```

---

#### Caso 1.2: Gasto Variable (Gastos Comunes)

**Request:**
```http
POST /api/recurring-expenses
Content-Type: application/json

{
  "expenseTypeId": "uuid-gastos-comunes",
  "amount": 71000,
  "dayOfMonth": 1,
  "paymentType": "debit",
  "debitCardId": "uuid-cuenta-rut",
  "frequency": "monthly",
  "isFixedAmount": false,
  "usePreviousMonthAmount": true
}
```

**Nota:** `amount` es un estimado inicial. Puede ser 0.

**Response:**
```json
{
  "id": "uuid-rec-gc",
  "expenseTypeId": "uuid-gastos-comunes",
  "amount": 71000,
  "dayOfMonth": 1,
  "paymentType": "debit",
  "debitCardId": "uuid-cuenta-rut",
  "frequency": "monthly",
  "isFixedAmount": false,
  "usePreviousMonthAmount": true,
  "isActive": true,
  ...
}
```

---

#### Caso 1.3: Suscripción con Tarjeta de Crédito

**Request:**
```http
POST /api/recurring-expenses
Content-Type: application/json

{
  "expenseTypeId": "uuid-chatgpt",
  "amount": 22452,
  "dayOfMonth": 15,
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "frequency": "monthly",
  "isFixedAmount": true
}
```

**Response:**
```json
{
  "id": "uuid-rec-chatgpt",
  "expenseTypeId": "uuid-chatgpt",
  "amount": 22452,
  "dayOfMonth": 15,
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "frequency": "monthly",
  "isFixedAmount": true,
  "isActive": true,
  ...
}
```

---

### Fase 2: Generación de Gastos

#### Opción A: Generación Manual (Mes Específico)

**Request:**
```http
POST /api/recurring-expenses/generate
Content-Type: application/json

{
  "year": 2025,
  "month": 7
}
```

**Proceso interno:**
```typescript
async generateMonthlyExpenses(year: number, month: number) {
  // 1. Obtener todas las config recurrentes activas
  const recurrings = await this.prisma.recurringExpense.findMany({
    where: { 
      isActive: true,
      frequency: 'monthly'
    },
    include: {
      expenseType: true,
      creditCard: true,
      debitCard: true
    }
  });
  
  const results = [];
  
  for (const rec of recurrings) {
    // 2. Verificar si ya fue generado
    if (this.wasAlreadyGenerated(rec, year, month)) {
      results.push({
        expenseTypeName: rec.expenseType.name,
        status: 'skipped',
        reason: 'Already generated for this period'
      });
      continue;
    }
    
    // 3. Determinar monto
    let amount = rec.amount;
    
    if (!rec.isFixedAmount && rec.usePreviousMonthAmount) {
      amount = await this.getPreviousMonthAmount(rec.expenseTypeId, year, month);
    }
    
    // 4. Crear expense
    try {
      const expense = await this.prisma.expense.create({
        data: {
          expenseTypeId: rec.expenseTypeId,
          amount,
          date: new Date(year, month - 1, rec.dayOfMonth),
          paymentType: rec.paymentType,
          creditCardId: rec.creditCardId,
          debitCardId: rec.debitCardId,
          isRecurring: true,
          notes: rec.isFixedAmount 
            ? 'Auto-generated from recurring config'
            : 'Auto-generated - please update amount'
        }
      });
      
      // 5. Actualizar last generated date
      await this.prisma.recurringExpense.update({
        where: { id: rec.id },
        data: { lastGeneratedDate: new Date() }
      });
      
      results.push({
        expenseTypeName: rec.expenseType.name,
        status: 'generated',
        expenseId: expense.id,
        amount: expense.amount,
        date: expense.date
      });
      
    } catch (error) {
      results.push({
        expenseTypeName: rec.expenseType.name,
        status: 'error',
        error: error.message
      });
    }
  }
  
  return {
    generated: results.filter(r => r.status === 'generated').length,
    skipped: results.filter(r => r.status === 'skipped').length,
    errors: results.filter(r => r.status === 'error').length,
    details: results
  };
}
```

**Response:**
```json
{
  "generated": 12,
  "skipped": 2,
  "errors": 0,
  "details": [
    {
      "expenseTypeName": "Arriendo",
      "status": "generated",
      "expenseId": "uuid-exp-arr",
      "amount": 400000,
      "date": "2025-07-01T00:00:00Z"
    },
    {
      "expenseTypeName": "ChatGPT",
      "status": "generated",
      "expenseId": "uuid-exp-gpt",
      "amount": 22452,
      "date": "2025-07-15T00:00:00Z"
    },
    {
      "expenseTypeName": "Uber One",
      "status": "skipped",
      "reason": "Already generated for this period"
    },
    {
      "expenseTypeName": "Gastos Comunes",
      "status": "generated",
      "expenseId": "uuid-exp-gc",
      "amount": 71866,
      "date": "2025-07-01T00:00:00Z"
    }
  ]
}
```

---

#### Opción B: Generación Anual

**Request:**
```http
POST /api/recurring-expenses/generate-year
Content-Type: application/json

{
  "year": 2025
}
```

**Proceso:**
Llama a `generateMonthlyExpenses()` para cada mes (1-12).

**Response:**
```json
{
  "year": 2025,
  "totalGenerated": 144,
  "byMonth": {
    "01": 12,
    "02": 12,
    "03": 12,
    "04": 12,
    "05": 12,
    "06": 12,
    "07": 12,
    "08": 12,
    "09": 12,
    "10": 12,
    "11": 12,
    "12": 12
  }
}
```

---

### Fase 3: Edición de Gastos Variables

#### Paso 3.1: Generar Gastos del Mes

```http
POST /api/recurring-expenses/generate
{
  "year": 2025,
  "month": 7
}
```

**Resultado:** Gasto de Gastos Comunes creado con monto del mes anterior ($71,866).

---

#### Paso 3.2: Llega la Cuenta Real

Usuario recibe cuenta de gastos comunes: $72,500 (varió).

---

#### Paso 3.3: Actualizar Monto

**Request:**
```http
PUT /api/expenses/uuid-exp-gc
Content-Type: application/json

{
  "amount": 72500,
  "notes": "Monto actualizado según cuenta recibida"
}
```

**Response:**
```json
{
  "id": "uuid-exp-gc",
  "amount": 72500,
  "notes": "Monto actualizado según cuenta recibida",
  "updatedAt": "2025-07-05T14:30:00Z",
  ...
}
```

---

### Fase 4: Gestión de Recurrentes

#### Pausar Gasto Recurrente

**Escenario:** Usuario cancela Uber One.

**Request:**
```http
PATCH /api/recurring-expenses/uuid-rec-uber-one/pause
```

**Proceso:**
```typescript
async pauseRecurringExpense(id: string) {
  return this.prisma.recurringExpense.update({
    where: { id },
    data: { 
      isActive: false,
      pausedAt: new Date()
    }
  });
}
```

**Response:**
```json
{
  "id": "uuid-rec-uber-one",
  "isActive": false,
  "pausedAt": "2025-07-15T10:00:00Z",
  ...
}
```

**Efecto:** En futuras generaciones, Uber One se salta (no se crea expense).

---

#### Reactivar con Nuevo Precio

**Escenario:** Usuario reactiva Uber One pero el precio subió.

**Request:**
```http
PATCH /api/recurring-expenses/uuid-rec-uber-one/reactivate
Content-Type: application/json

{
  "amount": 4500
}
```

**Proceso:**
```typescript
async reactivateRecurringExpense(id: string, newAmount?: number) {
  const data: any = {
    isActive: true,
    reactivatedAt: new Date()
  };
  
  if (newAmount !== undefined) {
    data.amount = newAmount;
  }
  
  return this.prisma.recurringExpense.update({
    where: { id },
    data
  });
}
```

**Response:**
```json
{
  "id": "uuid-rec-uber-one",
  "amount": 4500,
  "isActive": true,
  "reactivatedAt": "2025-10-01T10:00:00Z",
  ...
}
```

**Efecto:** Próximas generaciones usan $4,500.

---

#### Actualizar Precio de Recurrente

**Escenario:** Internet sube de $19,990 a $21,000.

**Request:**
```http
PUT /api/recurring-expenses/uuid-rec-internet
Content-Type: application/json

{
  "amount": 21000
}
```

**Response:**
```json
{
  "id": "uuid-rec-internet",
  "amount": 21000,
  "updatedAt": "2025-07-01T10:00:00Z",
  ...
}
```

**Efecto:**
- Gastos ya generados (ene-jun) mantienen $19,990
- Futuros gastos (jul-dic) se generan con $21,000

---

## Validaciones Importantes

### VL-REC-001: Expense Type Único

**Regla:** Un ExpenseType solo puede tener una config recurrente.

**Validación:**
```typescript
async validateUniqueExpenseType(expenseTypeId: string): Promise<void> {
  const existing = await this.prisma.recurringExpense.findUnique({
    where: { expenseTypeId }
  });
  
  if (existing) {
    throw new UnprocessableEntityException(
      'Expense type already has a recurring configuration'
    );
  }
}
```

---

### VL-REC-002: No Duplicar Generación

**Regla:** No generar si ya existe para ese período.

**Validación:**
```typescript
async wasAlreadyGenerated(
  rec: RecurringExpense,
  year: number,
  month: number
): Promise<boolean> {
  // Verificar por last generated date
  if (rec.lastGeneratedDate) {
    const lastGen = new Date(rec.lastGeneratedDate);
    const target = new Date(year, month - 1, 1);
    
    if (lastGen >= target) {
      return true; // Ya fue generado para este mes o posterior
    }
  }
  
  // Double-check: buscar expense real
  const exists = await this.prisma.expense.findFirst({
    where: {
      expenseTypeId: rec.expenseTypeId,
      isRecurring: true,
      date: {
        gte: new Date(year, month - 1, 1),
        lt: new Date(year, month, 1)
      }
    }
  });
  
  return !!exists;
}
```

---

### VL-REC-003: Monto >= 0 si Variable

**Regla:** Si isFixedAmount = false, amount puede ser 0.

**Validación:**
```typescript
function validateAmount(amount: number, isFixedAmount: boolean): void {
  if (isFixedAmount && amount <= 0) {
    throw new BadRequestException('Fixed amount must be greater than 0');
  }
  
  if (amount < 0) {
    throw new BadRequestException('Amount cannot be negative');
  }
}
```

---

## Helpers Útiles

### Obtener Monto del Mes Anterior

```typescript
async getPreviousMonthAmount(
  expenseTypeId: string,
  year: number,
  month: number
): Promise<number> {
  // Calcular mes anterior
  let prevMonth = month - 1;
  let prevYear = year;
  
  if (prevMonth < 1) {
    prevMonth = 12;
    prevYear--;
  }
  
  // Buscar expense del mes anterior
  const prevExpense = await this.prisma.expense.findFirst({
    where: {
      expenseTypeId,
      date: {
        gte: new Date(prevYear, prevMonth - 1, 1),
        lt: new Date(prevYear, prevMonth, 1)
      }
    },
    orderBy: { date: 'desc' }
  });
  
  // Si existe, usar su monto. Si no, usar monto configurado.
  return prevExpense?.amount ?? this.recurringConfig.amount;
}
```

---

### Ajustar Día de Mes

```typescript
function adjustDayOfMonth(dayOfMonth: number, year: number, month: number): number {
  const lastDay = new Date(year, month, 0).getDate();
  return Math.min(dayOfMonth, lastDay);
}
```

**Ejemplo:**
```
dayOfMonth configurado: 31
Mes: Abril (30 días)

adjustDayOfMonth(31, 2025, 4) → 30
```

---

## Casos Especiales

### Caso 1: Primer Mes (Sin Mes Anterior)

**Problema:** Se genera por primera vez, no hay mes anterior.

**Solución:**
```typescript
if (!prevExpense) {
  // Usar monto configurado como fallback
  return rec.amount || 0;
}
```

---

### Caso 2: Mes con Menos Días

**Problema:** dayOfMonth = 31, pero mes tiene 30 días.

**Solución:**
```typescript
const effectiveDay = adjustDayOfMonth(rec.dayOfMonth, year, month);
const date = new Date(year, month - 1, effectiveDay);
```

---

### Caso 3: Cambio de Tarjeta

**Problema:** Usuario cambia de tarjeta para un recurrente.

**Solución:**
```http
PUT /api/recurring-expenses/uuid-rec
{
  "creditCardId": "uuid-new-card"
}
```

**Efecto:** Futuros gastos usan nueva tarjeta. Históricos mantienen anterior.

---

### Caso 4: Frequency Annual

**Uso:** Gastos que ocurren una vez al año (ej: Seguro Anual).

**Config:**
```json
{
  "expenseTypeId": "uuid-seguro-anual",
  "amount": 500000,
  "dayOfMonth": 1,
  "frequency": "annual",
  ...
}
```

**Generación:**
```typescript
if (rec.frequency === 'annual') {
  // Solo generar en mes específico (ej: enero)
  if (month !== 1) {
    continue; // Skip
  }
}
```

---

## Testing

### Unit Tests

```typescript
describe('Recurring Expenses', () => {
  describe('generateMonthlyExpenses', () => {
    it('should generate all active recurring expenses', async () => {
      const result = await service.generateMonthlyExpenses(2025, 7);
      
      expect(result.generated).toBeGreaterThan(0);
      expect(result.errors).toBe(0);
    });
    
    it('should skip already generated expenses', async () => {
      await service.generateMonthlyExpenses(2025, 7);
      const result = await service.generateMonthlyExpenses(2025, 7);
      
      expect(result.skipped).toBeGreaterThan(0);
    });
    
    it('should use previous month amount for variables', async () => {
      const result = await service.generateMonthlyExpenses(2025, 8);
      
      const gcExpense = result.details.find(
        d => d.expenseTypeName === 'Gastos Comunes'
      );
      
      // Verificar que usó monto de julio
      expect(gcExpense.amount).toBe(72500);
    });
  });
  
  describe('pauseRecurringExpense', () => {
    it('should mark as inactive', async () => {
      const rec = await service.pauseRecurringExpense(recurringId);
      
      expect(rec.isActive).toBe(false);
      expect(rec.pausedAt).toBeDefined();
    });
  });
});
```

---

## Automatización (Futuro)

### Cron Job Mensual

```typescript
@Cron('0 0 1 * *') // Día 1 de cada mes a las 00:00
async autoGenerateMonthlyExpenses() {
  const today = new Date();
  const year = today.getFullYear();
  const month = today.getMonth() + 1;
  
  try {
    const result = await this.generateMonthlyExpenses(year, month);
    
    console.log(`Auto-generated expenses for ${year}-${month}:`, result);
    
    // Opcional: notificar al usuario
    if (result.generated > 0) {
      await this.notificationService.send({
        message: `${result.generated} gastos recurrentes generados para ${year}-${month}`,
        type: 'info'
      });
    }
  } catch (error) {
    console.error('Error auto-generating expenses:', error);
  }
}
```

---

## Referencias

- [Business Rules - BR-REC-001 to BR-REC-008](../../03-data-model/business-rules.md#gastos-recurrentes)
- [Endpoints - Recurring Expenses](../../04-api/endpoints.md#recurring-expenses)
- [Requirements - RF-022 to RF-025](../../01-product/requirements.md#gastos-recurrentes)