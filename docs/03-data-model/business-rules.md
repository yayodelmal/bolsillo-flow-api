# Reglas de Negocio

## Overview
Este documento describe las reglas de negocio del sistema Bolsillo Flow API. Estas reglas definen **cómo debe comportarse el sistema** y son independientes de la implementación técnica.

---

## Categorías

### BR-CAT-001: Jerarquía Máxima
**Regla:**  
El sistema permite máximo **2 niveles de jerarquía** en categorías: padre → hijo. No se permiten nietos.

**Razón:**  
Simplificación. Más niveles complican la UX y reportes sin agregar valor significativo.

**Validación:**
- Al crear categoría con `parent_id`, verificar que el padre no tenga `parent_id`
- Error si se intenta crear tercer nivel

**Ejemplo:**
```
✅ Válido:
Departamento (padre)
  └─ Servicios Básicos (hijo)

❌ Inválido:
Departamento (padre)
  └─ Servicios Básicos (hijo)
      └─ Electricidad (nieto) ← NO PERMITIDO
```

---

### BR-CAT-002: Eliminación con Hijos
**Regla:**  
No se puede soft-delete una categoría padre si tiene hijos activos.

**Razón:**  
Evitar inconsistencias. Si eliminas padre pero hijos existen, rompe la estructura jerárquica.

**Opciones:**
1. Eliminar todos los hijos primero
2. El sistema elimina padre e hijos en cascada (automático)

**Decisión:** Opción 2 (cascada automática)

**Validación:**
- Al soft-delete categoría, verificar si tiene hijos
- Si tiene hijos, marcar `deleted_at` en padre e hijos
- Registrar en log cuáles se eliminaron

---

### BR-CAT-003: Restauración con Padre Eliminado
**Regla:**  
No se puede restaurar una categoría hija si su padre está soft-deleted.

**Razón:**  
Evitar huérfanos. Una categoría hija sin padre activo es inválida.

**Validación:**
- Al restaurar categoría hija, verificar que padre tenga `deleted_at = null`
- Error si padre está eliminado
- Mensaje: "Debe restaurar la categoría padre primero"

---

### BR-CAT-004: Categoría con Expense Types Activos
**Regla:**  
No se puede soft-delete una categoría si tiene expense types activos asociados.

**Razón:**  
Los expense types dependen de la categoría. Eliminar la categoría rompería la integridad.

**Validación:**
- Contar expense types donde `category_id = X` y `is_active = true`
- Si count > 0, rechazar eliminación
- Mensaje: "No puede eliminar categoría con tipos de gasto activos"

**Alternativa:**  
Eliminar en cascada expense types también (decisión pendiente).

---

### BR-CAT-005: Cambio de Padre
**Regla:**  
Al cambiar el `parent_id` de una categoría, validar que no se creen bucles ni exceda 2 niveles.

**Razón:**  
Evitar estructuras circulares (A padre de B, B padre de A) y mantener límite de niveles.

**Validaciones:**
1. Nuevo padre no puede ser descendiente de la categoría actual
2. Si categoría tiene hijos, no puede moverse bajo otra categoría (violaría 2 niveles)
3. Nuevo padre debe existir y estar activo

**Ejemplo:**
```
Estado inicial:
A (padre)
  └─ B (hijo)

✅ Válido: Mover B bajo C
C (padre)
  └─ B (hijo)

❌ Inválido: Mover A bajo B (bucle)
B (padre)
  └─ A (hijo) ← B era hijo de A

❌ Inválido: Si B tiene hijos, no puede moverse bajo A
A (padre)
  └─ B (hijo)
      └─ X (nieto) ← Violaría 2 niveles
```

---

## Presupuestos

### BR-BUD-001: Solo Categorías Padre
**Regla:**  
Los presupuestos solo se pueden definir para categorías **padre** (sin `parent_id`).

**Razón:**  
Simplificación. El presupuesto de "Transporte" cubre automáticamente Uber + Copec.

**Validación:**
- Al crear/actualizar presupuesto, verificar que categoría no tenga `parent_id`
- Error si es categoría hija
- Mensaje: "Los presupuestos solo se definen en categorías padre"

---

### BR-BUD-002: Monto Positivo
**Regla:**  
El monto del presupuesto debe ser **mayor a 0**.

**Razón:**  
Un presupuesto de 0 o negativo no tiene sentido financiero.

**Validación:**
- `amount > 0`
- Error si amount ≤ 0

