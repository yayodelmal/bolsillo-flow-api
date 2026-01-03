# Gestión de Cuotas (Installments Management)

## Overview
Este documento describe el flujo completo de gestión de cuotas, desde la configuración inicial de un producto comprado en cuotas hasta su completación.

---

## Propósito

Gestionar productos o servicios comprados en cuotas mensuales, permitiendo:
- Configurar productos en cuotas
- Generar gastos mensuales por cada cuota
- Trackear progreso de pagos
- Auto-marcar como completado cuando se paga la última cuota
- Visualizar cuotas pendientes y pagadas

---

## Conceptos Clave

### Installment (Cuota)
Fracción mensual de un pago total dividido en N partes iguales.

**Componentes:**
- **Total Installments:** Cantidad total de cuotas (ej: 12)
- **Amount Per Installment:** Monto de cada cuota (ej: $56,249)
- **Current Installment:** Número de cuota actual (ej: 6)
- **Remaining Installments:** Cuotas que faltan por pagar (ej: 6)
- **Start Date:** Fecha de la primera cuota

### Installment Number
Formato `"N/M"` donde:
- N = número de cuota actual (1 a M)
- M = total de cuotas

**Ejemplo:** `"6/12"` significa cuota 6 de 12 totales.

### Installment Config
JSON almacenado en ExpenseType con la configuración de cuotas:

```json
{
  "totalInstallments": 12,
  "amountPerInstallment": 56249,
  "currentInstallment": 6,
  "remainingInstallments": 6,
  "startDate": "2025-02-25"
}
```

---

## Flujo Completo

### Fase 1: Configuración Inicial

#### Paso 1.1: Crear ExpenseType para Producto en Cuotas

**Request:**
```http
POST /api/expense-types
Content-Type: application/json

{
  "name": "iPhone 16 Pro",
  "categoryId": "uuid-tecnologia",
  "isRecurring": false,
  "isInstallment": true,
  "installmentConfig": {
    "totalInstallments": 12,
    "amountPerInstallment": 56249,
    "startDate": "2025-02-25"
  }
}
```

**Validaciones:**
- Si `isInstallment = true`, `installmentConfig` es obligatorio
- `totalInstallments` > 0
- `amountPerInstallment` > 0
- `startDate` es fecha válida

**Response:**
```json
{
  "id": "uuid-iphone",
  "name": "iPhone 16 Pro",
  "categoryId": "uuid-tecnologia",
  "isRecurring": false,
  "isInstallment": true,
  "installmentConfig": {
    "totalInstallments": 12,
    "amountPerInstallment": 56249,
    "currentInstallment": 0,
    "remainingInstallments": 12,
    "startDate": "2025-02-25"
  },
  "isActive": true,
  "createdAt": "2025-02-25T10:00:00Z"
}
```

**Nota:** Sistema auto-calcula `currentInstallment = 0` y `remainingInstallments = totalInstallments`.

---

### Fase 2: Registro de Cuotas Mensuales

#### Paso 2.1: Crear Primer Pago de Cuota

**Request:**
```http
POST /api/expenses
Content-Type: application/json

{
  "expenseTypeId": "uuid-iphone",
  "amount": 56249,
  "date": "2025-02-25",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "installmentNumber": "1/12"
}
```

**Validaciones:**
- `installmentNumber` debe tener formato `"N/M"`
- N debe ser <= M
- M debe coincidir con `totalInstallments` del ExpenseType
- `amount` debe coincidir con `amountPerInstallment`

**Response:**
```json
{
  "id": "uuid-expense-1",
  "expenseTypeId": "uuid-iphone",
  "amount": 56249,
  "date": "2025-02-25T00:00:00Z",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "billingPeriod": "2025-03",
  "installmentNumber": "1/12",
  "isRecurring": false,
  "createdAt": "2025-02-25T12:00:00Z"
}
```

---

#### Paso 2.2: Actualizar ExpenseType (Automático)

**Proceso interno del servicio:**
```typescript
async createExpense(dto: CreateExpenseDto) {
  // 1. Crear expense
  const expense = await this.prisma.expense.create({ data: dto });
  
  // 2. Si es cuota, actualizar expense type
  if (dto.installmentNumber) {
    const [current, total] = dto.installmentNumber.split('/').map(Number);
    
    const expenseType = await this.prisma.expenseType.findUnique({
      where: { id: dto.expenseTypeId }
    });
    
    await this.prisma.expenseType.update({
      where: { id: dto.expenseTypeId },
      data: {
        installmentConfig: {
          ...expenseType.installmentConfig,
          currentInstallment: current,
          remainingInstallments: total - current
        }
      }
    });
    
    // 3. Si es última cuota, marcar como completado
    if (current === total) {
      await this.markAsCompleted(dto.expenseTypeId);
    }
  }
  
  return expense;
}
```

