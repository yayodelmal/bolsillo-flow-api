# Ejemplos de Uso - Categories API

## Overview
Este documento contiene ejemplos prácticos de uso de los endpoints de categorías, incluyendo casos de uso comunes, flujos completos y manejo de errores.

---

## Casos de Uso Comunes

### 1. Setup Inicial - Crear Estructura de Categorías

#### Paso 1: Crear Categorías Padre

**Request: Crear "Departamento"**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "Departamento",
  "description": "Gastos del departamento",
  "color": "#3498DB"
}
```

**Response:**
```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "Departamento",
  "description": "Gastos del departamento",
  "color": "#3498DB",
  "parentId": null,
  "isActive": true,
  "createdAt": "2025-01-02T10:00:00.000Z",
  "updatedAt": "2025-01-02T10:00:00.000Z",
  "deletedAt": null
}
```

---

**Request: Crear "Transporte"**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "Transporte",
  "description": "Gastos de movilización",
  "color": "#E74C3C"
}
```

**Response:**
```json
{
  "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "name": "Transporte",
  "description": "Gastos de movilización",
  "color": "#E74C3C",
  "parentId": null,
  "isActive": true,
  "createdAt": "2025-01-02T10:01:00.000Z",
  "updatedAt": "2025-01-02T10:01:00.000Z",
  "deletedAt": null
}
```

---

**Request: Crear "Alimentación"**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "Alimentación",
  "description": "Gastos de comida",
  "color": "#2ECC71"
}
```

**Response:**
```json
{
  "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
  "name": "Alimentación",
  "description": "Gastos de comida",
  "color": "#2ECC71",
  "parentId": null,
  "isActive": true,
  "createdAt": "2025-01-02T10:02:00.000Z",
  "updatedAt": "2025-01-02T10:02:00.000Z",
  "deletedAt": null
}
```

---

#### Paso 2: Crear Categorías Hijas

**Request: Crear "Servicios Básicos" bajo "Departamento"**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "Servicios Básicos",
  "description": "Luz, agua, internet",
  "color": "#3498DB",
  "parentId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**Response:**
```json
{
  "id": "d4e5f6a7-b8c9-0123-def1-234567890123",
  "name": "Servicios Básicos",
  "description": "Luz, agua, internet",
  "color": "#3498DB",
  "parentId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "isActive": true,
  "createdAt": "2025-01-02T10:03:00.000Z",
  "updatedAt": "2025-01-02T10:03:00.000Z",
  "deletedAt": null
}
```

---

**Request: Crear "Uber" bajo "Transporte"**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "Uber",
  "parentId": "b2c3d4e5-f6a7-8901-bcde-f12345678901"
}
```

**Response:**
```json
{
  "id": "e5f6a7b8-c9d0-1234-ef12-345678901234",
  "name": "Uber",
  "description": null,
  "color": null,
  "parentId": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "isActive": true,
  "createdAt": "2025-01-02T10:04:00.000Z",
  "updatedAt": "2025-01-02T10:04:00.000Z",
  "deletedAt": null
}
```

---

