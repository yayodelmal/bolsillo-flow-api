# Cálculo de Billing Cycle (Período de Facturación)

## Overview
Este documento describe el algoritmo y la lógica para calcular automáticamente el período de facturación de gastos realizados con tarjeta de crédito.

---

## Propósito

Determinar automáticamente en qué período mensual será facturado un gasto según:
- La fecha en que se realizó la transacción
- El día de corte de la tarjeta de crédito

**Importancia:**
- Permite saber cuándo se debe pagar el gasto
- Agrupa gastos por período de facturación
- Calcula correctamente el total a pagar cada mes
- Fundamental para planificación de flujo de caja

---

## Conceptos Clave

### Día de Corte (Cutoff Day)
El último día del mes en que se incluyen transacciones en un período de facturación.

**Ejemplo:** Si el corte es día 15:
- Transacciones del 1 al 15 → Se facturan ese mes
- Transacciones del 16 en adelante → Se facturan el mes siguiente

### Billing Period
Período mensual en formato `YYYY-MM` que identifica en qué mes será facturado el gasto.

**Ejemplo:** `2025-07` significa que el gasto será incluido en la factura de julio 2025.

### Payment Due Date
Fecha límite de pago, típicamente algunos días después del corte.

**Ejemplo:** Si corte es día 15 y vencimiento es día 5:
- Período `2025-07` (corte 15 julio) → Pago antes del 5 agosto

---

## Algoritmo de Cálculo

### Pseudocódigo

```
FUNCIÓN calculateBillingPeriod(transactionDate, cutoffDay):
    // Extraer día del mes de la transacción
    transactionDay = transactionDate.day
    
    // Obtener mes y año de la transacción
    transactionMonth = transactionDate.month
    transactionYear = transactionDate.year
    
    // Regla principal
    SI transactionDay <= cutoffDay ENTONCES
        // La transacción cae antes o en el corte
        billingPeriod = formatPeriod(transactionYear, transactionMonth)
    SINO
        // La transacción cae después del corte
        nextMonth = addMonth(transactionDate, 1)
        billingPeriod = formatPeriod(nextMonth.year, nextMonth.month)
    FIN SI
    
    RETORNAR billingPeriod
FIN FUNCIÓN
```

### Implementación TypeScript

```typescript
function calculateBillingPeriod(
  transactionDate: Date,
  cutoffDay: number
): string {
  const transactionDay = transactionDate.getDate();
  const transactionMonth = transactionDate.getMonth(); // 0-11
  const transactionYear = transactionDate.getFullYear();
  
  if (transactionDay <= cutoffDay) {
    // Cae en el período actual
    const period = `${transactionYear}-${String(transactionMonth + 1).padStart(2, '0')}`;
    return period;
  } else {
    // Cae en el período siguiente
    const nextMonth = new Date(transactionYear, transactionMonth + 1, 1);
    const nextYear = nextMonth.getFullYear();
    const nextMonthNumber = nextMonth.getMonth() + 1;
    const period = `${nextYear}-${String(nextMonthNumber).padStart(2, '0')}`;
    return period;
  }
}
```

---

## Ejemplos Prácticos

### Caso 1: Gasto Antes del Corte

**Datos:**
- Tarjeta: BCI Crédito
- Día de corte: 15
- Fecha de transacción: 10 de junio 2025

**Cálculo:**
```
transactionDay = 10
cutoffDay = 15

10 <= 15 → TRUE

billingPeriod = "2025-06"
```

**Resultado:**
- Billing Period: `2025-06` (junio)
- Fecha de corte: 15 de junio
- Fecha de vencimiento: 5 de julio
- Días hasta pago: ~25 días desde la transacción

---

### Caso 2: Gasto Después del Corte

**Datos:**
- Tarjeta: BCI Crédito
- Día de corte: 15
- Fecha de transacción: 20 de junio 2025

**Cálculo:**
```
transactionDay = 20
cutoffDay = 15

20 <= 15 → FALSE

nextMonth = julio 2025
billingPeriod = "2025-07"
```

