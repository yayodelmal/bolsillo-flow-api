# Decisiones de Diseño (ADR)

## ADR-001: Arquitectura DDD-lite por dominios
**Fecha:** 2025-01-02
**Estado:** Aceptado

**Contexto:** 
Necesitamos organizar el código de manera escalable y mantenible.

**Decisión:**
Usar estructura DDD-lite agrupando por dominios de negocio:
- `financial/`: expenses, budgets, expense-types
- `payment-methods/`: credit-cards, debit-cards
- `automation/`: recurring-expenses
- `reports/`: analytics y reportes

**Consecuencias:**
+ Mejor separación de responsabilidades
+ Facilita scaling futuro
- Más complejo que estructura plana por entidad

---

## ADR-002: Presupuestos solo en categorías padre
**Fecha:** 2025-01-02
**Estado:** Aceptado

**Contexto:**
Categorías son jerárquicas (padre/hijos). ¿Dónde definir presupuestos?

**Decisión:**
Presupuestos solo a nivel de categoría padre.

**Razones:**
- Más simple de entender y gestionar
- Evita hilar muy fino
- Budget de "Transporte" cubre Uber + Copec automáticamente

**Consecuencias:**
+ Simplicidad
- Menos granularidad (si se necesita después, se puede agregar)