**Request: Crear "Supermercado" bajo "Alimentación"**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "Supermercado",
  "parentId": "c3d4e5f6-a7b8-9012-cdef-123456789012"
}
```

**Response:**
```json
{
  "id": "f6a7b8c9-d0e1-2345-f123-456789012345",
  "name": "Supermercado",
  "description": null,
  "color": null,
  "parentId": "c3d4e5f6-a7b8-9012-cdef-123456789012",
  "isActive": true,
  "createdAt": "2025-01-02T10:05:00.000Z",
  "updatedAt": "2025-01-02T10:05:00.000Z",
  "deletedAt": null
}
```

---

### 2. Consultar Estructura de Categorías

#### Obtener Solo Categorías Padre

**Request:**
```http
GET /api/categories?parentOnly=true
```

**Response:**
```json
[
  {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Departamento",
    "description": "Gastos del departamento",
    "color": "#3498DB",
    "parentId": null,
    "isActive": true,
    "createdAt": "2025-01-02T10:00:00.000Z",
    "updatedAt": "2025-01-02T10:00:00.000Z",
    "deletedAt": null,
    "children": []
  },
  {
    "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
    "name": "Transporte",
    "description": "Gastos de movilización",
    "color": "#E74C3C",
    "parentId": null,
    "isActive": true,
    "createdAt": "2025-01-02T10:01:00.000Z",
    "updatedAt": "2025-01-02T10:01:00.000Z",
    "deletedAt": null,
    "children": []
  },
  {
    "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
    "name": "Alimentación",
    "description": "Gastos de comida",
    "color": "#2ECC71",
    "parentId": null,
    "isActive": true,
    "createdAt": "2025-01-02T10:02:00.000Z",
    "updatedAt": "2025-01-02T10:02:00.000Z",
    "deletedAt": null,
    "children": []
  }
]
```

---

#### Obtener Todas las Categorías con Jerarquía

**Request:**
```http
GET /api/categories
```

**Response:**
```json
[
  {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Departamento",
    "description": "Gastos del departamento",
    "color": "#3498DB",
    "parentId": null,
    "isActive": true,
    "createdAt": "2025-01-02T10:00:00.000Z",
    "updatedAt": "2025-01-02T10:00:00.000Z",
    "deletedAt": null,
    "children": [
      {
        "id": "d4e5f6a7-b8c9-0123-def1-234567890123",
        "name": "Servicios Básicos",
        "description": "Luz, agua, internet",
        "color": "#3498DB",
        "parentId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "isActive": true,
        "createdAt": "2025-01-02T10:03:00.000Z",
        "updatedAt": "2025-01-02T10:03:00.000Z",
        "deletedAt": null
      }
    ]
  },
  {
    "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
    "name": "Transporte",
    "description": "Gastos de movilización",
    "color": "#E74C3C",
    "parentId": null,
    "isActive": true,
    "createdAt": "2025-01-02T10:01:00.000Z",
    "updatedAt": "2025-01-02T10:01:00.000Z",
    "deletedAt": null,
    "children": [
      {
        "id": "e5f6a7b8-c9d0-1234-ef12-345678901234",
        "name": "Uber",
        "description": null,
        "color": null,
        "parentId": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "isActive": true,
        "createdAt": "2025-01-02T10:04:00.000Z",
        "updatedAt": "2025-01-02T10:04:00.000Z",
        "deletedAt": null
      }
    ]
  }
]
```

---

#### Obtener Hijos de una Categoría Específica

**Request:**
```http
GET /api/categories/b2c3d4e5-f6a7-8901-bcde-f12345678901/children
```

**Response:**
```json
[
  {
    "id": "e5f6a7b8-c9d0-1234-ef12-345678901234",
    "name": "Uber",
    "description": null,
    "color": null,
    "parentId": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
    "isActive": true,
    "createdAt": "2025-01-02T10:04:00.000Z",
    "updatedAt": "2025-01-02T10:04:00.000Z",
    "deletedAt": null
  }
]
```

---

### 3. Actualizar Categorías

#### Actualización Completa (PUT)

**Request: Actualizar nombre y color**
```http
PUT /api/categories/b2c3d4e5-f6a7-8901-bcde-f12345678901
Content-Type: application/json

{
  "name": "Transporte y Movilidad",
  "description": "Gastos de movilización y transporte",
  "color": "#E67E22",
  "parentId": null,
  "isActive": true
}
```

**Response:**
```json
{
  "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "name": "Transporte y Movilidad",
  "description": "Gastos de movilización y transporte",
  "color": "#E67E22",
  "parentId": null,
  "isActive": true,
  "createdAt": "2025-01-02T10:01:00.000Z",
  "updatedAt": "2025-01-02T12:30:00.000Z",
  "deletedAt": null
}
```

---

#### Actualización Parcial (PATCH)

**Request: Solo cambiar color**
```http
PATCH /api/categories/c3d4e5f6-a7b8-9012-cdef-123456789012
Content-Type: application/json

{
  "color": "#27AE60"
}
```

**Response:**
```json
{
  "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
  "name": "Alimentación",
  "description": "Gastos de comida",
  "color": "#27AE60",
  "parentId": null,
  "isActive": true,
  "createdAt": "2025-01-02T10:02:00.000Z",
  "updatedAt": "2025-01-02T12:35:00.000Z",
  "deletedAt": null
}
```

---

#### Mover Categoría (Cambiar Parent)

**Request: Mover "Delivery" de "Transporte" a "Alimentación"**
```http
PATCH /api/categories/a7b8c9d0-e1f2-3456-a123-567890123456
Content-Type: application/json