**Resultado:**
- Billing Period: `2025-07` (julio)
- Fecha de corte: 15 de julio
- Fecha de vencimiento: 5 de agosto
- Días hasta pago: ~46 días desde la transacción

---

### Caso 3: Gasto Exactamente el Día de Corte

**Datos:**
- Tarjeta: BCI Crédito
- Día de corte: 15
- Fecha de transacción: 15 de junio 2025

**Cálculo:**
```
transactionDay = 15
cutoffDay = 15

15 <= 15 → TRUE

billingPeriod = "2025-06"
```

**Decisión de Diseño:** El día de corte se incluye en el período actual.

**Razón:** El corte ocurre "al final del día", no al inicio.

---

### Caso 4: Cambio de Año

**Datos:**
- Tarjeta: BCI Crédito
- Día de corte: 15
- Fecha de transacción: 20 de diciembre 2025

**Cálculo:**
```
transactionDay = 20
cutoffDay = 15

20 <= 15 → FALSE

nextMonth = enero 2026
billingPeriod = "2026-01"
```

**Resultado:**
- Billing Period: `2026-01` (enero 2026)
- Fecha de corte: 15 de enero 2026
- Fecha de vencimiento: 5 de febrero 2026

---

## Casos Especiales

### Meses con Menos de 31 Días

**Problema:** ¿Qué pasa si el día de corte es 31 pero el mes tiene 30 días (o 28/29)?

**Solución:** Usar el último día del mes como corte efectivo.

**Ejemplo:**
```
Tarjeta configurada: cutoffDay = 31
Transacción: 29 de abril 2025 (abril tiene 30 días)

Corte efectivo para abril: día 30

transactionDay = 29
effectiveCutoffDay = min(31, lastDayOfMonth(abril)) = 30

29 <= 30 → TRUE
billingPeriod = "2025-04"
```

**Implementación:**
```typescript
function getEffectiveCutoffDay(cutoffDay: number, month: number, year: number): number {
  const lastDay = new Date(year, month + 1, 0).getDate();
  return Math.min(cutoffDay, lastDay);
}
```

---

### Febrero (28/29 días)

**Ejemplo Año No Bisiesto:**
```
cutoffDay = 31
Transacción: 27 de febrero 2025 (28 días)

Corte efectivo: día 28

27 <= 28 → TRUE
billingPeriod = "2025-02"
```

**Ejemplo Año Bisiesto:**
```
cutoffDay = 31
Transacción: 28 de febrero 2024 (29 días)

Corte efectivo: día 29

28 <= 29 → TRUE
billingPeriod = "2024-02"
```

---

### Cambio de Día de Corte

**Escenario:** El banco cambia la fecha de corte de la tarjeta.

**Regla de Negocio:** Solo afecta gastos futuros, no recalcular históricos.

**Ejemplo:**
```
Estado Inicial:
- cutoffDay = 15
- Gasto del 10 junio: billingPeriod = "2025-06"

Cambio el 15 de junio:
- nuevo cutoffDay = 10

Efecto:
- Gasto del 10 junio: billingPeriod sigue siendo "2025-06" (no se recalcula)
- Gasto del 12 junio (nuevo): billingPeriod = "2025-07" (usa nuevo corte)
```

**Implementación:**
```typescript
// Al crear expense
const card = await prisma.creditCard.findUnique({
  where: { id: creditCardId }
});

// Usar cutoff_day actual de la tarjeta
const billingPeriod = calculateBillingPeriod(
  expenseDate,
  card.cutoffDay
);

// No recalcular expenses existentes al cambiar cutoff_day
```

---

## Cálculo de Payment Due Date

### Algoritmo

```
FUNCIÓN calculatePaymentDueDate(billingPeriod, cutoffDay, paymentDueDay):
    // Parsear billing period
    [year, month] = parsePeriod(billingPeriod) // "2025-06" -> 2025, 6
    
    // Fecha de corte del período
    cutoffDate = Date(year, month, cutoffDay)
    
    // Fecha de vencimiento = siguiente mes, día específico
    paymentMonth = month + 1
    paymentYear = year
    
    // Ajustar si es cambio de año
    SI paymentMonth > 12 ENTONCES
        paymentMonth = 1
        paymentYear = year + 1
    FIN SI
    
    paymentDueDate = Date(paymentYear, paymentMonth, paymentDueDay)
    
    RETORNAR paymentDueDate
FIN FUNCIÓN
```