---

### BR-BUD-003: Período Válido
**Regla:**  
El período debe ser formato **YYYY-MM** válido.

**Razón:**  
Consistencia en la base de datos y queries.

**Validación:**
- Regex: `^\d{4}-(0[1-9]|1[0-2])$`
- Mes entre 01 y 12
- Año razonable (ej: 2020-2099)

---

### BR-BUD-004: Único por Categoría-Período
**Regla:**  
Solo puede existir **un presupuesto** por combinación de `category_id` + `period`.

**Razón:**  
No tiene sentido dos presupuestos diferentes para la misma categoría en el mismo mes.

**Validación:**
- Constraint UNIQUE en BD: `[category_id, period]`
- Al crear, si ya existe, actualizar el existente
- Informar al usuario: "Ya existe presupuesto para esta categoría en este mes. Se actualizará."

---

### BR-BUD-005: Historial Inmutable
**Regla:**  
Los presupuestos históricos **no se eliminan**, solo se crean nuevos para períodos futuros.

**Razón:**  
Mantener trazabilidad de decisiones financieras pasadas.

**Validación:**
- No existe endpoint DELETE para presupuestos
- Solo CREATE y UPDATE
- UPDATE solo afecta el período específico, no históricos

---

## Tarjetas de Crédito

### BR-CC-001: Días de Corte y Vencimiento Válidos
**Regla:**  
Los días de corte y vencimiento deben estar entre **1 y 31**.

**Razón:**  
Límites físicos de días en un mes.

**Validación:**
- `cutoff_day >= 1 AND cutoff_day <= 31`
- `payment_due_day >= 1 AND payment_due_day <= 31`

**Caso especial:**
- Si corte es día 31 y mes tiene 30 días, usar día 30
- Si mes tiene 28/29 días (febrero), ajustar al último día

---

### BR-CC-002: Cupo Positivo
**Regla:**  
Si se define cupo, debe ser **mayor a 0**.

**Razón:**  
Un cupo de 0 o negativo no es válido.

**Validación:**
- Si `cupo` se proporciona: `cupo > 0`
- `cupo` es opcional (nullable)

---

### BR-CC-003: Cambio de Fecha de Corte No Retroactivo
**Regla:**  
Al cambiar la fecha de corte, solo afecta **gastos futuros** (fecha >= hoy), no recalcula billing_period de gastos históricos.

**Razón:**  
Los gastos pasados ya fueron facturados con la fecha de corte que existía en ese momento.

**Comportamiento:**
- Cambio de corte se registra con timestamp
- Gastos con `date < fecha_cambio`: mantienen billing_period original
- Gastos con `date >= fecha_cambio`: usan nuevo cutoff_day

**Ejemplo:**
```
Hoy: 20 de junio
Cambio corte de 15 a 10

Gasto del 12 de junio:
  → billing_period: "2025-06" (calculado con corte 15)
  → NO se recalcula

Gasto del 22 de junio (nuevo):
  → billing_period: "2025-07" (calculado con corte 10)
```

---

## Tarjetas de Débito

### BR-DC-001: Saldo Opcional
**Regla:**  
El saldo actual es **opcional** y puede ser null.

**Razón:**  
No todos los usuarios quieren trackear saldo de sus cuentas. El foco es gastos.

**Validación:**
- `current_balance` es nullable
- Si se proporciona, puede ser cualquier número (positivo, negativo, cero)

**Nota:** Saldo negativo representa sobregiro.

---

## Tipos de Gasto (Expense Types)

### BR-ET-001: Categoría Obligatoria y Existente
**Regla:**  
Todo tipo de gasto debe estar asociado a una **categoría existente y activa**.

**Razón:**  
Los tipos de gasto se agrupan por categoría para reportes.

**Validación:**
- `category_id` es obligatorio
- Categoría debe existir
- Categoría debe tener `is_active = true` y `deleted_at = null`

---

### BR-ET-002: Cuotas Válidas
**Regla:**  
Si `is_installment = true`, debe proporcionar configuración de cuotas válida.

**Razón:**  
Sin configuración de cuotas, no se puede generar los gastos mensuales.

**Validación de installment_config:**
- `total_installments > 0`
- `amount_per_installment > 0`
- `current_installment <= total_installments`
- `remaining_installments = total_installments - current_installment`
- `start_date` es fecha válida

