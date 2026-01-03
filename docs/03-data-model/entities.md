# Entidades del Sistema

## Category
Categorías jerárquicas para clasificar gastos.

**Campos:**
- `id`: UUID
- `name`: String - Nombre de la categoría
- `description`: String? - Descripción opcional
- `color`: String? - Color hex para UI (#FF5733)
- `parentId`: UUID? - Referencia a categoría padre (null = categoría padre)
- `isActive`: Boolean - Si está activa (default: true)
- `createdAt`: DateTime
- `updatedAt`: DateTime
- `deletedAt`: DateTime? - Soft delete

**Relaciones:**
- `parent`: Category (self-reference)
- `children`: Category[] (self-reference)
- `budgets`: Budget[] (futuro)
- `expenseTypes`: ExpenseType[] (futuro)

**Reglas de negocio:**
- Máximo 2 niveles de jerarquía (padre → hijo, no nieto)
- No se puede eliminar categoría padre si tiene hijos activos
- Al soft-delete, marca también hijos como deleted

---

## Budget
Presupuestos mensuales por categoría (solo padre).

[Completar cuando implementemos]

---

## CreditCard
Tarjetas de crédito con fechas de corte.

[Completar cuando implementemos]

...