### Implementación TypeScript

```typescript
function calculatePaymentDueDate(
  billingPeriod: string,
  cutoffDay: number,
  paymentDueDay: number
): Date {
  const [year, month] = billingPeriod.split('-').map(Number);
  
  // Siguiente mes después del corte
  let paymentMonth = month + 1;
  let paymentYear = year;
  
  if (paymentMonth > 12) {
    paymentMonth = 1;
    paymentYear++;
  }
  
  // Ajustar día de vencimiento si el mes no tiene suficientes días
  const lastDay = new Date(paymentYear, paymentMonth, 0).getDate();
  const effectivePaymentDay = Math.min(paymentDueDay, lastDay);
  
  return new Date(paymentYear, paymentMonth - 1, effectivePaymentDay);
}
```

---

## Integración con API

### Al Crear Expense

**Request:**
```json
{
  "expenseTypeId": "uuid",
  "amount": 15000,
  "date": "2025-06-18",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito"
}
```

**Flujo del Servicio:**
```typescript
async createExpense(dto: CreateExpenseDto) {
  // 1. Obtener tarjeta
  const card = await this.prisma.creditCard.findUnique({
    where: { id: dto.creditCardId }
  });
  
  // 2. Calcular billing period
  const billingPeriod = this.calculateBillingPeriod(
    new Date(dto.date),
    card.cutoffDay
  );
  
  // 3. Calcular payment due date
  const paymentDueDate = this.calculatePaymentDueDate(
    billingPeriod,
    card.cutoffDay,
    card.paymentDueDay
  );
  
  // 4. Crear expense
  const expense = await this.prisma.expense.create({
    data: {
      ...dto,
      billingPeriod,
      // paymentDueDate no se guarda, se calcula on-demand
    }
  });
  
  // 5. Agregar info de billing cycle en response
  return {
    ...expense,
    billingCycleInfo: {
      cutoffDate: `${billingPeriod}-${card.cutoffDay}`,
      paymentDueDate: paymentDueDate.toISOString(),
      message: `Este gasto será facturado en ${billingPeriod} y debes pagarlo el ${paymentDueDate.toLocaleDateString()}`
    }
  };
}
```

**Response:**
```json
{
  "id": "uuid",
  "amount": 15000,
  "date": "2025-06-18T00:00:00Z",
  "billingPeriod": "2025-07",
  "billingCycleInfo": {
    "cutoffDate": "2025-07-15",
    "paymentDueDate": "2025-08-05T00:00:00Z",
    "message": "Este gasto será facturado en 2025-07 y debes pagarlo el 05/08/2025"
  }
}
```

---

## Próximo Pago de Tarjeta

### Determinar Período Actual

```typescript
function getCurrentBillingPeriod(cutoffDay: number): string {
  const today = new Date();
  const currentDay = today.getDate();
  
  if (currentDay <= cutoffDay) {
    // Estamos antes del corte, el período actual es este mes
    return formatPeriod(today.getFullYear(), today.getMonth() + 1);
  } else {
    // Estamos después del corte, el período actual es el próximo mes
    const nextMonth = new Date(today.getFullYear(), today.getMonth() + 1, 1);
    return formatPeriod(nextMonth.getFullYear(), nextMonth.getMonth() + 1);
  }
}
```

### Calcular Total a Pagar