**Estado del ExpenseType después de crear cuota 1:**
```json
{
  "installmentConfig": {
    "totalInstallments": 12,
    "amountPerInstallment": 56249,
    "currentInstallment": 1,
    "remainingInstallments": 11,
    "startDate": "2025-02-25"
  }
}
```

---

#### Paso 2.3: Continuar Pagando Cuotas

**Cuota 2:**
```http
POST /api/expenses
{
  "expenseTypeId": "uuid-iphone",
  "amount": 56249,
  "date": "2025-03-25",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "installmentNumber": "2/12"
}
```

**Cuota 3:**
```http
POST /api/expenses
{
  "expenseTypeId": "uuid-iphone",
  "amount": 56249,
  "date": "2025-04-25",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "installmentNumber": "3/12"
}
```

... y así sucesivamente hasta la cuota 12.

**Progreso del InstallmentConfig:**
```
Cuota 1: currentInstallment = 1, remainingInstallments = 11
Cuota 2: currentInstallment = 2, remainingInstallments = 10
Cuota 3: currentInstallment = 3, remainingInstallments = 9
...
Cuota 11: currentInstallment = 11, remainingInstallments = 1
```

---

### Fase 3: Completación Automática

#### Paso 3.1: Pagar Última Cuota

**Request:**
```http
POST /api/expenses
{
  "expenseTypeId": "uuid-iphone",
  "amount": 56249,
  "date": "2026-01-25",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "installmentNumber": "12/12"
}
```

**Proceso interno:**
```typescript
async createExpense(dto: CreateExpenseDto) {
  const expense = await this.prisma.expense.create({ data: dto });
  
  if (dto.installmentNumber) {
    const [current, total] = dto.installmentNumber.split('/').map(Number);
    
    // Actualizar config
    await this.prisma.expenseType.update({
      where: { id: dto.expenseTypeId },
      data: {
        installmentConfig: {
          ...config,
          currentInstallment: 12,
          remainingInstallments: 0
        }
      }
    });
    
    // TRIGGER: current === total
    if (current === total) {
      await this.prisma.expenseType.update({
        where: { id: dto.expenseTypeId },
        data: {
          isCompleted: true,    // ← Marca como completado
          isActive: false,      // ← Desactiva (no aparece en listados)
          completedAt: new Date()
        }
      });
    }
  }
  
  return expense;
}
```

---

#### Paso 3.2: Estado Final del ExpenseType

**Request:**
```http
GET /api/expense-types/uuid-iphone
```

**Response:**
```json
{
  "id": "uuid-iphone",
  "name": "iPhone 16 Pro",
  "categoryId": "uuid-tecnologia",
  "isRecurring": false,
  "isInstallment": true,
  "installmentConfig": {
    "totalInstallments": 12,
    "amountPerInstallment": 56249,
    "currentInstallment": 12,
    "remainingInstallments": 0,
    "startDate": "2025-02-25"
  },
  "isCompleted": true,
  "isActive": false,
  "completedAt": "2026-01-25T12:00:00Z",
  "createdAt": "2025-02-25T10:00:00Z",
  "updatedAt": "2026-01-25T12:00:00Z"
}
```

**Nota:** 
- `isCompleted = true`
- `isActive = false` (no aparece en listados de expense types activos)
- `completedAt` registra cuándo se completó

---

### Fase 4: Consultas y Reportes

#### Consultar Cuotas Pagadas de un Producto

**Request:**
```http
GET /api/expenses?expenseTypeId=uuid-iphone&sortBy=date&sortOrder=asc
```

**Response:**
```json
{
  "data": [
    {
      "id": "uuid-exp-1",
      "amount": 56249,
      "date": "2025-02-25",
      "installmentNumber": "1/12",
      "billingPeriod": "2025-03"
    },
    {
      "id": "uuid-exp-2",
      "amount": 56249,
      "date": "2025-03-25",
      "installmentNumber": "2/12",
      "billingPeriod": "2025-04"
    },
    ...
    {
      "id": "uuid-exp-12",
      "amount": 56249,
      "date": "2026-01-25",
      "installmentNumber": "12/12",
      "billingPeriod": "2026-02"
    }
  ],
  "meta": {
    "total": 12
  }
}
```

---

#### Vista Anual de Cuotas

**Request:**
```http
GET /api/expenses/annual-installments?year=2025
```

