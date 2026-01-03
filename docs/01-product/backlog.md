# Backlog de Features

## Overview
Este documento contiene todas las funcionalidades identificadas para futuras versiones de Bolsillo Flow API, organizadas por prioridad y complejidad.

---

## 🔴 Alta Prioridad (Post-MVP)

### BL-001: Alertas de Presupuesto
**Descripción:**  
Sistema de alertas cuando el gasto en una categoría alcanza ciertos umbrales del presupuesto.

**User Story:**  
Como usuario, quiero recibir alertas cuando esté cerca de mi límite de presupuesto, para controlar mejor mis gastos antes de pasarme.

**Funcionalidades:**
- Definir umbrales de alerta por categoría (ej: 75%, 90%, 100%)
- Notificaciones cuando se alcanza un umbral
- Ver historial de alertas disparadas
- Configurar si quiero alertas por categoría específica

**Criterios de aceptación:**
- Configuración flexible de umbrales (%)
- Alerta se dispara al crear gasto que supera umbral
- No spam: máximo 1 alerta por umbral por día
- Puedo desactivar alertas por categoría

**Complejidad estimada:** Media  
**Dependencias:** RF-008 (Presupuestos), RF-017 (Gastos)

**Consideraciones técnicas:**
- Endpoint: `POST /budgets/:id/alerts/configure`
- Cálculo en tiempo real al crear expense
- Opcional: Sistema de notificaciones (email, push)

---

### BL-002: Exportación de Datos
**Descripción:**  
Exportar gastos y reportes a formatos estándar para análisis externo o backup.

**User Story:**  
Como usuario, quiero exportar mis datos a CSV o Excel, para analizarlos con otras herramientas o hacer backups.

**Funcionalidades:**
- Exportar gastos de un período a CSV
- Exportar reporte mensual completo a Excel
- Exportar toda la base de datos (backup completo)
- Filtros flexibles antes de exportar

**Formatos soportados:**
- CSV (gastos individuales)
- Excel (reportes con múltiples sheets)
- JSON (backup completo)

**Endpoints propuestos:**
```
GET /exports/expenses?month=2025-06&format=csv
GET /exports/monthly-report?month=2025-06&format=xlsx
GET /exports/full-backup?format=json
```

**Complejidad estimada:** Media  
**Dependencias:** Todos los módulos implementados

**Consideraciones técnicas:**
- Usar librerías: csv-writer, exceljs
- Streaming para archivos grandes
- Límite de tamaño de exportación
- Rate limiting en endpoints de exportación

---

### BL-003: Importación de Datos Históricos
**Descripción:**  
Importar datos desde Excel o CSV para poblar la base de datos con historial.

**User Story:**  
Como usuario, quiero importar mi Excel de 2025, para tener mi historial completo en el sistema.

**Funcionalidades:**
- Importar desde Excel con formato específico
- Importar desde CSV con mapping de columnas
- Validación de datos antes de importar
- Preview de datos a importar
- Opción de rollback si algo falla

**Proceso:**
1. Upload de archivo
2. Validación y preview
3. Confirmación de usuario
4. Importación transaccional
5. Reporte de resultados (éxitos/errores)

**Endpoints propuestos:**
```
POST /imports/upload
POST /imports/validate
POST /imports/confirm
GET /imports/:id/status
```

**Complejidad estimada:** Alta  
**Dependencias:** Todos los módulos core

**Consideraciones técnicas:**
- Parser de Excel robusto
- Transacciones para atomicidad
- Queue para procesar en background
- Límite de tamaño de archivo (ej: 5MB, 10k rows)

---

## 🟡 Media Prioridad

### BL-004: Gastos Compartidos
**Descripción:**  
Soporte para gastos que se dividen entre múltiples personas.

**User Story:**  
Como usuario, quiero registrar gastos compartidos con mi pareja o roommates, para llevar control de quién debe qué.

**Funcionalidades:**
- Crear "participantes" (personas con las que comparto gastos)
- Marcar un gasto como compartido
- Definir porcentaje o monto que corresponde a cada uno
- Ver balance: quién me debe y a quién le debo
- Marcar gastos como "liquidados"

**Modelo de datos adicional:**
```
SharedExpense:
  - expense_id
  - participants: [{ user_id, percentage, amount }]
  - status: pending | settled
  - settled_at

Participant:
  - id
  - name
  - relationship: roommate | partner | friend | other
```