**Ejemplo válido:**
```json
{
  "total_installments": 12,
  "amount_per_installment": 56249,
  "current_installment": 3,
  "remaining_installments": 9,
  "start_date": "2025-02-25"
}
```

---

### BR-ET-003: Auto-Completado de Cuotas
**Regla:**  
Cuando `current_installment = total_installments`, automáticamente:
- `is_completed = true`
- `is_active = false`

**Razón:**  
Un producto totalmente pagado no debe aparecer en listados activos.

**Trigger:**
- Al crear/actualizar Expense con `installment_number = "N/N"`
- Sistema actualiza ExpenseType automáticamente

**Reversión:**
- No hay reversión automática
- Si se elimina último pago, debe reactivarse manualmente

---

### BR-ET-004: No Eliminar con Gastos Asociados
**Regla:**  
No se puede soft-delete un ExpenseType si tiene gastos (Expenses) asociados.

**Razón:**  
Los gastos dependen del tipo. Eliminar el tipo rompería referencias.

**Validación:**
- Contar expenses donde `expense_type_id = X`
- Si count > 0, rechazar eliminación
- Mensaje: "No puede eliminar tipo de gasto con transacciones asociadas"

**Alternativa:**
- Solo marcar `is_active = false` (desactivar)
- Mantiene integridad pero oculta de listados activos

---

## Gastos (Expenses)

### BR-EXP-001: Tipo de Gasto Activo
**Regla:**  
Solo se pueden crear gastos de tipos (ExpenseType) que estén **activos** (`is_active = true`, `deleted_at = null`).

**Razón:**  
No tiene sentido crear gastos de tipos descontinuados.

**Validación:**
- Al crear expense, verificar que expense_type esté activo
- Error si está inactivo o eliminado
- Mensaje: "No puede crear gastos de un tipo inactivo"

**Excepción:**
- Editar gastos históricos de tipos ahora inactivos: permitido

---

### BR-EXP-002: Monto Positivo
**Regla:**  
El monto debe ser **mayor a 0**.

**Razón:**  
Gastos negativos no tienen sentido en este sistema (no maneja ingresos/reembolsos en MVP).

**Validación:**
- `amount > 0`
- Error si amount ≤ 0

**Decisión pendiente:**  
¿Permitir montos negativos para reembolsos? (Backlog)

---

### BR-EXP-003: Consistencia de Método de Pago
**Regla:**  
Si `payment_type = 'credit'` → `credit_card_id` es **obligatorio**  
Si `payment_type = 'debit'` → `debit_card_id` es **obligatorio**  
Si `payment_type = 'transfer'` o `'cash'` → ningún card_id requerido

**Razón:**  
Integridad referencial y para cálculos de billing period.

**Validación:**
- Validar a nivel de DTO
- Error si no coincide

---

### BR-EXP-004: Cálculo Automático de Billing Period
**Regla:**  
Si `payment_type = 'credit'`, el sistema **calcula automáticamente** el `billing_period` basado en:
- Fecha del gasto (`date`)
- Día de corte de la tarjeta (`cutoff_day`)

**Algoritmo:**
```
if (date.day <= cutoff_day) {
  billing_period = date.format('YYYY-MM')
} else {
  billing_period = nextMonth(date).format('YYYY-MM')
}
```

**Razón:**  
Automatización. El usuario no debe calcular manualmente en qué período se facturará.

**Validación:**
- Sistema sobrescribe cualquier `billing_period` proporcionado
- Usuario es informado: "Este gasto será facturado en [período]"

**Ejemplo:**
```
Tarjeta: corte día 15
Gasto: 10 de junio → billing_period = "2025-06"
Gasto: 20 de junio → billing_period = "2025-07"
```

---

### BR-EXP-005: Billing Period Solo para Crédito
**Regla:**  
Si `payment_type != 'credit'`, entonces `billing_period = null`.

**Razón:**  
El concepto de billing period solo aplica a tarjetas de crédito.

**Validación:**
- Si payment_type es debit/transfer/cash, forzar billing_period a null

---

### BR-EXP-006: Tarjeta Activa
**Regla:**  
Solo se pueden crear gastos con tarjetas **activas** (`is_active = true`, `deleted_at = null`).

**Razón:**  
No tiene sentido usar tarjetas canceladas/eliminadas.

**Validación:**
- Verificar que credit_card o debit_card esté activa
- Error si está inactiva o eliminada

**Excepción:**
- Editar gastos históricos de tarjetas ahora inactivas: permitido

---