```typescript
async getNextPayment(creditCardId: string) {
  const card = await this.prisma.creditCard.findUnique({
    where: { id: creditCardId }
  });
  
  const currentPeriod = this.getCurrentBillingPeriod(card.cutoffDay);
  
  const expenses = await this.prisma.expense.findMany({
    where: {
      creditCardId,
      billingPeriod: currentPeriod
    },
    include: {
      expenseType: true
    }
  });
  
  const totalAmount = expenses.reduce((sum, exp) => sum + exp.amount, 0);
  
  const paymentDueDate = this.calculatePaymentDueDate(
    currentPeriod,
    card.cutoffDay,
    card.paymentDueDay
  );
  
  const today = new Date();
  const daysUntilPayment = Math.ceil(
    (paymentDueDate.getTime() - today.getTime()) / (1000 * 60 * 60 * 24)
  );
  
  return {
    cardName: card.name,
    currentBillingPeriod: currentPeriod,
    cutoffDate: `${currentPeriod}-${card.cutoffDay}`,
    paymentDueDate,
    daysUntilPayment,
    totalAmount,
    expensesCount: expenses.length,
    breakdown: expenses
  };
}
```

---

## Testing

### Unit Tests

```typescript
describe('Billing Cycle Calculator', () => {
  describe('calculateBillingPeriod', () => {
    it('should calculate period before cutoff', () => {
      const date = new Date('2025-06-10');
      const cutoffDay = 15;
      
      const period = calculateBillingPeriod(date, cutoffDay);
      
      expect(period).toBe('2025-06');
    });
    
    it('should calculate period after cutoff', () => {
      const date = new Date('2025-06-20');
      const cutoffDay = 15;
      
      const period = calculateBillingPeriod(date, cutoffDay);
      
      expect(period).toBe('2025-07');
    });
    
    it('should include cutoff day in current period', () => {
      const date = new Date('2025-06-15');
      const cutoffDay = 15;
      
      const period = calculateBillingPeriod(date, cutoffDay);
      
      expect(period).toBe('2025-06');
    });
    
    it('should handle year transition', () => {
      const date = new Date('2025-12-20');
      const cutoffDay = 15;
      
      const period = calculateBillingPeriod(date, cutoffDay);
      
      expect(period).toBe('2026-01');
    });
    
    it('should handle month with 30 days', () => {
      const date = new Date('2025-04-29');
      const cutoffDay = 31;
      
      // Cutoff efectivo = 30 (último día de abril)
      const period = calculateBillingPeriod(date, cutoffDay);
      
      expect(period).toBe('2025-04');
    });
    
    it('should handle February non-leap year', () => {
      const date = new Date('2025-02-27');
      const cutoffDay = 31;
      
      // Cutoff efectivo = 28 (último día de febrero 2025)
      const period = calculateBillingPeriod(date, cutoffDay);
      
      expect(period).toBe('2025-02');
    });
  });
});
```

---

## Matriz de Ejemplos

| Fecha Transacción | Cutoff Day | Billing Period | Payment Due (día 5) |
|-------------------|------------|----------------|---------------------|
| 2025-01-10        | 15         | 2025-01        | 2025-02-05          |
| 2025-01-15        | 15         | 2025-01        | 2025-02-05          |
| 2025-01-16        | 15         | 2025-02        | 2025-03-05          |
| 2025-02-28        | 31         | 2025-02        | 2025-03-05          |
| 2025-04-30        | 31         | 2025-04        | 2025-05-05          |
| 2025-12-31        | 15         | 2026-01        | 2026-02-05          |

---

## Performance Considerations

### Caching

No es necesario cachear el cálculo ya que es:
- Computación muy rápida (< 1ms)
- Determinista (mismo input → mismo output)
- Sin I/O

### Database Indexing

Crear índice en `billing_period` para queries rápidas:

```sql
CREATE INDEX idx_expenses_billing_period ON expenses(billing_period);
CREATE INDEX idx_expenses_credit_card_billing ON expenses(credit_card_id, billing_period);
```

---

## Referencias

- [Business Rules - BR-EXP-004](../../03-data-model/business-rules.md#br-exp-004-cálculo-automático-de-billing-period)
- [Endpoints - POST /expenses](../../04-api/endpoints.md#post-expenses)
- [Endpoints - GET /credit-cards/:id/next-payment](../../04-api/endpoints.md#get-credit-cardsidnext-payment)