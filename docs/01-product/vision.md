# Visión del Producto

## Problema
Gestionar finanzas personales en Excel es funcional pero limitado. Es difícil:
- Controlar gastos variables (supermercado, Uber, delivery)
- Calcular automáticamente períodos de facturación de tarjetas
- Hacer seguimiento de cuotas pendientes
- Generar reportes y comparaciones temporales

## Solución
Bolsillo Flow es una API REST para gestión integral de finanzas personales que:
- Permite granularidad en el registro de gastos
- Calcula automáticamente billing periods según fechas de corte
- Gestiona cuotas con auto-completado
- Genera gastos recurrentes
- Ofrece reportes mensuales/anuales

## Público Objetivo
Inicialmente: uso personal para gestión de finanzas en Chile (CLP).
Futuro: potencial para multi-usuario y gastos compartidos.

## Alcance MVP
- CRUD de categorías, gastos, tarjetas, presupuestos
- Cálculo automático de billing periods
- Gestión de cuotas e installments
- Gastos recurrentes (generación manual)
- Reportes básicos mensuales/anuales