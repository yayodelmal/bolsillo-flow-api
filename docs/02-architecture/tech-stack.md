# Tech Stack

## Overview
Bolsillo Flow API está construida con tecnologías modernas y estables, priorizando developer experience, type safety y escalabilidad.

---

## Core Stack

### Runtime & Language
- **Node.js**: `v20.19.0` (LTS)
  - Runtime de JavaScript/TypeScript
  - Excelente ecosistema y performance
  - Soporte de largo plazo

- **TypeScript**: `~5.3.0`
  - Type safety en todo el codebase
  - Mejor developer experience con autocompletado
  - Menos bugs en runtime

### Framework
- **NestJS**: `^10.0.0`
  - Framework progresivo para Node.js
  - Arquitectura modular y escalable
  - Decorators, Dependency Injection, Guards, Interceptors
  - Inspirado en Angular, pero para backend
  - Excelente para aplicaciones enterprise

**¿Por qué NestJS?**
- ✅ Estructura clara y opinada
- ✅ TypeScript-first
- ✅ Módulos reutilizables
- ✅ Testing integrado
- ✅ Gran ecosistema de librerías

---

## Database

### Database Engine
- **PostgreSQL**: `15`
  - Base de datos relacional robusta
  - ACID compliant
  - Excelente para datos financieros (precisión decimal)
  - JSON support para campos flexibles (installment_config)
  - Ampliamente adoptada y documentada

**¿Por qué PostgreSQL?**
- ✅ Transacciones ACID (crítico para finanzas)
- ✅ Tipos de datos precisos (DECIMAL para montos)
- ✅ Soporte JSON para configuraciones flexibles
- ✅ Performance excelente
- ✅ Comunidad activa

### ORM
- **Prisma**: `^5.0.0`
  - ORM type-safe moderno
  - Schema declarativo
  - Migraciones automáticas
  - Prisma Client generado
  - Prisma Studio para explorar datos

**¿Por qué Prisma?**
- ✅ Type safety end-to-end
- ✅ Schema como single source of truth
- ✅ Migraciones simples y versionadas
- ✅ Excelente DX con autocompletado
- ✅ Performance optimizada
- ✅ Prisma Studio como admin UI

---

## Validation & Transformation

- **class-validator**: `^0.14.0`
  - Validación declarativa con decorators
  - Integración nativa con NestJS
  - Mensajes de error customizables

- **class-transformer**: `^0.5.1`
  - Transformación de plain objects a class instances
  - Serialización/deserialización automática
  - Control fino de qué exponer en DTOs

**Ejemplo:**
```typescript
export class CreateCategoryDto {
  @IsString()
  @MinLength(3)
  @MaxLength(50)
  name: string;

  @IsOptional()
  @IsString()
  description?: string;

  @IsOptional()
  @IsUUID()
  parentId?: string;
}
```

---

## Development Tools

### Package Manager
- **npm**: Default de Node.js
  - Simple y ampliamente soportado
  - Lock file para reproducibilidad

### Code Quality

- **ESLint**: `^8.0.0`
  - Linting de código
  - Reglas de NestJS y TypeScript
  - Enforce best practices

- **Prettier**: `^3.0.0`
  - Code formatting automático
  - Consistencia en todo el codebase
  - Integración con ESLint

### Testing (Futuro)

- **Jest**: Incluido con NestJS
  - Unit tests
  - Integration tests
  - Coverage reports

- **Supertest**: Testing de endpoints HTTP
  - E2E tests de API

---

## Environment & Configuration

- **dotenv**: Manejo de variables de entorno
  - `.env` para desarrollo local
  - `.env.example` como template
  - Nunca commitear `.env` real

- **@nestjs/config**: Módulo de configuración de NestJS
  - Type-safe configuration
  - Validación de env vars
  - Diferentes configs por ambiente

---

## Utilities

- **date-fns**: Manipulación de fechas
  - Lightweight (vs Moment.js)
  - Inmutable y funcional
  - Tree-shakeable
  - Útil para cálculos de billing cycles

- **uuid**: Generación de UUIDs
  - IDs únicos y seguros
  - Standard v4

---

## Future Considerations

### Potenciales Adiciones

**Caché:**
- Redis: Para cachear reportes pesados
- @nestjs/cache-manager: Integración con NestJS

**Autenticación:**
- Passport.js: Estrategias de auth
- JWT: Tokens
- bcrypt: Hashing de passwords

**Documentación API:**
- Swagger/OpenAPI: Auto-generación de docs
- @nestjs/swagger: Integración con NestJS

**Monitoring:**
- Winston/Pino: Logging estructurado
- Sentry: Error tracking
- Prometheus: Métricas

**CI/CD:**
- GitHub Actions: Automatización
- Docker: Containerización
- Docker Compose: Orquestación local

---

## Version Matrix

| Herramienta      | Versión      | Status    |
|------------------|--------------|-----------|
| Node.js          | 20.19.0      | ✅ LTS    |
| TypeScript       | ~5.3.0       | ✅ Stable |
| NestJS           | ^10.0.0      | ✅ Stable |
| PostgreSQL       | 15           | ✅ Stable |
| Prisma           | ^5.0.0       | ✅ Stable |
| class-validator  | ^0.14.0      | ✅ Stable |
| class-transformer| ^0.5.1       | ✅ Stable |

---

## Architecture Patterns

### Design Patterns Utilizados

1. **Dependency Injection**
   - Provisto por NestJS
   - Servicios inyectables
   - Testabilidad mejorada

2. **Repository Pattern**
   - Prisma Service actúa como repository
   - Abstracción de acceso a datos
   - Facilita testing con mocks

3. **DTO Pattern**
   - Validación de entrada
   - Transformación de salida
   - Type safety en API boundaries

4. **Domain-Driven Design (lite)**
   - Módulos organizados por dominio
   - `financial/`, `payment-methods/`, `automation/`
   - Bounded contexts claros

---

## Performance Considerations

### Database
- Indexes en campos frecuentemente consultados
- Queries optimizadas con Prisma
- Connection pooling automático

### API
- Paginación en listados grandes
- Lazy loading de relaciones
- Caching de reportes (futuro)

### Code
- Tree-shaking con ES modules
- Lazy loading de módulos pesados
- Evitar N+1 queries con Prisma includes

---

## Security Considerations

### Database
- Prepared statements (Prisma automático)
- Validación de inputs
- Soft deletes para auditoría

### API
- Validación exhaustiva con class-validator
- Guards para proteger endpoints (futuro)
- Rate limiting (futuro)
- CORS configurado apropiadamente

### Environment
- Secrets en variables de entorno
- `.env` en `.gitignore`
- No hardcodear credenciales

---

## References

- [NestJS Documentation](https://docs.nestjs.com/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/15/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)