{
  "parentId": "c3d4e5f6-a7b8-9012-cdef-123456789012"
}
```

**Response:**
```json
{
  "id": "a7b8c9d0-e1f2-3456-a123-567890123456",
  "name": "Delivery",
  "description": null,
  "color": null,
  "parentId": "c3d4e5f6-a7b8-9012-cdef-123456789012",
  "isActive": true,
  "createdAt": "2025-01-02T10:06:00.000Z",
  "updatedAt": "2025-01-02T13:00:00.000Z",
  "deletedAt": null
}
```

---

### 4. Eliminar y Restaurar

#### Soft Delete de Categoría Hija

**Request:**
```http
DELETE /api/categories/e5f6a7b8-c9d0-1234-ef12-345678901234
```

**Response:**
```http
HTTP/1.1 204 No Content
```

---

#### Verificar que Está Eliminada

**Request: Listar sin includeDeleted**
```http
GET /api/categories
```

**Response:**
```json
[
  {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Departamento",
    ...
  },
  {
    "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
    "name": "Transporte y Movilidad",
    "children": []
  }
]
```

**Nota:** "Uber" ya no aparece en children de "Transporte".

---

**Request: Listar con includeDeleted**
```http
GET /api/categories?includeDeleted=true
```

**Response:**
```json
[
  {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Departamento",
    ...
  },
  {
    "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
    "name": "Transporte y Movilidad",
    "children": [
      {
        "id": "e5f6a7b8-c9d0-1234-ef12-345678901234",
        "name": "Uber",
        "deletedAt": "2025-01-02T14:00:00.000Z"
      }
    ]
  }
]
```

---

#### Restaurar Categoría Eliminada

**Request:**
```http
PATCH /api/categories/e5f6a7b8-c9d0-1234-ef12-345678901234/restore
```

**Response:**
```json
{
  "id": "e5f6a7b8-c9d0-1234-ef12-345678901234",
  "name": "Uber",
  "description": null,
  "color": null,
  "parentId": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "isActive": true,
  "createdAt": "2025-01-02T10:04:00.000Z",
  "updatedAt": "2025-01-02T14:30:00.000Z",
  "deletedAt": null
}
```

---

#### Soft Delete de Categoría Padre (Cascada)

**Request:**
```http
DELETE /api/categories/b2c3d4e5-f6a7-8901-bcde-f12345678901
```

**Response:**
```http
HTTP/1.1 204 No Content
```

**Efecto:** 
- "Transporte y Movilidad" marcado como deleted
- "Uber" (hijo) también marcado como deleted
- Ambos desaparecen de listados normales

---

### 5. Manejo de Errores

#### Error: Crear Categoría sin Nombre

**Request:**
```http
POST /api/categories
Content-Type: application/json

{
  "description": "Sin nombre"
}
```

**Response:**
```json
{
  "statusCode": 400,
  "message": [
    "name should not be empty",
    "name must be longer than or equal to 3 characters"
  ],
  "error": "Bad Request"
}
```

---

#### Error: Nombre Muy Corto

**Request:**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "AB"
}
```

**Response:**
```json
{
  "statusCode": 400,
  "message": [
    "name must be longer than or equal to 3 characters"
  ],
  "error": "Bad Request"
}
```

---

#### Error: Color en Formato Incorrecto

**Request:**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "Salud",
  "color": "rojo"
}
```

**Response:**
```json
{
  "statusCode": 400,
  "message": [
    "color must be a valid hex color (#RRGGBB)"
  ],
  "error": "Bad Request"
}
```

---

#### Error: Parent ID No Existe

**Request:**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "Nueva Categoría",
  "parentId": "00000000-0000-0000-0000-000000000000"
}
```

**Response:**
```json
{
  "statusCode": 404,
  "message": "Parent category not found",
  "error": "Not Found"
}
```

---

#### Error: Intentar Crear Tercer Nivel

**Request:**
```http
POST /api/categories
Content-Type: application/json

{
  "name": "Electricidad",
  "parentId": "d4e5f6a7-b8c9-0123-def1-234567890123"
}
```

**Nota:** "Servicios Básicos" ya es hijo de "Departamento".

**Response:**
```json
{
  "statusCode": 422,
  "message": "Cannot create category with more than 2 levels of hierarchy",
  "error": "Unprocessable Entity"
}
```

---

#### Error: Eliminar Categoría con Expense Types

