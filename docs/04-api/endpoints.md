# Endpoints de la API

## Overview
Este documento describe todos los endpoints REST de Bolsillo Flow API, organizados por dominio.

**Base URL:** `http://localhost:3000/api`  
**Formato de respuesta:** JSON  
**Versionado:** No implementado en MVP (considerar `/api/v1` para futuro)

---

## Convenciones Generales

### Códigos de Estado HTTP

| Código | Significado | Cuándo |
|--------|-------------|--------|
| 200 | OK | Operación exitosa (GET, PUT, PATCH) |
| 201 | Created | Recurso creado exitosamente (POST) |
| 204 | No Content | Operación exitosa sin contenido (DELETE) |
| 400 | Bad Request | Validación de datos falló |
| 404 | Not Found | Recurso no encontrado |
| 422 | Unprocessable Entity | Regla de negocio violada |
| 500 | Internal Server Error | Error del servidor |

---

### Formato de Errores

```json
{
  "statusCode": 400,
  "message": ["name should not be empty", "amount must be a positive number"],
  "error": "Bad Request"
}
```

**Errores de negocio (422):**
```json
{
  "statusCode": 422,
  "message": "Cannot delete category with active expense types",
  "error": "Unprocessable Entity"
}
```

---

### Query Parameters Comunes

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `includeDeleted` | boolean | Incluir registros soft-deleted |
| `includeInactive` | boolean | Incluir registros inactivos |
| `page` | number | Número de página (paginación) |
| `limit` | number | Registros por página |
| `sortBy` | string | Campo para ordenar |
| `sortOrder` | asc\|desc | Orden ascendente o descendente |

---

## Financial Domain

### Categories

#### `GET /categories`
Lista todas las categorías.

**Query Parameters:**
- `includeDeleted` (boolean): Incluir soft-deleted
- `parentOnly` (boolean): Solo categorías padre
- `includeInactive` (boolean): Incluir inactivas

**Response 200:**
```json
[
  {
    "id": "uuid",
    "name": "Departamento",
    "description": "Gastos del departamento",
    "color": "#FF5733",
    "parentId": null,
    "isActive": true,
    "createdAt": "2025-01-02T10:00:00Z",
    "updatedAt": "2025-01-02T10:00:00Z",
    "deletedAt": null,
    "children": [
      {
        "id": "uuid",
        "name": "Servicios Básicos",
        "parentId": "uuid-departamento",
        ...
      }
    ]
  }
]
```

---

#### `GET /categories/:id`
Obtiene una categoría por ID.

**Path Parameters:**
- `id` (uuid): ID de la categoría

**Query Parameters:**
- `includeDeleted` (boolean): Permitir obtener soft-deleted

**Response 200:**
```json
{
  "id": "uuid",
  "name": "Departamento",
  "description": "Gastos del departamento",
  "color": "#FF5733",
  "parentId": null,
  "isActive": true,
  "createdAt": "2025-01-02T10:00:00Z",
  "updatedAt": "2025-01-02T10:00:00Z",
  "deletedAt": null,
  "parent": null,
  "children": [...]
}
```

**Response 404:**
```json
{
  "statusCode": 404,
  "message": "Category not found",
  "error": "Not Found"
}
```

---

#### `GET /categories/:id/children`
Obtiene las subcategorías de una categoría.

**Path Parameters:**
- `id` (uuid): ID de la categoría padre

**Response 200:**
```json
[
  {
    "id": "uuid",
    "name": "Servicios Básicos",
    "parentId": "uuid-departamento",
    "isActive": true,
    ...
  }
]
```

---

#### `POST /categories`
Crea una nueva categoría.

**Request Body:**
```json
{
  "name": "Transporte",
  "description": "Gastos de movilización",
  "color": "#3498DB",
  "parentId": null
}
```