**Response:**
```json
{
  "year": 2025,
  "installments": [
    {
      "expenseTypeId": "uuid-iphone",
      "expenseTypeName": "iPhone 16 Pro",
      "totalInstallments": 12,
      "currentInstallment": 12,
      "remainingInstallments": 0,
      "amountPerInstallment": 56249,
      "totalPaid": 674988,
      "totalRemaining": 0,
      "isCompleted": true,
      "payments": [
        {
          "installmentNumber": "1/12",
          "date": "2025-02-25",
          "amount": 56249,
          "billingPeriod": "2025-03"
        },
        ...
      ]
    },
    {
      "expenseTypeId": "uuid-monitor",
      "expenseTypeName": "Monitor Samsung",
      "totalInstallments": 6,
      "currentInstallment": 2,
      "remainingInstallments": 4,
      "amountPerInstallment": 106404,
      "totalPaid": 212808,
      "totalRemaining": 425616,
      "isCompleted": false,
      "payments": [...]
    }
  ],
  "summary": {
    "totalProducts": 2,
    "totalPaid": 887796,
    "totalRemaining": 425616,
    "completedProducts": 1,
    "activeProducts": 1
  }
}
```

---

## Validaciones Importantes

### VL-INST-001: Formato de Installment Number

**Regla:** Debe ser `"N/M"` donde N ≤ M.

**Validación:**
```typescript
function validateInstallmentNumber(installmentNumber: string, totalInstallments: number): void {
  const regex = /^(\d+)\/(\d+)$/;
  const match = installmentNumber.match(regex);
  
  if (!match) {
    throw new BadRequestException('Invalid installment number format. Expected "N/M"');
  }
  
  const [_, current, total] = match.map(Number);
  
  if (current > total) {
    throw new BadRequestException('Current installment cannot exceed total installments');
  }
  
  if (total !== totalInstallments) {
    throw new BadRequestException(
      `Total installments in number (${total}) does not match expense type config (${totalInstallments})`
    );
  }
}
```

---

### VL-INST-002: Monto Debe Coincidir

**Regla:** El monto del expense debe ser igual a `amountPerInstallment`.

**Validación:**
```typescript
async validateInstallmentAmount(
  expenseTypeId: string,
  amount: number
): Promise<void> {
  const expenseType = await this.prisma.expenseType.findUnique({
    where: { id: expenseTypeId }
  });
  
  const expectedAmount = expenseType.installmentConfig.amountPerInstallment;
  
  if (amount !== expectedAmount) {
    throw new BadRequestException(
      `Amount (${amount}) does not match expected installment amount (${expectedAmount})`
    );
  }
}
```

**Nota:** Permite pequeñas variaciones si hay redondeo (< $1).

---

### VL-INST-003: No Duplicar Cuota del Mismo Mes

**Regla:** No se puede pagar la misma cuota dos veces en el mismo mes.

**Validación:**
```typescript
async checkDuplicateInstallment(
  expenseTypeId: string,
  installmentNumber: string,
  date: Date
): Promise<void> {
  const monthStart = new Date(date.getFullYear(), date.getMonth(), 1);
  const monthEnd = new Date(date.getFullYear(), date.getMonth() + 1, 0);
  
  const existing = await this.prisma.expense.findFirst({
    where: {
      expenseTypeId,
      installmentNumber,
      date: {
        gte: monthStart,
        lte: monthEnd
      }
    }
  });
  
  if (existing) {
    throw new UnprocessableEntityException(
      `Installment ${installmentNumber} already paid in this month`
    );
  }
}
```

---

### VL-INST-004: Expense Type Debe Ser Installment

**Regla:** Solo se puede usar `installmentNumber` si el ExpenseType tiene `isInstallment = true`.

**Validación:**
```typescript
async validateExpenseTypeIsInstallment(expenseTypeId: string): Promise<void> {
  const expenseType = await this.prisma.expenseType.findUnique({
    where: { id: expenseTypeId }
  });
  
  if (!expenseType.isInstallment) {
    throw new BadRequestException(
      'Cannot set installmentNumber on non-installment expense type'
    );
  }
}
```

---

## Casos Especiales

### Caso 1: Variación de Monto por Interés

**Problema:** Algunos meses la cuota varía ligeramente por ajustes de interés.

**Solución:**
Permitir que `amount` difiera de `amountPerInstallment` si:
- La diferencia es < 1% del monto configurado
- Se registra en notas la razón

**Implementación:**
```typescript
const tolerance = config.amountPerInstallment * 0.01; // 1%
const diff = Math.abs(amount - config.amountPerInstallment);

if (diff > tolerance) {
  throw new BadRequestException('Amount exceeds tolerance threshold');
}

if (diff > 0) {
  expense.notes = `Amount adjusted: ${amount} vs expected ${config.amountPerInstallment}`;
}
```

---

### Caso 2: Pago Anticipado de Varias Cuotas

**Escenario:** Usuario paga cuotas 10, 11 y 12 juntas.

**Opción A (Recomendada):** Crear 3 expenses separados con la misma fecha.