**Casos de uso:**
- Arriendo dividido 50/50 con roommate
- Supermercado 60/40 con pareja
- Cena grupal dividida en partes iguales

**Complejidad estimada:** Alta  
**Dependencias:** RF-017 (Gastos)

**Notas de diseño:**
- MVP personal: 100% de cada gasto es del usuario
- Esta feature requiere campo `user_id` en Expenses
- Considerar multi-tenancy si se escala a múltiples usuarios

---

### BL-005: Tags/Etiquetas
**Descripción:**  
Sistema de etiquetas flexibles para clasificación adicional de gastos.

**User Story:**  
Como usuario, quiero etiquetar gastos como "urgente", "planificado" o "imprevisto", para análisis adicional más allá de categorías.

**Funcionalidades:**
- Crear tags personalizados
- Asignar múltiples tags a un gasto
- Filtrar gastos por tag
- Reportes agrupados por tag
- Tags predefinidos comunes

**Tags sugeridos por defecto:**
- Planificado / Imprevisto
- Urgente / No urgente
- Necesario / Opcional
- Personal / Trabajo
- Deducible (para impuestos)

**Modelo de datos:**
```
Tag:
  - id
  - name
  - color
  - user_id (si es custom)

ExpenseTag (many-to-many):
  - expense_id
  - tag_id
```

**Complejidad estimada:** Media  
**Dependencias:** RF-017 (Gastos)

---

### BL-006: Comparación Temporal
**Descripción:**  
Comparar gastos entre diferentes períodos para identificar tendencias.

**User Story:**  
Como usuario, quiero comparar mis gastos de junio vs julio, para ver cómo cambió mi comportamiento.

**Funcionalidades:**
- Comparar mes a mes
- Comparar mismo mes de diferentes años
- Comparar trimestres
- Identificar variaciones significativas (>20%)
- Visualización de diferencias

**Reportes incluidos:**
- Gastos totales: mes A vs mes B
- Por categoría: qué aumentó/disminuyó
- Nuevos tipos de gasto o eliminados
- Promedio móvil de últimos 3/6 meses

**Endpoints propuestos:**
```
GET /reports/compare?period1=2025-06&period2=2025-07
GET /reports/compare-year?month=06&year1=2024&year2=2025
GET /reports/trends?category=alimentacion&months=6
```

**Complejidad estimada:** Media  
**Dependencias:** RF-020 (Reportes), suficiente data histórica

---

### BL-007: Proyecciones y Forecasting
**Descripción:**  
Proyectar gastos futuros basándose en histórico y tendencias.

**User Story:**  
Como usuario, quiero saber cuánto gastaré este mes, para planificar mejor mi cash flow.

**Funcionalidades:**
- Proyección de gasto mensual basada en promedio histórico
- Ajuste por estacionalidad (ej: más luz en invierno)
- Alertas si voy muy por encima de proyección
- Proyección de cuotas pendientes
- "A este ritmo, gastarás X este mes"

**Cálculos:**
- Promedio simple últimos 3/6 meses
- Promedio ponderado (más peso a meses recientes)
- Detección de outliers (meses atípicos)
- Suma de recurrentes + cuotas + estimado de variables

**Complejidad estimada:** Alta  
**Dependencias:** 6+ meses de data histórica

**Consideraciones técnicas:**
- Algoritmos estadísticos básicos
- Opcional: ML simple para mejorar predicciones
- Cache de proyecciones (actualizar diariamente)

---

## 🟢 Baja Prioridad

### BL-008: Dashboard Interactivo
**Descripción:**  
Dashboard visual con gráficos y métricas principales.

**User Story:**  
Como usuario, quiero ver un dashboard con gráficos, para entender rápidamente mi situación financiera.

**Componentes:**
- Gráfico de dona: gastos por categoría
- Gráfico de línea: evolución mensual
- KPIs: total gastado, vs presupuesto, promedio diario
- Próximos pagos de tarjeta
- Cuotas pendientes
- Top 5 gastos del mes

**Endpoints necesarios:**
```
GET /dashboard/overview?month=2025-06
GET /dashboard/charts/by-category?month=2025-06
GET /dashboard/charts/evolution?months=6
```

**Complejidad estimada:** Media  
**Dependencias:** Frontend (fuera de scope de API por ahora)