**Validaciones:**
- `name`: Required, 3-50 caracteres
- `description`: Optional, max 200 caracteres
- `color`: Optional, formato hex (#RRGGBB)
- `parentId`: Optional, UUID válido

**Response 201:**
```json
{
  "id": "uuid-generated",
  "name": "Transporte",
  "description": "Gastos de movilización",
  "color": "#3498DB",
  "parentId": null,
  "isActive": true,
  "createdAt": "2025-01-02T10:00:00Z",
  "updatedAt": "2025-01-02T10:00:00Z",
  "deletedAt": null
}
```

**Response 400:**
```json
{
  "statusCode": 400,
  "message": ["name should not be empty"],
  "error": "Bad Request"
}
```

**Response 422:**
```json
{
  "statusCode": 422,
  "message": "Cannot create category with more than 2 levels of hierarchy",
  "error": "Unprocessable Entity"
}
```

---

#### `PUT /categories/:id`
Actualiza una categoría completa.

**Path Parameters:**
- `id` (uuid): ID de la categoría

**Request Body:**
```json
{
  "name": "Transporte Actualizado",
  "description": "Nueva descripción",
  "color": "#2ECC71",
  "parentId": null,
  "isActive": true
}
```

**Response 200:**
```json
{
  "id": "uuid",
  "name": "Transporte Actualizado",
  ...
}
```

---

#### `PATCH /categories/:id`
Actualiza parcialmente una categoría.

**Request Body:**
```json
{
  "color": "#E74C3C"
}
```

**Response 200:** (igual que PUT)

---

#### `DELETE /categories/:id`
Soft-delete de una categoría.

**Path Parameters:**
- `id` (uuid): ID de la categoría

**Response 204:** No Content

**Response 422:**
```json
{
  "statusCode": 422,
  "message": "Cannot delete category with active expense types",
  "error": "Unprocessable Entity"
}
```

---

#### `PATCH /categories/:id/restore`
Restaura una categoría soft-deleted.

**Response 200:**
```json
{
  "id": "uuid",
  "name": "Categoría Restaurada",
  "deletedAt": null,
  ...
}
```

---

### Budgets

#### `GET /budgets`
Lista todos los presupuestos.

**Query Parameters:**
- `categoryId` (uuid): Filtrar por categoría
- `period` (YYYY-MM): Filtrar por período
- `year` (number): Filtrar por año

**Response 200:**
```json
[
  {
    "id": "uuid",
    "categoryId": "uuid-departamento",
    "amount": 600000,
    "period": "2025-07",
    "createdAt": "2025-01-02T10:00:00Z",
    "updatedAt": "2025-01-02T10:00:00Z",
    "category": {
      "id": "uuid",
      "name": "Departamento"
    }
  }
]
```

---

#### `GET /budgets/:id`
Obtiene un presupuesto por ID.

**Response 200:**
```json
{
  "id": "uuid",
  "categoryId": "uuid-departamento",
  "amount": 600000,
  "period": "2025-07",
  "createdAt": "2025-01-02T10:00:00Z",
  "updatedAt": "2025-01-02T10:00:00Z",
  "category": {...}
}
```

---

#### `POST /budgets`
Crea o actualiza un presupuesto.

**Request Body:**
```json
{
  "categoryId": "uuid-departamento",
  "amount": 600000,
  "period": "2025-07"
}
```

**Validaciones:**
- `categoryId`: Required, UUID válido, debe ser categoría padre
- `amount`: Required, > 0
- `period`: Required, formato YYYY-MM

**Response 201:**
```json
{
  "id": "uuid",
  "categoryId": "uuid-departamento",
  "amount": 600000,
  "period": "2025-07",
  "createdAt": "2025-01-02T10:00:00Z",
  "updatedAt": "2025-01-02T10:00:00Z"
}
```

**Nota:** Si ya existe presupuesto para esa categoría/período, se actualiza (upsert).

---

#### `GET /budgets/category/:categoryId/period/:period`
Obtiene presupuesto de una categoría en un período específico.

**Path Parameters:**
- `categoryId` (uuid)
- `period` (YYYY-MM)

**Response 200:**
```json
{
  "id": "uuid",
  "categoryId": "uuid",
  "amount": 600000,
  "period": "2025-07",
  ...
}
```

**Response 404:**
```json
{
  "statusCode": 404,
  "message": "No budget defined for this category in this period",
  "error": "Not Found"
}
```

---

#### `GET /budgets/category/:categoryId/history`
Historial de presupuestos de una categoría.

**Query Parameters:**
- `startPeriod` (YYYY-MM): Período inicial
- `endPeriod` (YYYY-MM): Período final

**Response 200:**
```json
[
  {
    "id": "uuid",
    "period": "2025-07",
    "amount": 600000
  },
  {
    "id": "uuid",
    "period": "2025-06",
    "amount": 550000
  }
]
```

---

#### `DELETE /budgets/:id`
Elimina un presupuesto (hard delete).

**Nota:** Generalmente no se recomienda, pero disponible para correcciones.

**Response 204:** No Content

---

### Expense Types

#### `GET /expense-types`
Lista todos los tipos de gasto.

**Query Parameters:**
- `categoryId` (uuid): Filtrar por categoría
- `includeDeleted` (boolean)
- `includeInactive` (boolean)
- `isRecurring` (boolean): Solo recurrentes
- `isInstallment` (boolean): Solo cuotas

**Response 200:**
```json
[
  {
    "id": "uuid",
    "name": "Arriendo",
    "categoryId": "uuid-departamento",
    "isRecurring": true,
    "isInstallment": false,
    "installmentConfig": null,
    "isActive": true,
    "createdAt": "2025-01-02T10:00:00Z",
    "updatedAt": "2025-01-02T10:00:00Z",
    "deletedAt": null,
    "category": {
      "id": "uuid",
      "name": "Departamento"
    }
  }
]
```

---

#### `GET /expense-types/:id`
Obtiene un tipo de gasto por ID.

**Response 200:**
```json
{
  "id": "uuid",
  "name": "iPhone 16 Pro",
  "categoryId": "uuid-tecnologia",
  "isRecurring": false,
  "isInstallment": true,
  "installmentConfig": {
    "totalInstallments": 12,
    "amountPerInstallment": 56249,
    "currentInstallment": 3,
    "remainingInstallments": 9,
    "startDate": "2025-02-25"
  },
  "isActive": true,
  ...
}
```

---

#### `POST /expense-types`
Crea un tipo de gasto.

**Request Body (Simple):**
```json
{
  "name": "Arriendo",
  "categoryId": "uuid-departamento",
  "isRecurring": true,
  "isInstallment": false
}
```

**Request Body (Con cuotas):**
```json
{
  "name": "Monitor Samsung",
  "categoryId": "uuid-tecnologia",
  "isRecurring": false,
  "isInstallment": true,
  "installmentConfig": {
    "totalInstallments": 6,
    "amountPerInstallment": 106404,
    "startDate": "2025-10-15"
  }
}
```

**Validaciones:**
- `name`: Required, 3-100 caracteres
- `categoryId`: Required, UUID válido
- `isRecurring`: Optional, default false
- `isInstallment`: Optional, default false
- `installmentConfig`: Required si isInstallment = true

**Response 201:**
```json
{
  "id": "uuid",
  "name": "Monitor Samsung",
  "categoryId": "uuid-tecnologia",
  "isInstallment": true,
  "installmentConfig": {
    "totalInstallments": 6,
    "amountPerInstallment": 106404,
    "currentInstallment": 0,
    "remainingInstallments": 6,
    "startDate": "2025-10-15"
  },
  ...
}
```

---

#### `PUT /expense-types/:id`
Actualiza un tipo de gasto.

**Request Body:**
```json
{
  "name": "Arriendo Actualizado",
  "categoryId": "uuid-departamento",
  "isActive": true
}
```

**Response 200:** (igual estructura)

---

#### `DELETE /expense-types/:id`
Soft-delete de un tipo de gasto.

**Response 204:** No Content

**Response 422:**
```json
{
  "statusCode": 422,
  "message": "Cannot delete expense type with associated expenses",
  "error": "Unprocessable Entity"
}
```

---

### Expenses

#### `GET /expenses`
Lista gastos con filtros.

**Query Parameters:**
- `month` (YYYY-MM): Filtrar por mes
- `startDate` (YYYY-MM-DD): Fecha desde
- `endDate` (YYYY-MM-DD): Fecha hasta
- `categoryId` (uuid): Filtrar por categoría
- `expenseTypeId` (uuid): Filtrar por tipo
- `paymentType` (credit|debit|transfer|cash): Filtrar por método
- `creditCardId` (uuid): Filtrar por tarjeta crédito
- `debitCardId` (uuid): Filtrar por tarjeta débito
- `billingPeriod` (YYYY-MM): Filtrar por período facturación
- `minAmount` (number): Monto mínimo
- `maxAmount` (number): Monto máximo
- `page` (number): Paginación
- `limit` (number): Registros por página

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid",
      "expenseTypeId": "uuid",
      "amount": 400000,
      "date": "2025-07-01T00:00:00Z",
      "paymentType": "debit",
      "creditCardId": null,
      "debitCardId": "uuid-cuenta-rut",
      "billingPeriod": null,
      "isRecurring": true,
      "installmentNumber": null,
      "notes": "Pago mensual de arriendo",
      "createdAt": "2025-07-01T10:00:00Z",
      "updatedAt": "2025-07-01T10:00:00Z",
      "deletedAt": null,
      "expenseType": {
        "id": "uuid",
        "name": "Arriendo",
        "category": {
          "id": "uuid",
          "name": "Departamento"
        }
      },
      "debitCard": {
        "id": "uuid",
        "name": "Cuenta RUT"
      }
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "totalPages": 8
  }
}
```

---

#### `GET /expenses/:id`
Obtiene un gasto por ID.

**Response 200:**
```json
{
  "id": "uuid",
  "expenseTypeId": "uuid",
  "amount": 56249,
  "date": "2025-07-25T00:00:00Z",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "debitCardId": null,
  "billingPeriod": "2025-08",
  "isRecurring": false,
  "installmentNumber": "6/12",
  "notes": null,
  "expenseType": {...},
  "creditCard": {...}
}
```

---

#### `POST /expenses`
Crea un gasto.

**Request Body (Débito):**
```json
{
  "expenseTypeId": "uuid-arriendo",
  "amount": 400000,
  "date": "2025-07-01",
  "paymentType": "debit",
  "debitCardId": "uuid-cuenta-rut",
  "notes": "Pago mensual"
}
```

**Request Body (Crédito):**
```json
{
  "expenseTypeId": "uuid-uber-eats",
  "amount": 15000,
  "date": "2025-07-18",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito"
}
```

**Request Body (Cuota):**
```json
{
  "expenseTypeId": "uuid-iphone",
  "amount": 56249,
  "date": "2025-07-25",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "installmentNumber": "6/12"
}
```

**Validaciones:**
- `expenseTypeId`: Required, UUID válido
- `amount`: Required, > 0
- `date`: Required, fecha válida
- `paymentType`: Required, enum válido
- `creditCardId`: Required si paymentType = credit
- `debitCardId`: Required si paymentType = debit
- `installmentNumber`: Opcional, formato "N/M"

**Response 201:**
```json
{
  "id": "uuid",
  "expenseTypeId": "uuid",
  "amount": 15000,
  "date": "2025-07-18T00:00:00Z",
  "paymentType": "credit",
  "creditCardId": "uuid-bci-credito",
  "billingPeriod": "2025-08",
  "billingCycleInfo": {
    "cutoffDate": "2025-07-15",
    "paymentDueDate": "2025-08-05",
    "message": "Este gasto será facturado en agosto 2025 y debes pagarlo el 5 de septiembre"
  },
  ...
}
```

---

#### `PUT /expenses/:id`
Actualiza un gasto completo.

**Request Body:**
```json
{
  "amount": 420000,
  "notes": "Arriendo aumentó de precio"
}
```

**Response 200:** (igual estructura)

---

#### `DELETE /expenses/:id`
Elimina un gasto (soft-delete).

**Response 204:** No Content

---

#### `GET /expenses/summary`
Resumen de gastos con agrupaciones.

**Query Parameters:**
- `month` (YYYY-MM): Required
- `groupBy` (category|expenseType|paymentType): Required

**Response 200 (groupBy=category):**
```json
{
  "month": "2025-07",
  "totalSpent": 1009209,
  "groups": [
    {
      "categoryId": "uuid-departamento",
      "categoryName": "Departamento",
      "total": 598205,
      "count": 5,
      "percentage": 59.2
    },
    {
      "categoryId": "uuid-suscripciones",
      "categoryName": "Suscripciones",
      "total": 35868,
      "count": 6,
      "percentage": 3.6
    }
  ]
}
```

---

#### `GET /expenses/monthly-report`
Reporte mensual completo con presupuestos.

**Query Parameters:**
- `month` (YYYY-MM): Required

**Response 200:**
```json
{
  "month": "2025-07",
  "totalSpent": 1009209,
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
        }
      ]
    }
  ]
}
```

---

#### `GET /expenses/annual-installments`
Vista anual de cuotas.

**Query Parameters:**
- `year` (number): Required

**Response 200:**
```json
{
  "year": 2025,
  "installments": [
    {
      "expenseTypeId": "uuid-iphone",
      "expenseTypeName": "iPhone 16 Pro",
      "totalInstallments": 12,
      "currentInstallment": 6,
      "remainingInstallments": 6,
      "amountPerInstallment": 56249,
      "totalPaid": 337494,
      "totalRemaining": 337494,
      "payments": [
        {
          "installmentNumber": "1/12",
          "date": "2025-02-25",
          "amount": 56249,
          "billingPeriod": "2025-03"
        },
        ...
      ]
    }
  ]
}
```

---

## Payment Methods Domain

### Credit Cards

#### `GET /credit-cards`
Lista todas las tarjetas de crédito.

**Query Parameters:**
- `includeDeleted` (boolean)
- `includeInactive` (boolean)

**Response 200:**
```json
[
  {
    "id": "uuid",
    "name": "BCI Crédito",
    "bank": "BCI",
    "lastFourDigits": "1234",
    "cutoffDay": 15,
    "paymentDueDay": 5,
    "cupo": 2000000,
    "isActive": true,
    "createdAt": "2025-01-02T10:00:00Z",
    "updatedAt": "2025-01-02T10:00:00Z",
    "deletedAt": null
  }
]
```

---

#### `GET /credit-cards/:id`
Obtiene una tarjeta por ID.

**Response 200:** (igual estructura que lista)

---

#### `POST /credit-cards`
Crea una tarjeta de crédito.

**Request Body:**
```json
{
  "name": "BCI Crédito",
  "bank": "BCI",
  "lastFourDigits": "1234",
  "cutoffDay": 15,
  "paymentDueDay": 5,
  "cupo": 2000000
}
```

**Validaciones:**
- `name`: Required, 3-50 caracteres
- `bank`: Required, 3-50 caracteres
- `lastFourDigits`: Optional, exactamente 4 dígitos
- `cutoffDay`: Required, 1-31
- `paymentDueDay`: Required, 1-31
- `cupo`: Optional, > 0 si se proporciona

**Response 201:**
```json
{
  "id": "uuid",
  "name": "BCI Crédito",
  "bank": "BCI",
  ...
}
```

---

#### `PUT /credit-cards/:id`
Actualiza una tarjeta.

**Request Body:**
```json
{
  "name": "BCI Crédito Principal",
  "cutoffDay": 10,
  "cupo": 2500000
}
```

**Nota:** Cambio de `cutoffDay` solo afecta gastos futuros.

**Response 200:** (igual estructura)

---

#### `DELETE /credit-cards/:id`
Soft-delete de una tarjeta.

**Response 204:** No Content

**Response 422:**
```json
{
  "statusCode": 422,
  "message": "Cannot delete credit card with associated expenses",
  "error": "Unprocessable Entity"
}
```

---

#### `GET /credit-cards/:id/next-payment`
Calcula el próximo pago de la tarjeta.

**Response 200:**
```json
{
  "cardName": "BCI Crédito",
  "currentBillingPeriod": "2025-07",
  "cutoffDate": "2025-07-15",
  "paymentDueDate": "2025-08-05",
  "daysUntilPayment": 23,
  "totalAmount": 856249,
  "expensesCount": 15,
  "breakdown": [
    {
      "expenseId": "uuid",
      "expenseTypeName": "iPhone 16 Pro",
      "amount": 56249,
      "date": "2025-07-25",
      "isInstallment": true,
      "installmentInfo": "6/12"
    },
    {
      "expenseId": "uuid",
      "expenseTypeName": "Uber Eats",
      "amount": 153259,
      "date": "2025-07-18",
      "isInstallment": false,
      "installmentInfo": null
    }
  ]
}
```

---

#### `GET /credit-cards/:id/billing-history`
Historial de períodos de facturación.

**Query Parameters:**
- `limit` (number): Últimos N períodos

**Response 200:**
```json
[
  {
    "billingPeriod": "2025-07",
    "cutoffDate": "2025-07-15",
    "paymentDueDate": "2025-08-05",
    "totalAmount": 856249,
    "expensesCount": 15,
    "paid": false
  },
  {
    "billingPeriod": "2025-06",
    "cutoffDate": "2025-06-15",
    "paymentDueDate": "2025-07-05",
    "totalAmount": 780340,
    "expensesCount": 12,
    "paid": true
  }
]
```

---

### Debit Cards

#### `GET /debit-cards`
Lista todas las tarjetas de débito.

**Response 200:**
```json
[
  {
    "id": "uuid",
    "name": "Cuenta RUT",
    "bank": "Banco Estado",
    "currentBalance": null,
    "isActive": true,
    "createdAt": "2025-01-02T10:00:00Z",
    "updatedAt": "2025-01-02T10:00:00Z",
    "deletedAt": null
  }
]
```

---

#### `POST /debit-cards`
Crea una tarjeta de débito.

**Request Body:**
```json
{
  "name": "BCI Débito",
  "bank": "BCI",
  "currentBalance": 500000
}
```

**Validaciones:**
- `name`: Required, 3-50 caracteres
- `bank`: Required, 3-50 caracteres
- `currentBalance`: Optional, puede ser cualquier número

**Response 201:** (igual estructura que lista)

---

#### `PUT /debit-cards/:id`
Actualiza una tarjeta de débito.

**Response 200:** (igual estructura)

---

#### `DELETE /debit-cards/:id`
Soft-delete de una tarjeta.

**Response 204:** No Content

---

## Automation Domain

### Recurring Expenses

#### `GET /recurring-expenses`
Lista gastos recurrentes.

**Query Parameters:**
- `isActive` (boolean): Filtrar por activos/inactivos

**Response 200:**
```json
[
  {
    "id": "uuid",
    "expenseTypeId": "uuid-arriendo",
    "amount": 400000,
    "dayOfMonth": 1,
    "paymentType": "debit",
    "creditCardId": null,
    "debitCardId": "uuid-cuenta-rut",
    "frequency": "monthly",
    "isFixedAmount": true,
    "isActive": true,
    "lastGeneratedDate": "2025-07-15T10:00:00Z",
    "createdAt": "2025-01-02T10:00:00Z",
    "updatedAt": "2025-07-01T10:00:00Z",
    "expenseType": {
      "id": "uuid",
      "name": "Arriendo",
      "category": {
        "name": "Departamento"
      }
    }
  }
]
```

---

#### `GET /recurring-expenses/:id`
Obtiene un gasto recurrente por ID.

**Response 200:** (igual estructura)

---

#### `POST /recurring-expenses`
Crea configuración de gasto recurrente.

**Request Body (Fijo):**
```json
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

