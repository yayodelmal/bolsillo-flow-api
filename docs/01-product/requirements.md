# Requisitos Funcionales

## Overview
Este documento detalla los requisitos funcionales de Bolsillo Flow API, organizados por dominio y priorizados para el MVP.

---

## 1. Gestión de Categorías

### RF-001: Crear Categoría
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** crear categorías para clasificar mis gastos  
**Para** organizar y agrupar mis transacciones

**Criterios de aceptación:**
- Puedo crear categorías padre (sin parent_id)
- Puedo crear categorías hijas (con parent_id)
- Máximo 2 niveles de jerarquía (padre → hijo, no nieto)
- Nombre es obligatorio (3-50 caracteres)
- Descripción y color son opcionales
- Color debe ser formato hex (#RRGGBB)
- Nueva categoría se crea como activa por defecto

**Validaciones:**
- Nombre no puede estar vacío
- Parent_id debe existir si se proporciona
- Parent_id no puede ser de una categoría hija (evitar 3er nivel)

---

### RF-002: Listar Categorías
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** ver todas mis categorías  
**Para** conocer mi estructura de clasificación

**Criterios de aceptación:**
- Puedo listar todas las categorías activas
- Opcionalmente incluir categorías inactivas (query param)
- Opcionalmente filtrar solo categorías padre (query param)
- La respuesta incluye relaciones padre/hijos
- Categorías soft-deleted no aparecen por defecto

**Query params opcionales:**
- `includeInactive=true`: Mostrar también inactivas
- `parentOnly=true`: Solo categorías padre
- `includeDeleted=true`: Incluir soft-deleted

---

### RF-003: Obtener Categoría por ID
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** ver el detalle de una categoría  
**Para** revisar su información y subcategorías

**Criterios de aceptación:**
- Retorna toda la información de la categoría
- Incluye categoría padre si existe
- Incluye lista de categorías hijas si existen
- Error 404 si la categoría no existe o está soft-deleted
- Opcionalmente incluir categorías deleted (query param)

---

### RF-004: Obtener Hijos de Categoría
**Prioridad:** Media (MVP)  
**Como** usuario  
**Quiero** ver las subcategorías de una categoría padre  
**Para** entender su estructura jerárquica

**Criterios de aceptación:**
- Retorna array de categorías hijas
- Solo categorías activas por defecto
- Array vacío si no tiene hijos
- Error 404 si la categoría padre no existe

---

### RF-005: Actualizar Categoría
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** modificar una categoría  
**Para** corregir información o reorganizar la estructura

**Criterios de aceptación:**
- Puedo actualizar nombre, descripción, color
- Puedo cambiar el parent_id (mover categoría)
- Puedo activar/desactivar categoría
- No puedo crear bucles (A padre de B, B padre de A)
- No puedo crear más de 2 niveles de jerarquía

**Validaciones:**
- Mismo formato que crear categoría
- Verificar que no se creen bucles en jerarquía
- Si tiene hijos, no puede convertirse en hija de otra

---

### RF-006: Eliminar Categoría (Soft Delete)
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** eliminar una categoría  
**Para** mantener limpia mi estructura

**Criterios de aceptación:**
- Soft delete: marca deleted_at, no borra físicamente
- No se puede eliminar categoría con hijos activos
- No se puede eliminar categoría con gastos asociados (futuro)
- Al eliminar padre, se marcan deleted sus hijos también
- Categoría eliminada no aparece en listados normales

**Validaciones:**
- Error si tiene hijos activos
- Error si tiene expense_types activos (cuando se implemente)

---

### RF-007: Restaurar Categoría
**Prioridad:** Media (MVP)  
**Como** usuario  
**Quiero** restaurar una categoría eliminada  
**Para** recuperarla si la borré por error

**Criterios de aceptación:**
- Marca deleted_at = null
- Si es categoría hija, su padre debe estar activo
- Restaurar padre no restaura automáticamente hijos
- Categoría restaurada vuelve a aparecer en listados

---

## 2. Gestión de Presupuestos

### RF-008: Definir Presupuesto Mensual
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** definir un presupuesto para una categoría  
**Para** controlar cuánto planeo gastar en ese rubro

**Criterios de aceptación:**
- Solo puedo definir presupuesto en categorías padre
- Presupuesto es por mes específico (YYYY-MM)
- Monto debe ser positivo
- Un presupuesto por categoría por mes (unique constraint)
- Si actualizo presupuesto del mismo mes/categoría, se actualiza el monto

**Validaciones:**
- Category_id debe existir y ser categoría padre
- Period debe ser formato YYYY-MM válido
- Amount debe ser > 0

---

### RF-009: Consultar Presupuesto
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** ver el presupuesto de una categoría en un mes  
**Para** saber cuánto tengo asignado

**Criterios de aceptación:**
- Retorna presupuesto de categoría + mes específico
- Si no existe presupuesto, retorna null o mensaje claro
- No retorna error, solo indica que no está definido

---

### RF-010: Historial de Presupuestos
**Prioridad:** Media (MVP)  
**Como** usuario  
**Quiero** ver cómo ha cambiado mi presupuesto en el tiempo  
**Para** analizar mis decisiones financieras

**Criterios de aceptación:**
- Retorna todos los presupuestos de una categoría
- Ordenados por período (más reciente primero)
- Incluye mes y monto de cada presupuesto
- Puedo filtrar por rango de fechas (opcional)

---

## 3. Gestión de Tarjetas

### RF-011: Registrar Tarjeta de Crédito
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** registrar mi tarjeta de crédito  
**Para** calcular automáticamente períodos de facturación

**Criterios de aceptación:**
- Nombre/alias es obligatorio
- Banco es obligatorio
- Día de corte es obligatorio (1-31)
- Día de vencimiento es obligatorio (1-31)
- Últimos 4 dígitos y cupo son opcionales
- Tarjeta se crea como activa por defecto

**Validaciones:**
- Días de corte/vencimiento entre 1 y 31
- Si se proporciona, cupo debe ser > 0

---

### RF-012: Registrar Tarjeta de Débito
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** registrar mi tarjeta de débito  
**Para** asociarla a gastos directos

**Criterios de aceptación:**
- Nombre/alias es obligatorio
- Banco es obligatorio
- Saldo actual es opcional
- Tarjeta se crea como activa por defecto

---

### RF-013: Actualizar Fecha de Corte
**Prioridad:** Media (MVP)  
**Como** usuario  
**Quiero** cambiar la fecha de corte de mi tarjeta  
**Para** reflejar cambios del banco

**Criterios de aceptación:**
- Solo afecta gastos futuros (desde fecha de cambio)
- No recalcula billing_period de gastos históricos
- Gastos con fecha >= hoy usan nuevo corte

**Nota:** Este cambio no es retroactivo para evitar inconsistencias en gastos ya facturados.

---

### RF-014: Calcular Próximo Pago de Tarjeta
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** saber cuánto debo pagar en el próximo vencimiento  
**Para** preparar el dinero

**Criterios de aceptación:**
- Retorna total a pagar del período de facturación actual
- Muestra fecha de corte y fecha de vencimiento
- Lista todos los gastos del período (breakdown)
- Indica cuántos días faltan para el pago
- Diferencia entre gastos normales y cuotas

**Cálculo:**
```
Período actual = basado en fecha de hoy y día de corte
Total = SUM(expenses WHERE billing_period = período_actual)
```

---

## 4. Gestión de Tipos de Gasto

### RF-015: Crear Tipo de Gasto
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** definir tipos de gasto específicos  
**Para** clasificar mis transacciones

**Criterios de aceptación:**
- Nombre es obligatorio
- Categoría es obligatoria (debe existir)
- Puedo marcar si es recurrente
- Puedo marcar si es cuota/installment
- Si es cuota, debo proporcionar configuración de cuotas

**Configuración de cuotas (JSON):**
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

### RF-016: Marcar Cuota como Completada
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** que el sistema detecte cuando pagué todas las cuotas  
**Para** saber qué productos terminé de pagar

**Criterios de aceptación:**
- Automático al crear/actualizar último gasto de cuota
- Si installment_number = "12/12", marca is_completed = true
- Si is_completed = true, entonces is_active = false
- Puedo consultar tipos de gasto completados en historial
- No aparece en listados activos

**Trigger:**
```
Al crear Expense con installment_number = "N/N":
  → ExpenseType.is_completed = true
  → ExpenseType.is_active = false
```

---

## 5. Gestión de Gastos

### RF-017: Registrar Gasto Individual
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** registrar un gasto puntual  
**Para** llevar control de mis transacciones

**Criterios de aceptación:**
- Tipo de gasto es obligatorio (expense_type_id)
- Monto es obligatorio (> 0)
- Fecha es obligatoria
- Método de pago es obligatorio (credit/debit/transfer/cash)
- Si método es crédito, credit_card_id es obligatorio
- Si método es débito, debit_card_id es obligatorio
- Notas es opcional

**Cálculo automático (si es crédito):**
- Billing_period calculado según fecha y día de corte
- Sistema informa: "Este gasto será facturado en [período] y debes pagarlo el [fecha]"

---

### RF-018: Registrar Cuota
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** registrar el pago de una cuota  
**Para** llevar control de mis compras en cuotas

**Criterios de aceptación:**
- Gasto vinculado a ExpenseType con is_installment = true
- Installment_number es obligatorio (formato "3/12")
- Monto corresponde a amount_per_installment del tipo
- Si es última cuota (N/N), ejecuta RF-016

**Formato installment_number:**
- Debe ser "N/M" donde N ≤ M
- N = número de cuota actual
- M = total de cuotas

---

### RF-019: Listar Gastos por Mes
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** ver todos mis gastos de un mes  
**Para** revisar en qué gasté

**Criterios de aceptación:**
- Filtro por mes (YYYY-MM) obligatorio
- Opcionalmente filtrar por categoría
- Opcionalmente filtrar por método de pago
- Opcionalmente filtrar por rango de montos
- Incluye información de expense_type y categoría
- Ordenado por fecha descendente por defecto

**Query params:**
- `month`: YYYY-MM (obligatorio)
- `category_id`: UUID (opcional)
- `payment_type`: credit|debit|transfer|cash (opcional)
- `min_amount`: number (opcional)
- `max_amount`: number (opcional)

---

### RF-020: Resumen Mensual por Categoría
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** ver cuánto gasté por categoría en un mes  
**Para** entender mi distribución de gastos

**Criterios de aceptación:**
- Agrupa gastos por categoría padre
- Muestra total gastado vs presupuesto (si existe)
- Calcula porcentaje usado del presupuesto
- Indica si me pasé del presupuesto
- Incluye conteo de transacciones por categoría

**Respuesta esperada:**
```json
{
  "month": "2025-06",
  "total_spent": 1009209,
  "categories": [
    {
      "category_name": "Departamento",
      "total": 598205,
      "budget": 600000,
      "percentage_used": 99.7,
      "expenses_count": 5,
      "over_budget": false
    },
    ...
  ]
}
```

---

### RF-021: Vista Anual de Cuotas
**Prioridad:** Media (MVP)  
**Como** usuario  
**Quiero** ver todas mis cuotas del año  
**Para** planificar pagos futuros

**Criterios de aceptación:**
- Lista todos los gastos marcados como cuota
- Filtro por año
- Muestra cuotas pagadas y pendientes
- Agrupa por producto (ExpenseType)
- Indica progreso (3/12 cuotas pagadas)

---

## 6. Gastos Recurrentes

### RF-022: Configurar Gasto Recurrente
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** definir gastos que se repiten todos los meses  
**Para** no tener que registrarlos manualmente

**Criterios de aceptación:**
- Tipo de gasto es obligatorio
- Monto es obligatorio (puede ser 0 si es variable)
- Día del mes es obligatorio (1-31)
- Método de pago es obligatorio
- Frecuencia: mensual o anual
- Indico si el monto es fijo o variable
- Se crea como activo por defecto

**Ejemplos:**
- Arriendo: $400,000, día 1, fijo, mensual
- Gastos comunes: $0, día 1, variable, mensual (ajusto después)

---

### RF-023: Generar Gastos Recurrentes
**Prioridad:** Alta (MVP)  
**Como** usuario  
**Quiero** generar todos los gastos recurrentes de un período  
**Para** tenerlos listos sin crear uno por uno

**Criterios de aceptación:**
- Puedo generar por mes específico o año completo
- Solo genera recurrentes activos (is_active = true)
- No duplica: verifica last_generated_date
- Si monto es variable, puede usar monto del mes anterior (CAE)
- Si is_fixed_amount = true, usa el monto configurado
- Si is_fixed_amount = false y use_previous_month = true, busca último gasto de ese tipo

**Endpoint propuesto:**
```
POST /recurring-expenses/generate
{
  "year": 2025,
  "month": 7  // opcional, si no se envía genera todo el año
}
```

---

### RF-024: Pausar Gasto Recurrente
**Prioridad:** Media (MVP)  
**Como** usuario  
**Quiero** pausar un gasto recurrente  
**Para** dejarlo de generar temporalmente sin eliminarlo

**Criterios de aceptación:**
- Marca is_active = false
- Deja de generarse en futuros llamados a generar
- No afecta gastos ya generados
- Puedo reactivarlo después

---

### RF-025: Reactivar Gasto Recurrente
**Prioridad:** Media (MVP)  
**Como** usuario  
**Quiero** reactivar un gasto pausado  
**Para** que vuelva a generarse, posiblemente con nuevo precio

**Criterios de aceptación:**
- Marca is_active = true
- Puedo actualizar el monto al reactivar (caso: suscripción subió de precio)
- Futuros gastos se generan con el nuevo monto
- No afecta gastos históricos

**Endpoint propuesto:**
```
PATCH /recurring-expenses/:id/reactivate
{
  "amount": 4500  // nuevo monto opcional
}
```

---

## 7. Reportes

### RF-026: Reporte Mensual Consolidado
**Prioridad:** Media (MVP)  
**Como** usuario  
**Quiero** ver un resumen completo de un mes  
**Para** entender mi situación financiera

**Criterios de aceptación:**
- Total gastado en el mes
- Desglose por categoría padre
- Comparación con presupuestos
- Métodos de pago más usados
- Top 5 gastos más grandes
- Cuotas pagadas ese mes

---

### RF-027: Comparación Mes a Mes
**Prioridad:** Baja (Backlog)  
**Como** usuario  
**Quiero** comparar gastos entre dos meses  
**Para** identificar cambios en mis hábitos

---

### RF-028: Tendencias por Categoría
**Prioridad:** Baja (Backlog)  
**Como** usuario  
**Quiero** ver cómo ha evolucionado mi gasto en una categoría  
**Para** detectar patrones

---

## 8. Validaciones Generales

### RF-029: Validación de Fechas
**Decisión pendiente:**  
- ¿Se permiten gastos con fecha futura?
- ¿Hasta qué tan atrás se puede registrar un gasto?

### RF-030: Montos Negativos
**Decisión pendiente:**  
- ¿Se permiten montos negativos para reembolsos?
- ¿O es solo una app de gastos (positivos únicamente)?

### RF-031: Duplicados
**Decisión pendiente:**  
- ¿Validar gastos duplicados? (mismo monto, día, tipo)
- ¿O permitirlos porque pueden ser legítimos?

---

## Priorización para MVP

### ✅ Prioridad Alta (Implementar en Fase 1)
- RF-001 a RF-007: Categorías completo
- RF-008 a RF-010: Presupuestos básico
- RF-011 a RF-014: Tarjetas
- RF-015 a RF-016: Tipos de gasto
- RF-017 a RF-020: Gastos core
- RF-022 a RF-025: Recurrentes

### 🔶 Prioridad Media (Fase 2)
- RF-021: Vista anual cuotas
- RF-026: Reporte consolidado

### 🔵 Prioridad Baja (Backlog)
- RF-027 a RF-028: Analytics avanzados
- RF-029 a RF-031: Validaciones a definir iterando

---

## Notas de Implementación

### Soft Delete Universal
Todas las entidades principales soportan soft delete:
- Categories
- ExpenseTypes
- CreditCards
- DebitCards
- Expenses (opcional, generalmente no se eliminan)

### Auditoría
Todas las entidades tienen:
- `created_at`: Timestamp de creación
- `updated_at`: Timestamp de última modificación

### Consistencia de Datos
- Las relaciones foráneas deben validarse siempre
- No se permite eliminar entidades con dependencias activas
- Cambios en configuración no son retroactivos (salvo excepciones específicas)