### BR-EXP-007: Formato de Número de Cuota
**Regla:**  
Si el gasto es parte de cuotas, `installment_number` debe tener formato **"N/M"** donde:
- N = número de cuota actual (1 a M)
- M = total de cuotas
- N ≤ M

**Razón:**  
Consistencia y posibilidad de parsear automáticamente.

**Validación:**
- Regex: `^(\d+)/(\d+)$`
- Parsear N y M
- Validar N ≤ M
- Error si formato incorrecto

**Ejemplo válido:** "3/12", "1/6", "12/12"  
**Ejemplo inválido:** "13/12", "0/6", "3-12"

---

### BR-EXP-008: Cuota Corresponde a ExpenseType
**Regla:**  
Si `installment_number` se proporciona, el `expense_type` debe tener `is_installment = true`.

**Razón:**  
Solo tipos de gasto configurados como cuotas pueden tener gastos con número de cuota.

**Validación:**
- Verificar expense_type.is_installment = true
- Error si es false
- Mensaje: "Este tipo de gasto no está configurado para cuotas"

---

### BR-EXP-009: No Duplicar Cuota del Mismo Mes
**Regla:**  
No se pueden crear dos gastos del mismo `expense_type` con el mismo `installment_number` en el mismo mes.

**Razón:**  
Una cuota 3/12 no se paga dos veces en el mismo mes.

**Validación:**
- Buscar expenses con:
  - mismo expense_type_id
  - mismo installment_number
  - fecha en el mismo mes (YYYY-MM)
- Si existe, rechazar

**Nota:** Esto previene duplicados accidentales.

---

### BR-EXP-010: Validación de Fecha Futura
**Regla (Decisión pendiente):**  
¿Se permiten gastos con fecha futura?

**Opciones:**
- **Opción A:** No permitir (solo gastos ya realizados)
- **Opción B:** Permitir (para planificación)

**Implicación:**
- Si se permite, afecta cálculos de reportes "al día de hoy"
- Necesitaría filtro: "gastos hasta hoy" vs "gastos planificados"

**Decisión actual:** Pendiente, implementar cuando se necesite.

---

### BR-EXP-011: Soft Delete de Gastos
**Regla (Decisión pendiente):**  
¿Los gastos se pueden eliminar (soft delete) o solo editar?

**Consideraciones:**
- **Pro eliminar:** Corrección de errores graves
- **Contra eliminar:** Pérdida de trazabilidad

**Decisión tentativa:**  
- Permitir soft delete solo si el gasto tiene < 7 días
- Gastos antiguos solo se pueden editar con notas de corrección

---

## Gastos Recurrentes

### BR-REC-001: Tipo de Gasto Único
**Regla:**  
Un `expense_type` solo puede tener **una configuración recurrente**.

**Razón:**  
No tiene sentido que "Arriendo" se genere dos veces al mes con configuraciones diferentes.

**Validación:**
- Constraint UNIQUE en `expense_type_id`
- Error si ya existe RecurringExpense para ese tipo
- Mensaje: "Este tipo de gasto ya tiene configuración recurrente"

---

### BR-REC-002: Día del Mes Válido
**Regla:**  
`day_of_month` debe estar entre **1 y 31**.

**Razón:**  
Límites de días en un mes.

**Validación:**
- `day_of_month >= 1 AND day_of_month <= 31`

**Caso especial:**
- Si es día 31 y el mes tiene 30 días, generar el día 30
- Si es febrero y día > 28/29, generar el último día del mes

---

### BR-REC-003: Monto Puede ser Cero si es Variable
**Regla:**  
Si `is_fixed_amount = false`, el `amount` puede ser **0** (se ajusta después).

**Razón:**  
Gastos como "Gastos Comunes" varían mes a mes. Se generan con 0 y usuario edita.

**Validación:**
- Si is_fixed_amount = true → amount > 0
- Si is_fixed_amount = false → amount >= 0 (puede ser 0)

---

### BR-REC-004: No Duplicar Generación
**Regla:**  
Al generar gastos recurrentes, no duplicar si ya existe para ese período.

**Razón:**  
Evitar gastos duplicados si se llama endpoint dos veces.

**Validación:**
- Verificar `last_generated_date`
- Si last_generated_date >= inicio del período actual, skip
- Actualizar last_generated_date después de generar