**Request Body (Variable):**
```json
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

**Validaciones:**
- `expenseTypeId`: Required, UUID válido, único
- `amount`: Required, >= 0 (puede ser 0 si isFixedAmount = false)
- `dayOfMonth`: Required, 1-31
- `paymentType`: Required, enum válido
- `frequency`: Required, monthly o annual
- `isFixedAmount`: Required, boolean

**Response 201:**
```json
{
  "id": "uuid",
  "expenseTypeId": "uuid-arriendo",
  "amount": 400000,
  ...
}
```

**Response 422:**
```json
{
  "statusCode": 422,
  "message": "Expense type already has a recurring configuration",
  "error": "Unprocessable Entity"
}
```

---

#### `PUT /recurring-expenses/:id`
Actualiza configuración recurrente.

**Request Body:**
```json
{
  "amount": 420000,
  "isActive": true
}
```

**Nota:** Cambio de `amount` afecta futuros gastos generados, no históricos.

**Response 200:** (igual estructura)

---

#### `DELETE /recurring-expenses/:id`
Elimina configuración recurrente.

**Response 204:** No Content

---

#### `PATCH /recurring-expenses/:id/pause`
Pausa un gasto recurrente.

**Response 200:**
```json
{
  "id": "uuid",
  "isActive": false,
  ...
}
```

---

#### `PATCH /recurring-expenses/:id/reactivate`
Reactiva un gasto recurrente (con posibilidad de actualizar monto).

**Request Body (Opcional):**
```json
{
  "amount": 4500
}
```

**Response 200:**
```json
{
  "id": "uuid",
  "amount": 4500,
  "isActive": true,
  ...
}
```

---

#### `POST /recurring-expenses/generate`
Genera gastos recurrentes para un período.

**Request Body:**
```json
{
  "year": 2025,
  "month": 7
}
```

**Validaciones:**
- `year`: Required, número
- `month`: Optional, 1-12 (si no se proporciona, genera todo el año)

**Response 200:**
```json
{
  "generated": 12,
  "skipped": 2,
  "errors": 0,
  "details": [
    {
      "expenseTypeName": "Arriendo",
      "status": "generated",
      "expenseId": "uuid",
      "amount": 400000,
      "date": "2025-07-01"
    },
    {
      "expenseTypeName": "Uber One",
      "status": "skipped",
      "reason": "Already generated for this period"
    }
  ]
}
```

---

#### `POST /recurring-expenses/generate-year`
Genera todos los recurrentes de un año completo.

**Request Body:**
```json
{
  "year": 2025
}
```

**Response 200:**
```json
{
  "year": 2025,
  "totalGenerated": 144,
  "byMonth": {
    "01": 12,
    "02": 12,
    "03": 12,
    ...
  }
}
```

---

## Reports Domain (Futuro)

### Reportes

#### `GET /reports/monthly`
Reporte mensual consolidado.

**Query Parameters:**
- `month` (YYYY-MM): Required

**Response 200:**
```json
{
  "month": "2025-07",
  "summary": {
    "totalSpent": 1009209,
    "totalBudget": 1200000,
    "budgetUsedPercentage": 84.1,
    "overBudgetCategories": 0
  },
  "byCategory": [...],
  "byPaymentMethod": {
    "credit": 856249,
    "debit": 150000,
    "transfer": 2960,
    "cash": 0
  },
  "topExpenses": [
    {
      "expenseTypeName": "Arriendo",
      "amount": 400000,
      "date": "2025-07-01"
    },
    ...
  ],
  "installments": {
    "paid": 168747,
    "remaining": 505992
  }
}
```

---

#### `GET /reports/compare`
Comparación entre dos períodos.

**Query Parameters:**
- `period1` (YYYY-MM): Required
- `period2` (YYYY-MM): Required

**Response 200:**
```json
{
  "period1": "2025-06",
  "period2": "2025-07",
  "comparison": {
    "totalChange": 50000,
    "percentageChange": 5.2,
    "categories": [
      {
        "categoryName": "Alimentación",
        "period1Amount": 150000,
        "period2Amount": 180000,
        "change": 30000,
        "percentageChange": 20
      }
    ]
  }
}
```

---

## Health & Utility Endpoints

#### `GET /health`
Health check de la API.

**Response 200:**
```json
{
  "status": "ok",
  "timestamp": "2025-07-15T10:00:00Z",
  "uptime": 86400,
  "database": "connected"
}
```

---

#### `GET /api-info`
Información de la API.

**Response 200:**
```json
{
  "name": "Bolsillo Flow API",
  "version": "1.0.0",
  "environment": "development",
  "documentation": "/api/docs"
}
```

---

## Convenciones de Paginación

Para endpoints que retornan listas grandes:

**Request:**
```
GET /expenses?page=2&limit=20
```

**Response:**
```json
{
  "data": [...],
  "meta": {
    "page": 2,
    "limit": 20,
    "total": 156,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true
  }
}
```

---

## Versionado (Futuro)

Cuando se implemente versionado:

```
/api/v1/categories
/api/v2/categories
```

Mantener v1 por período de depreciación antes de eliminar.

---

## Rate Limiting (Futuro)

Headers en respuesta:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1625097600
```

---

## CORS

En desarrollo, CORS permite todos los orígenes.

En producción, whitelist específica:
```typescript
cors({
  origin: [
    'https://bolsillo-flow.app',
    'https://app.bolsillo-flow.com'
  ]
})
```

---

## Autenticación (Futuro)

Headers requeridos:
```
Authorization: Bearer <jwt_token>
```

Respuesta sin auth:
```json
{
  "statusCode": 401,
  "message": "Unauthorized",
  "error": "Unauthorized"
}
```

---

## Webhooks (Futuro)

Para notificaciones de eventos:
- Gasto creado
- Presupuesto excedido
- Pago de tarjeta próximo

---

## Referencias

- [Requirements](../01-product/requirements.md)
- [Business Rules](../03-data-model/business-rules.md)
- [Swagger/OpenAPI](https://swagger.io/) (futuro)