```http
POST /api/expenses
{"installmentNumber": "10/12", "date": "2025-11-15", ...}

POST /api/expenses
{"installmentNumber": "11/12", "date": "2025-11-15", ...}

POST /api/expenses
{"installmentNumber": "12/12", "date": "2025-11-15", ...}
```

**Opción B:** Crear endpoint especial `POST /expenses/pay-multiple-installments`.

**Decisión:** Opción A para MVP. Opción B para backlog.

---

### Caso 3: Cuota Faltante (No Pagada)

**Problema:** Usuario olvida registrar cuota 5, pero registra cuota 6.

**Validación (Opcional):**
```typescript
// Advertir si se salta una cuota
const currentInstallment = config.currentInstallment;
const [newCurrent] = installmentNumber.split('/').map(Number);

if (newCurrent !== currentInstallment + 1) {
  console.warn(
    `Warning: Expected installment ${currentInstallment + 1}, but received ${newCurrent}`
  );
  // No bloquear, solo advertir
}
```

**Decisión:** Permitir saltos. Usuario es responsable de registrar todas las cuotas.

---

### Caso 4: Eliminar Cuota Ya Pagada

**Problema:** Usuario registró mal una cuota y quiere eliminarla.

**Solución:**
```typescript
async deleteExpense(expenseId: string): Promise<void> {
  const expense = await this.prisma.expense.findUnique({
    where: { id: expenseId }
  });
  
  // Si es cuota, actualizar expense type
  if (expense.installmentNumber) {
    const [current, total] = expense.installmentNumber.split('/').map(Number);
    
    await this.prisma.expenseType.update({
      where: { id: expense.expenseTypeId },
      data: {
        installmentConfig: {
          ...config,
          currentInstallment: current - 1,
          remainingInstallments: total - (current - 1)
        },
        isCompleted: false, // Revertir completado si era última cuota
        isActive: true
      }
    });
  }
  
  await this.prisma.expense.delete({ where: { id: expenseId } });
}
```

---

## Helpers Útiles

### Calcular Total Pagado

```typescript
function calculateTotalPaid(config: InstallmentConfig): number {
  return config.currentInstallment * config.amountPerInstallment;
}
```

### Calcular Total Restante

```typescript
function calculateTotalRemaining(config: InstallmentConfig): number {
  return config.remainingInstallments * config.amountPerInstallment;
}
```

### Calcular Porcentaje Completado

```typescript
function calculateCompletionPercentage(config: InstallmentConfig): number {
  return (config.currentInstallment / config.totalInstallments) * 100;
}
```

### Próxima Cuota Esperada

```typescript
function getNextInstallmentNumber(config: InstallmentConfig): string | null {
  if (config.remainingInstallments === 0) {
    return null; // Ya completado
  }
  
  const next = config.currentInstallment + 1;
  return `${next}/${config.totalInstallments}`;
}
```

### Fecha Esperada de Próxima Cuota

```typescript
function getNextInstallmentDate(config: InstallmentConfig): Date | null {
  if (config.remainingInstallments === 0) {
    return null;
  }
  
  const startDate = new Date(config.startDate);
  const monthsToAdd = config.currentInstallment;
  
  return new Date(
    startDate.getFullYear(),
    startDate.getMonth() + monthsToAdd,
    startDate.getDate()
  );
}
```

---

## Testing

### Unit Tests

```typescript
describe('Installment Management', () => {
  describe('validateInstallmentNumber', () => {
    it('should accept valid format', () => {
      expect(() => validateInstallmentNumber('6/12', 12)).not.toThrow();
    });
    
    it('should reject invalid format', () => {
      expect(() => validateInstallmentNumber('6-12', 12)).toThrow();
    });
    
    it('should reject current > total', () => {
      expect(() => validateInstallmentNumber('13/12', 12)).toThrow();
    });
  });
  
  describe('markAsCompleted', () => {
    it('should mark as completed when last installment paid', async () => {
      const expense = await service.createExpense({
        installmentNumber: '12/12',
        ...
      });
      
      const expenseType = await prisma.expenseType.findUnique({
        where: { id: expense.expenseTypeId }
      });
      
      expect(expenseType.isCompleted).toBe(true);
      expect(expenseType.isActive).toBe(false);
    });
  });
});
```

---

## Referencias

- [Business Rules - BR-ET-003](../../03-data-model/business-rules.md#br-et-003-auto-completado-de-cuotas)
- [Business Rules - BR-EXP-007](../../03-data-model/business-rules.md#br-exp-007-formato-de-número-de-cuota)
- [Endpoints - POST /expenses](../../04-api/endpoints.md#post-expenses)
- [Endpoints - GET /expenses/annual-installments](../../04-api/endpoints.md#get-expensesannual-installments)