**Ejemplo:**
```
Hoy: 15 de julio
Recurrente: Arriendo, día 1

Llamada 1 a /generate-month?month=2025-07:
  → Genera gasto del 1 de julio
  → last_generated_date = 2025-07-15

Llamada 2 a /generate-month?month=2025-07:
  → last_generated_date ya es julio
  → Skip, no duplicar
```

---

### BR-REC-005: Solo Activos se Generan
**Regla:**  
Solo se generan gastos de RecurringExpenses con `is_active = true`.

**Razón:**  
Gastos pausados no deben generarse.

**Validación:**
- Filtrar por is_active = true al generar
- Recurrentes pausados se ignoran silenciosamente

---

### BR-REC-006: Usar Monto del Mes Anterior (CAE)
**Regla:**  
Si `is_fixed_amount = false` y `use_previous_month_amount = true`, usar el monto del último gasto de ese tipo.

**Razón:**  
Gastos como CAE varían poco mes a mes. Tomar el anterior como estimado inicial.

**Algoritmo:**
```
1. Buscar último Expense de ese expense_type
2. Si existe, usar su amount
3. Si no existe, usar amount configurado (puede ser 0)
4. Usuario puede editar después
```

**Ejemplo:**
```
CAE junio: $53,500
Generar julio:
  → use_previous_month_amount = true
  → amount inicial = $53,500
  → Usuario edita a $54,000 cuando llega la cuenta real
```

---

### BR-REC-007: Método de Pago Consistente
**Regla:**  
El método de pago definido en RecurringExpense debe ser válido (mismas reglas que Expense).

**Validación:**
- Si payment_type = credit → credit_card_id obligatorio
- Si payment_type = debit → debit_card_id obligatorio
- Tarjeta debe estar activa

---

### BR-REC-008: Reactivación con Nuevo Monto
**Regla:**  
Al reactivar (`is_active = false` → `true`), se puede actualizar el `amount`.

**Razón:**  
Suscripciones pueden haber cambiado de precio mientras estaban pausadas.

**Comportamiento:**
- Endpoint: `PATCH /recurring-expenses/:id/reactivate`
- Body opcional: `{ amount: 4500 }`
- Actualiza amount y marca is_active = true
- Futuros gastos se generan con nuevo monto

---

## Reportes

### BR-REP-001: Gastos Agrupados por Categoría Padre
**Regla:**  
Los reportes mensuales agrupan gastos por **categoría padre**, no por hijos.

**Razón:**  
El presupuesto está en el padre. Agrupar por hijos fragmentaría el análisis.

**Ejemplo:**
```
Reporte de julio:
Transporte (padre): $150,000
  ├─ Uber: $80,000
  └─ Copec: $70,000

Total mostrado: $150,000 en "Transporte"
No: $80,000 en "Uber" + $70,000 en "Copec" separados
```

---

### BR-REP-002: Cálculo de Porcentaje de Presupuesto
**Regla:**  
`percentage_used = (total_gastado / budget) * 100`

**Casos especiales:**
- Si no hay presupuesto definido: `percentage_used = null`
- Si presupuesto = 0: `percentage_used = null` (no dividir por 0)

---

### BR-REP-003: Indicador Over Budget
**Regla:**  
`over_budget = (total_gastado > budget)`

**Casos especiales:**
- Si no hay presupuesto: `over_budget = false` (no hay límite que superar)

---

### BR-REP-004: Próximo Pago de Tarjeta
**Regla:**  
El total a pagar es la suma de todos los gastos del **billing_period actual**.

**Cálculo de período actual:**
```
Hoy: 20 de junio
Corte: día 15

Si hoy <= 15:
  → Período actual = junio (2025-06)
  → Próximo pago = 5 de julio

Si hoy > 15:
  → Período actual = julio (2025-07)
  → Próximo pago = 5 de agosto
```

**Incluye:**
- Gastos normales del período
- Cuotas del período
- Todo tipo de gasto con esa tarjeta

---

## Validaciones Generales

### BR-GEN-001: UUIDs Válidos
**Regla:**  
Todos los IDs deben ser **UUIDs v4** válidos.

**Razón:**  
Consistencia, seguridad (no predecibles), compatibilidad con Prisma.

**Validación:**
- Regex UUID v4
- Generación automática por BD (default uuid())

---

### BR-GEN-002: Fechas en ISO Format
**Regla:**  
Todas las fechas se manejan en formato **ISO 8601** (YYYY-MM-DD o YYYY-MM-DDTHH:mm:ss).