**Nota:** Esto es más un requisito de frontend, pero la API debe exponer los datos necesarios.

---

### BL-009: Multi-Moneda
**Descripción:**  
Soporte para múltiples monedas y conversiones automáticas.

**User Story:**  
Como usuario que viaja, quiero registrar gastos en USD o EUR, para llevar control completo.

**Funcionalidades:**
- Definir moneda por gasto
- Conversión automática a CLP usando tasa del día
- Reportes en moneda base (CLP) y original
- Histórico de tasas de cambio
- Gastos en moneda extranjera destacados

**Complejidad estimada:** Alta  
**Dependencias:** API externa de tasas (ej: exchangerate-api.com)

**Consideraciones:**
- No es prioridad para MVP (solo CLP)
- Requiere campos adicionales: currency, exchange_rate, original_amount
- Integración con API de tasas de cambio
- Caché de tasas diarias

---

### BL-010: Metas de Ahorro
**Descripción:**  
Definir metas de ahorro y trackear progreso.

**User Story:**  
Como usuario, quiero definir una meta de ahorrar 500k este mes, para alcanzar un objetivo financiero.

**Funcionalidades:**
- Crear meta de ahorro con monto objetivo
- Plazo de la meta (fecha límite)
- Tracking automático: ingresos - gastos = ahorro
- Progreso visual (%)
- Alertas si no voy a cumplir la meta

**Modelo de datos:**
```
SavingGoal:
  - id
  - name
  - target_amount
  - deadline
  - current_amount (calculado)
  - status: in_progress | achieved | failed
```

**Complejidad estimada:** Media  
**Dependencias:** Tracking de ingresos (actualmente no existe)

**Nota:** Requiere agregar concepto de "ingresos" que hoy no existe en el sistema.

---

### BL-011: Recordatorios
**Descripción:**  
Recordatorios automáticos para pagar cuentas o registrar gastos.

**User Story:**  
Como usuario, quiero que me recuerden pagar el arriendo el 1 de cada mes, para no olvidarlo.

**Funcionalidades:**
- Recordatorio de gastos recurrentes por pagar
- Recordatorio de vencimiento de tarjeta
- Recordatorio de registrar gastos (si no he registrado nada en X días)
- Configurar canales: email, push, SMS

**Complejidad estimada:** Alta  
**Dependencias:** Sistema de notificaciones, cron jobs

---

### BL-012: Análisis de Patrones
**Descripción:**  
Detectar patrones y sugerir optimizaciones automáticamente.

**User Story:**  
Como usuario, quiero que el sistema me diga "gastas mucho en delivery los viernes", para tomar mejores decisiones.

**Funcionalidades:**
- Detección de patrones: días de semana con más gasto
- Categorías que más crecen mes a mes
- Gastos atípicos (outliers)
- Sugerencias de optimización
- "Podrías ahorrar X si reducís Y en Z%"

**Ejemplos de insights:**
- "Gastas 30% más los fines de semana"
- "Uber Eats aumentó 50% vs mes pasado"
- "Tu gasto en suscripciones es 20% de tu presupuesto total"

**Complejidad estimada:** Alta  
**Dependencias:** Suficiente data histórica, algoritmos de análisis

---

### BL-013: Integración con Bancos (Open Banking)
**Descripción:**  
Conectar directamente con el banco para importar transacciones automáticamente.

**User Story:**  
Como usuario, quiero que mis gastos se registren automáticamente desde mi banco, para no tener que hacerlo manual.

**Funcionalidades:**
- Conectar cuenta bancaria vía API
- Importación automática de transacciones
- Categorización automática basada en ML
- Confirmación manual de categorizaciones
- Sync periódico (diario/semanal)

**Complejidad estimada:** Muy Alta  
**Dependencias:** 
- APIs bancarias (disponibilidad en Chile)
- Autenticación OAuth con bancos
- Cumplimiento regulatorio

**Notas:**
- En Chile aún no hay amplio soporte de Open Banking
- Requiere autenticación y seguridad robusta
- Alternativa: importación manual de CSV del banco

---

### BL-014: Modo Offline
**Descripción:**  
Soporte para registrar gastos offline y sincronizar después.

**User Story:**  
Como usuario que viaja, quiero registrar gastos sin internet, para sincronizarlos cuando tenga conexión.

**Funcionalidades:**
- Queue local de gastos pendientes de sync
- Sincronización automática al recuperar conexión
- Detección y resolución de conflictos
- Indicador visual de gastos no sincronizados