**Request:**
```http
DELETE /api/categories/a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Nota:** "Departamento" tiene expense types activos asociados.

**Response:**
```json
{
  "statusCode": 422,
  "message": "Cannot delete category with active expense types",
  "error": "Unprocessable Entity"
}
```

---

#### Error: Restaurar Hija con Padre Eliminado

**Escenario:**
1. "Transporte" (padre) está soft-deleted
2. "Uber" (hijo) está soft-deleted
3. Intentar restaurar solo "Uber"

**Request:**
```http
PATCH /api/categories/e5f6a7b8-c9d0-1234-ef12-345678901234/restore
```

**Response:**
```json
{
  "statusCode": 422,
  "message": "Cannot restore category with deleted parent. Restore parent first.",
  "error": "Unprocessable Entity"
}
```

---

#### Error: Categoría No Encontrada

**Request:**
```http
GET /api/categories/99999999-9999-9999-9999-999999999999
```

**Response:**
```json
{
  "statusCode": 404,
  "message": "Category not found",
  "error": "Not Found"
}
```

---

#### Error: Crear Bucle en Jerarquía

**Escenario:**
1. "A" es padre de "B"
2. Intentar hacer "B" padre de "A"

**Request:**
```http
PATCH /api/categories/a1b2c3d4-e5f6-7890-abcd-ef1234567890
Content-Type: application/json

{
  "parentId": "d4e5f6a7-b8c9-0123-def1-234567890123"
}
```

**Nota:** Intentando hacer "Departamento" hijo de "Servicios Básicos".

**Response:**
```json
{
  "statusCode": 422,
  "message": "Cannot create circular reference in category hierarchy",
  "error": "Unprocessable Entity"
}
```

---

#### Error: Mover Categoría con Hijos bajo Otra

**Escenario:**
"Departamento" tiene hijo "Servicios Básicos".
Intentar mover "Departamento" bajo "Alimentación".

**Request:**
```http
PATCH /api/categories/a1b2c3d4-e5f6-7890-abcd-ef1234567890
Content-Type: application/json

{
  "parentId": "c3d4e5f6-a7b8-9012-cdef-123456789012"
}
```

**Response:**
```json
{
  "statusCode": 422,
  "message": "Cannot move category with children under another parent (would exceed 2 levels)",
  "error": "Unprocessable Entity"
}
```

---

## Flujos Completos

### Flujo 1: Reorganización de Categorías

**Objetivo:** Reorganizar estructura de categorías después de revisar gastos.

**Estado Inicial:**
```
Transporte (padre)
  └─ Uber (hijo)
  └─ Delivery (hijo)

Alimentación (padre)
  └─ Supermercado (hijo)
```

**Objetivo Final:**
```
Transporte (padre)
  └─ Uber (hijo)

Alimentación (padre)
  └─ Supermercado (hijo)
  └─ Delivery (hijo)
```

---

**Paso 1: Obtener ID de "Delivery"**
```http
GET /api/categories/b2c3d4e5-f6a7-8901-bcde-f12345678901/children
```

**Response:**
```json
[
  {
    "id": "e5f6a7b8-c9d0-1234-ef12-345678901234",
    "name": "Uber"
  },
  {
    "id": "a7b8c9d0-e1f2-3456-a123-567890123456",
    "name": "Delivery"
  }
]
```

---

**Paso 2: Mover "Delivery" a "Alimentación"**
```http
PATCH /api/categories/a7b8c9d0-e1f2-3456-a123-567890123456
Content-Type: application/json