**Razón:**  
Estándar internacional, compatible con Prisma, fácil de parsear.

**Validación:**
- DTOs usan `@IsDate()` + `@Type(() => Date)`
- Base de datos: tipo DateTime

---

### BR-GEN-003: Montos en Decimal
**Regla:**  
Todos los montos monetarios usan tipo **Decimal** (no Float).

**Razón:**  
Precisión. Float tiene errores de redondeo inaceptables para finanzas.

**Configuración Prisma:**
```prisma
amount Decimal @db.Decimal(12, 2)
// Hasta 999,999,999.99
```

---

### BR-GEN-004: Soft Delete Consistente
**Regla:**  
Las entidades con soft delete:
- Tienen campo `deleted_at` (nullable DateTime)
- `deleted_at = null` → activo
- `deleted_at != null` → eliminado

**Queries por defecto:**
- Filtrar `WHERE deleted_at IS NULL`
- Endpoint especial para incluir deleted: `?includeDeleted=true`

---

### BR-GEN-005: Auditoría Obligatoria
**Regla:**  
Todas las entidades tienen:
- `created_at`: Timestamp de creación (auto)
- `updated_at`: Timestamp de última modificación (auto)

**Razón:**  
Trazabilidad, debugging, análisis de uso.

---

## Excepciones y Casos Especiales

### EX-001: Meses con Menos de 31 Días
**Problema:**  
Día de corte = 31, pero mes tiene 30 días (o 28/29 en febrero).

**Solución:**  
Usar el **último día del mes**.

**Ejemplo:**
```
Corte configurado: día 31
Mes: Abril (30 días)
Corte efectivo: día 30

Gasto del 29 de abril → billing_period = "2025-04"
Gasto del 30 de abril → billing_period = "2025-04" (último día)
Gasto del 1 de mayo → billing_period = "2025-05"
```

---

### EX-002: Cambio de Zona Horaria
**Problema:**  
Usuario en Chile (UTC-3/-4 según DST), servidor en UTC.

**Solución:**  
- Todas las fechas se almacenan en **UTC**
- Conversión a timezone local en frontend
- API siempre trabaja en UTC

---

### EX-003: Gastos del Día de Corte
**Problema:**  
Gasto exactamente el día de corte, ¿en qué período cae?

**Decisión:**  
Cae en el **período actual** (antes del corte).

**Ejemplo:**
```
Corte: día 15
Gasto: 15 de junio a las 23:59

billing_period = "2025-06" (no 2025-07)
```

**Razón:**  
El corte es "al final del día 15", no "al inicio".

---

## Priorización de Reglas

### Críticas (deben implementarse en MVP)
- Todas las reglas de Categories (jerarquía, eliminación)
- Todas las reglas de Budgets (solo padre, monto positivo)
- Billing period automático (BR-EXP-004)
- Consistencia de método de pago (BR-EXP-003)
- Auto-completado de cuotas (BR-ET-003)
- No duplicar generación recurrentes (BR-REC-004)

### Importantes (implementar post-MVP)
- Validación de fecha futura (BR-EXP-010)
- Usar monto anterior en recurrentes (BR-REC-006)
- No duplicar cuotas del mismo mes (BR-EXP-009)

### Nice to have (backlog)
- Soft delete de gastos con restricciones de tiempo (BR-EXP-011)
- Validaciones avanzadas de duplicados

---

## Matriz de Validación

| Regla | Validación | Cuándo | Dónde |
|-------|-----------|--------|-------|
| BR-CAT-001 | Max 2 niveles | Al crear/actualizar categoría | CategoriesService |
| BR-CAT-002 | Cascada en eliminación | Al soft-delete padre | CategoriesService |
| BR-BUD-001 | Solo padre | Al crear presupuesto | BudgetsService |
| BR-EXP-004 | Billing period | Al crear gasto con crédito | ExpensesService |
| BR-REC-004 | No duplicar | Al generar recurrentes | RecurringExpensesService |

---

## Changelog de Reglas

### 2025-01-02 (Inicial)
- Todas las reglas definidas en versión inicial
- Decisiones pendientes marcadas
- Priorización establecida

### (Futuras actualizaciones)
- Documentar cambios en reglas
- Razón del cambio
- Impacto en sistema existente

---

## Referencias

- [Requirements](../01-product/requirements.md)
- [Entities](./entities.md)
- [Billing Cycle Calculation](../05-workflows/billing-cycle-calculation.md)