**Complejidad estimada:** Alta  
**Dependencias:** Frontend mobile, local storage, sync mechanism

**Nota:** Más relevante para app móvil que para API web.

---

## 🔵 Ideas Exploratorias

### BL-015: Gamificación
- Badges por cumplir presupuestos
- Streaks de días registrando gastos
- Challenges mensuales (ej: "reduce 10% en delivery")
- Leaderboard con amigos (si hay multi-usuario)

### BL-016: Asistente con IA
- Chatbot para consultas ("¿cuánto gasté en transporte este mes?")
- Sugerencias inteligentes de categorización
- Detección de gastos sospechosos/duplicados
- Resumen narrativo mensual generado por IA

### BL-017: Integración con Apps
- Integración con apps de delivery (Uber Eats, PedidosYa)
- Integración con apps de transporte (Uber, DiDi)
- Importación de recibos digitales
- Integración con calendario (gastos recurrentes en calendario)

### BL-018: Modo Familiar
- Múltiples usuarios en una familia
- Permisos y roles (admin, viewer)
- Presupuesto familiar compartido
- Asignaciones para hijos/dependientes

---

## Priorización Visual

```
Alta Prioridad (Post-MVP):
  ├─ BL-001: Alertas de Presupuesto
  ├─ BL-002: Exportación de Datos
  └─ BL-003: Importación de Datos

Media Prioridad:
  ├─ BL-004: Gastos Compartidos
  ├─ BL-005: Tags/Etiquetas
  ├─ BL-006: Comparación Temporal
  └─ BL-007: Proyecciones

Baja Prioridad:
  ├─ BL-008: Dashboard Interactivo
  ├─ BL-009: Multi-Moneda
  ├─ BL-010: Metas de Ahorro
  ├─ BL-011: Recordatorios
  ├─ BL-012: Análisis de Patrones
  ├─ BL-013: Open Banking
  └─ BL-014: Modo Offline

Ideas Exploratorias:
  ├─ BL-015: Gamificación
  ├─ BL-016: Asistente con IA
  ├─ BL-017: Integración con Apps
  └─ BL-018: Modo Familiar
```

---

## Criterios de Priorización

### Alta Prioridad
- Complementa funcionalidad core del MVP
- Alto valor para el usuario
- Complejidad media-baja
- No requiere dependencias externas críticas

### Media Prioridad
- Mejora significativa de UX
- Casos de uso comunes
- Complejidad variable
- Puede requerir algunas dependencias

### Baja Prioridad
- Nice to have
- Casos de uso específicos o menos frecuentes
- Complejidad alta
- Puede requerir dependencias externas complejas

### Ideas Exploratorias
- Innovadoras pero no validadas
- Complejidad desconocida
- Requieren investigación adicional
- Pueden no implementarse nunca

---

## Roadmap Tentativo

### Q1 2025 (Post-MVP)
- ✅ MVP completado
- BL-001: Alertas de Presupuesto
- BL-002: Exportación básica (CSV)

### Q2 2025
- BL-003: Importación de datos históricos
- BL-005: Tags/Etiquetas
- BL-006: Comparación temporal básica

### Q3 2025
- BL-004: Gastos Compartidos (si se necesita)
- BL-007: Proyecciones simples
- BL-008: Endpoints para dashboard

### Q4 2025
- Evaluación de features de baja prioridad
- Análisis de ideas exploratorias
- Decisión de siguientes pasos basado en uso real

**Nota:** Este roadmap es tentativo y se ajustará según feedback de uso real del MVP.

---

## Proceso de Evaluación

Antes de implementar cualquier feature del backlog:

1. **Validar necesidad**: ¿Resuelve un problema real que tengo?
2. **Estimar esfuerzo**: ¿Vale la pena el tiempo de desarrollo?
3. **Evaluar dependencias**: ¿Qué se necesita antes?
4. **Definir alcance**: MVP de la feature
5. **Implementar**: Incremental, con testing
6. **Medir impacto**: ¿Mejoró la experiencia?

---

## Contribuciones

Si se te ocurren más ideas:
1. Documéntalas en este archivo
2. Asigna prioridad tentativa
3. Describe user story y complejidad estimada
4. Evalúa dependencias

El backlog es un documento vivo que evoluciona con el proyecto.