{
  "parentId": "c3d4e5f6-a7b8-9012-cdef-123456789012"
}
```

**Response:**
```json
{
  "id": "a7b8c9d0-e1f2-3456-a123-567890123456",
  "name": "Delivery",
  "parentId": "c3d4e5f6-a7b8-9012-cdef-123456789012",
  ...
}
```

---

**Paso 3: Verificar Nueva Estructura**
```http
GET /api/categories?parentOnly=true
```

**Response:**
```json
[
  {
    "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
    "name": "Transporte",
    "children": [
      {
        "id": "e5f6a7b8-c9d0-1234-ef12-345678901234",
        "name": "Uber"
      }
    ]
  },
  {
    "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
    "name": "Alimentación",
    "children": [
      {
        "id": "f6a7b8c9-d0e1-2345-f123-456789012345",
        "name": "Supermercado"
      },
      {
        "id": "a7b8c9d0-e1f2-3456-a123-567890123456",
        "name": "Delivery"
      }
    ]
  }
]
```

✅ **Reorganización completada**

---

### Flujo 2: Limpieza de Categorías No Usadas

**Objetivo:** Eliminar categorías que ya no se usan.

**Paso 1: Identificar categorías para eliminar**
```http
GET /api/categories
```

Identificar manualmente o con lógica adicional cuáles no tienen expense types.

---

**Paso 2: Intentar eliminar**
```http
DELETE /api/categories/x1y2z3a4-b5c6-d7e8-f9a0-b1c2d3e4f5a6
```

**Escenario A: Sin dependencias**
```http
HTTP/1.1 204 No Content
```

**Escenario B: Con dependencias**
```json
{
  "statusCode": 422,
  "message": "Cannot delete category with active expense types",
  "error": "Unprocessable Entity"
}
```

---

**Paso 3: Si hay dependencias, primero mover o eliminar expense types**

Esto requiere llamadas a endpoints de expense-types (fuera del scope de este documento).

---

**Paso 4: Retry eliminación después de limpiar dependencias**
```http
DELETE /api/categories/x1y2z3a4-b5c6-d7e8-f9a0-b1c2d3e4f5a6
```

```http
HTTP/1.1 204 No Content
```

✅ **Categoría eliminada**

---

## Testing Tips

### 1. Crear Estructura Completa de Prueba

```bash
# Script para crear estructura de test
curl -X POST http://localhost:3000/api/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Parent","color":"#FF0000"}'

# Guardar ID del response
PARENT_ID="..."

curl -X POST http://localhost:3000/api/categories \
  -H "Content-Type: application/json" \
  -d "{\"name\":\"Test Child\",\"parentId\":\"$PARENT_ID\"}"
```

---

### 2. Validar Jerarquía

```bash
# Obtener categoría con hijos
curl http://localhost:3000/api/categories/$PARENT_ID

# Verificar que children array no esté vacío
```

---

### 3. Test de Validaciones

```bash
# Test: Nombre vacío
curl -X POST http://localhost:3000/api/categories \
  -H "Content-Type: application/json" \
  -d '{"description":"No name"}' \
  | jq '.statusCode'
# Esperar: 400

# Test: Color inválido
curl -X POST http://localhost:3000/api/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"Test","color":"invalid"}' \
  | jq '.statusCode'
# Esperar: 400
```

---

### 4. Test de Soft Delete

```bash
# Eliminar
curl -X DELETE http://localhost:3000/api/categories/$CATEGORY_ID

# Verificar que no aparece en lista normal
curl http://localhost:3000/api/categories | jq 'map(.id)' | grep $CATEGORY_ID
# Esperar: vacío

# Verificar que aparece con includeDeleted
curl "http://localhost:3000/api/categories?includeDeleted=true" \
  | jq 'map(.id)' | grep $CATEGORY_ID
# Esperar: ID presente
```

---

### 5. Test de Restauración

```bash
# Restaurar
curl -X PATCH http://localhost:3000/api/categories/$CATEGORY_ID/restore

# Verificar que aparece en lista normal
curl http://localhost:3000/api/categories | jq 'map(.id)' | grep $CATEGORY_ID
# Esperar: ID presente
```

---

## Postman Collection

### Variables de Entorno

```json
{
  "baseUrl": "http://localhost:3000/api",
  "categoryParentId": "",
  "categoryChildId": "",
  "categoryToDelete": ""
}
```

---

### Requests Principales

1. **Create Parent Category**
   - Method: POST
   - URL: `{{baseUrl}}/categories`
   - Body: Raw JSON
   - Tests: Set `categoryParentId` from response

2. **Create Child Category**
   - Method: POST
   - URL: `{{baseUrl}}/categories`
   - Body: Include `parentId: {{categoryParentId}}`
   - Tests: Set `categoryChildId` from response

3. **List All Categories**
   - Method: GET
   - URL: `{{baseUrl}}/categories`

4. **Get Category by ID**
   - Method: GET
   - URL: `{{baseUrl}}/categories/{{categoryParentId}}`

5. **Update Category**
   - Method: PATCH
   - URL: `{{baseUrl}}/categories/{{categoryParentId}}`
   - Body: Campos a actualizar

6. **Delete Category**
   - Method: DELETE
   - URL: `{{baseUrl}}/categories/{{categoryChildId}}`

7. **Restore Category**
   - Method: PATCH
   - URL: `{{baseUrl}}/categories/{{categoryChildId}}/restore`

---

## Referencias

- [Endpoints API](../endpoints.md)
- [Business Rules - Categories](../../03-data-model/business-rules.md#categorías)
- [Requirements - Categories](../../01-product/requirements.md#categorías)