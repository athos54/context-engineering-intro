# CLAUDE.md

This file provides comprehensive guidance to Claude Code when working with **JavaScript (Node.js + React)** code in this repository.

## Core Development Philosophy

### UI mobile first

- Mobile first es una filosofía de diseño que prioriza la experiencia en móvil por encima del escritorio. Diseña y desarrolla pensando primero en móviles y escala a tablet y desktop. La UI debe funcionar correctamente en **móvil, tablet y desktop**.

### KISS (Keep It Simple, Stupid)

La simplicidad debe ser un objetivo clave. Elige soluciones directas sobre complejas siempre que sea posible. El código simple es más fácil de entender, mantener y depurar.

### YAGNI (You Aren't Gonna Need It)

Evita construir funcionalidades “por si acaso”. Implementa características **solo cuando** sean necesarias.

### Design Principles

- **Dependency Inversion**: Los módulos de alto nivel no deben depender de los de bajo nivel. Ambos deben depender de abstracciones.
- **Open/Closed Principle**: Las entidades de software deben estar **abiertas a extensión** pero **cerradas a modificación**.
- **Single Responsibility**: Cada función, clase y módulo debe tener un propósito claro.
- **Fail Fast**: Valida pronto y falla rápido. Lanza errores temprano ante entradas inválidas.

---

## 🧱 Code Structure & Modularity

### File and Function Limits

- **Nunca crear un archivo > 500 líneas**. Si te acercas, refactoriza y divide en módulos.
- **Funciones < 50 líneas** con una única responsabilidad.
- **Clases < 100 líneas** y representando un concepto único.
- **Organiza el código en módulos claros**, agrupando por feature (vertical slice) o responsabilidad.
- **Longitud de línea máx: 100 caracteres** (enforzado por ESLint/Prettier).

### Project Architecture (Monorepo opcional)

Sigue arquitectura por vertical slices, con tests junto al código que prueban. Ejemplo (monorepo con `pnpm` workspaces o `npm workspaces`):

```
.
├─ apps/
│  ├─ server/                 # Node.js (Express/Fastify)
│  │  ├─ src/
│  │  │  ├─ index.js
│  │  │  ├─ app.js            # configuración del servidor
│  │  │  ├─ modules/
│  │  │  │  ├─ user/
│  │  │  │  │  ├─ controller.js
│  │  │  │  │  ├─ service.js
│  │  │  │  │  ├─ repository.js
│  │  │  │  │  └─ tests/
│  │  │  │  │     ├─ controller.test.js
│  │  │  │  │     └─ service.test.js
│  │  │  └─ shared/
│  │  └─ tests/
│  └─ web/                    # React (Vite o Next.js si aplica)
│     ├─ src/
│     │  ├─ app/
│     │  ├─ features/
│     │  │  └─ user/
│     │  │     ├─ components/
│     │  │     ├─ hooks/
│     │  │     └─ tests/
│     │  └─ shared/
│     └─ public/
├─ packages/
│  ├─ ui/                     # librería de componentes React compartidos
│  ├─ config/                 # eslint, prettier, tsconfig (si TS), etc.
│  └─ utils/                  # utilidades compartidas
├─ .env.example
├─ package.json               # scripts raíz y workspaces
└─ pnpm-workspace.yaml        # (si usas pnpm)
```

> Nota: Si el proyecto es de **un solo paquete**, colócalo en la raíz (p.ej. `/server` o `/web`) con la misma organización de `src/`.

---

## 🛠️ Development Environment

### Node & Package Management

Este proyecto usa **Node.js >= 18** (LTS recomendado) y **pnpm** (preferido por rendimiento) o npm/yarn.

```bash
# Instalar Node LTS (recomendado con nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
nvm install --lts
nvm use --lts

# Instalar pnpm (opcional, recomendado)
corepack enable  # en Node >= 16.10 activa pnpm/yarn
corepack prepare pnpm@latest --activate

# Instalar dependencias (raíz o en cada app)
pnpm install

# Añadir un paquete (no edites package.json a mano)
pnpm add axios

# Añadir dependencia de desarrollo
pnpm add -D eslint prettier vitest

# Eliminar un paquete
pnpm remove axios
```

### Development Commands (scripts típicos)

Ajusta según tu `package.json`:

```bash
# Ejecutar servidor/backend en modo dev (apps/server)
pnpm --filter "server" dev

# Ejecutar frontend React en modo dev (apps/web)
pnpm --filter "web" dev

# Ejecutar todos los tests
pnpm test

# Tests con cobertura
pnpm test -- --coverage

# Linting
pnpm lint

# Formateo
pnpm format

# Type checking (si usas TypeScript)
pnpm typecheck

# Build de producción
pnpm build
```

> **CRÍTICO**: **NUNCA** actualices dependencias **editando a mano** `package.json`. Usa siempre `pnpm add`/`pnpm remove`.

---

## 📋 Style & Conventions

### JavaScript Style Guide

- Usa **ES Modules** (`"type": "module"` en `package.json`) salvo que el entorno exija CJS.
- **Comillas dobles** para strings.
- **Comas finales** en estructuras multilínea.
- **Usa punto y coma** (consistencia).
- **ESLint + Prettier**: ESLint para reglas y Prettier para formato.
- **Longitud de línea 100** (regla ESLint/Prettier).
- Preferir **funciones puras** y composición.

Ejemplo de configuración mínima (compartida en `packages/config/.eslintrc.json` y extendida):

```json
{
  "root": true,
  "env": { "es2022": true, "node": true, "browser": true, "jest": true },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:jsx-a11y/recommended",
    "prettier"
  ],
  "parserOptions": { "ecmaVersion": 2022, "sourceType": "module" },
  "rules": {
    "max-len": ["error", { "code": 100, "ignoreUrls": true }],
    "quotes": ["error", "double"],
    "semi": ["error", "always"],
    "comma-dangle": ["error", "always-multiline"],
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  },
  "settings": { "react": { "version": "detect" } }
}
```

### JSDoc Standards

Usa **JSDoc** para todas las funciones/clases públicas. Ejemplo:

```js
/**
 * Calcula el precio con descuento.
 * @param {number} price - Precio original del producto.
 * @param {number} discountPercent - Descuento (0-100).
 * @param {number} [minAmount=0.01] - Precio mínimo permitido.
 * @returns {number} Precio final tras descuento.
 * @throws {RangeError} Si discountPercent no está entre 0 y 100.
 * @example
 * calculateDiscount(100, 20) // => 80
 */
export function calculateDiscount(price, discountPercent, minAmount = 0.01) {
  if (discountPercent < 0 || discountPercent > 100) {
    throw new RangeError("discountPercent must be 0..100");
  }
  const discounted = price * (1 - discountPercent / 100);
  return Math.max(discounted, minAmount);
}
```

### Naming Conventions

- **Variables y funciones**: `camelCase`
- **Clases y componentes React**: `PascalCase`
- **Constantes**: `UPPER_SNAKE_CASE`
- **Privado (convención)**: `_leadingUnderscore`
- **Archivos React**: `ComponentName.jsx` (o `.tsx` si TS)

---

## 🧪 Testing Strategy

### TDD

1. **Escribe el test primero**
2. **Míralo fallar**
3. **Código mínimo** para pasar el test
4. **Refactoriza** manteniendo en verde
5. **Repite**

### Herramientas

- **Vitest** (o **Jest**) para unit/integration en Node y React.
- **React Testing Library** para tests de UI.
- **Playwright** para E2E.

```js
// Ejemplo con Vitest + React Testing Library
import { render, screen } from "@testing-library/react";
import { describe, it, expect } from "vitest";
import { Button } from "./Button.jsx";

describe("Button", () => {
  it("renderiza con el texto", () => {
    render(<Button>Guardar</Button>);
    expect(
      screen.getByRole("button", { name: /guardar/i })
    ).toBeInTheDocument();
  });
});
```

### Organización de tests

- **Unit tests**: funciones/métodos en aislamiento
- **Integration**: interacción entre componentes (e.g., controller + service)
- **E2E**: flujos completos (Playwright)
- Coloca tests **junto al código** (`*.test.js`)
- **Cobertura 80%+**, priorizando rutas críticas

---

## 🚨 Error Handling

### Errores de dominio (Node)

```js
export class PaymentError extends Error {
  constructor(message = "Payment error") {
    super(message);
    this.name = "PaymentError";
  }
}

export class InsufficientFundsError extends PaymentError {
  constructor(required, available) {
    super(`Insufficient funds: required ${required}, available ${available}`);
    this.name = "InsufficientFundsError";
    this.required = required;
    this.available = available;
  }
}
```

### Middleware de errores (Express)

```js
// app.js
import express from "express";
export const app = express();
app.use(express.json());

// ... tus rutas aquí

// Error handler al final
app.use((err, req, res, next) => {
  req.log?.error?.(err); // si usas pino-http
  const status = err.name === "InsufficientFundsError" ? 402 : 500;
  res.status(status).json({ error: err.name, message: err.message });
});
```

### Error Boundaries (React)

```jsx
import React from "react";

export class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  componentDidCatch(error, info) {
    /* log */
  }
  render() {
    if (this.state.hasError) return <p>Lo sentimos, algo ha fallado.</p>;
    return this.props.children;
  }
}
```

---

## 🔧 Configuration Management

### Variables de entorno y settings

Usa `.env` + validación con **Zod** (o `env-var`, `envalid`).

```js
// config/env.js
import "dotenv/config";
import { z } from "zod";

const EnvSchema = z.object({
  NODE_ENV: z
    .enum(["development", "test", "production"])
    .default("development"),
  PORT: z.coerce.number().int().positive().default(3000),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url().optional(),
  API_KEY: z.string().min(1),
});

export const env = EnvSchema.parse(process.env);
```

Uso:

```js
import { env } from "./config/env.js";
app.listen(env.PORT, () => console.log(`Listening on ${env.PORT}`));
```

---

## 🏗️ Data Models and Validation

### Validación de entrada (Zod)

```js
import { z } from "zod";

export const ProductCreate = z.object({
  name: z.string().min(1).max(255),
  description: z.string().optional(),
  price: z.number().positive(),
  category: z.string(),
  tags: z.array(z.string()).default([]),
});

export function validateProduct(input) {
  return ProductCreate.parse(input);
}
```

Si usas **Prisma** o **Drizzle** para la base de datos, refleja los campos **tal cual** en los modelos de aplicación para evitar mapeos innecesarios.

---

## 🔄 API Route Standards (REST)

### Express Router

```js
// src/modules/products/router.js
import { Router } from "express";
import * as svc from "./service.js";
import { ProductCreate } from "./schemas.js";

export const router = Router();

router.get("/", async (req, res) => {
  const { skip = 0, limit = 100, category } = req.query;
  const products = await svc.list({ skip, limit, category });
  res.json(products);
});

router.post("/", async (req, res, next) => {
  try {
    const data = ProductCreate.parse(req.body);
    const created = await svc.create(data);
    res.status(201).json(created);
  } catch (err) {
    next(err);
  }
});
```

Convenciones:

- **Rutas**: `/api/v1/products`, `/api/v1/products/:id`, sub-recursos como `/api/v1/products/:id/reviews`.
- **Parámetros** consistentes (`:productId`, `:userId`).
- **Paginación**: `?skip=0&limit=100`.
- **Filtrado**: parámetros query (`?category=...`).
- **Errores** JSON (`{ error, message }`).

---

## 🗄️ Database Naming Standards

### Claves primarias por entidad (snake_case en DB, camelCase en app)

```sql
-- ✅ Estandarizado: PK por entidad
sessions.session_id UUID PRIMARY KEY
leads.lead_id UUID PRIMARY KEY
messages.message_id UUID PRIMARY KEY
```

### Convenciones de campos

```sql
-- PK: {entity}_id
session_id, lead_id, message_id
-- FK: {referenced_entity}_id
session_id REFERENCES sessions(session_id)
-- Timestamps: {action}_at
created_at, updated_at, started_at, expires_at
-- Booleans: is_{state}
is_connected, is_active, is_qualified
-- Counts: {entity}_count
message_count, lead_count
-- Durations: {property}_{unit}
duration_seconds, timeout_minutes
```

### Repositorio (Node)

```js
// repositories/leadRepository.js
export class LeadRepository {
  constructor(db) {
    this.db = db;
  }
  async findById(leadId) {
    return this.db.lead.findUnique({ where: { leadId } });
  }
  async create(data) {
    return this.db.lead.create({ data });
  }
}
```

---

## 📝 Documentation Standards

- Cada módulo con **JSDoc** explicando su propósito.
- Funciones públicas con **JSDoc completo**.
- Lógica compleja con comentarios `// Reason:` explicando decisiones.
- Mantén **README.md** actualizado con setup/ejemplos.
- Mantén **CHANGELOG.md** para el historial.
- Genera **OpenAPI** (p.ej. `zod-to-openapi` o `swagger-jsdoc`) si expones una API.

### Ejemplo de documentación de endpoint (Swagger JSDoc)

```js
/**
 * @openapi
 * /api/v1/products:
 *   get:
 *     summary: List all products
 *     parameters:
 *       - in: query
 *         name: skip
 *         schema: { type: integer, default: 0 }
 *       - in: query
 *         name: limit
 *         schema: { type: integer, default: 100 }
 *       - in: query
 *         name: category
 *         schema: { type: string }
 *     responses:
 *       200: { description: OK }
 */
```

---

## 🚀 Performance Considerations

- **Perf profiling**: `clinic.js`, `0x`, `node --prof`, React Profiler.
- **Cache**: `lru-cache` en capa de servicio.
- **I/O**: usa `Promise.all` y streaming (`pipeline`) donde aplique.
- **CPU-bound**: `worker_threads` o procesos hijos.
- **Frontend**: split de código, memoización (`useMemo`, `useCallback`), `React.lazy`.

```js
import LRU from "lru-cache";
const cache = new LRU({ max: 1000, ttl: 60_000 });
```

---

## 🛡️ Security Best Practices

- Nunca subas secretos: usa **.env** y variables de entorno.
- Valida **toda** entrada (Zod/Joi/Valibot).
- Queries parametrizadas (Prisma/Drizzle ya lo hacen por ti).
- **Rate limiting**: `express-rate-limit`.
- **Helmet** para cabeceras seguras.
- **CORS** explícito.
- **HTTPS** en producción.
- **Auth** y **RBAC**: JWT/OAuth; en frontend, guarda tokens **solo** en cookies seguras (httponly) cuando sea posible.

```js
import helmet from "helmet";
import rateLimit from "express-rate-limit";
app.use(helmet());
app.use(rateLimit({ windowMs: 60_000, max: 100 }));
```

---

## 🔍 Debugging Tools

```bash
# Node inspector\ nnode --inspect apps/server/src/index.js

# Pretty logs con pino
pnpm add pino pino-pretty -D
node apps/server/src/index.js | pino-pretty

# React DevTools y React Profiler
```

---

## 📊 Monitoring and Observability

- **Pino**/**Winston** con logs estructurados (JSON).
- **pino-http** para peticiones HTTP.
- **OpenTelemetry** para trazas/metrics.
- **web-vitals** en frontend.

```js
import pino from "pino";
export const logger = pino({ level: process.env.LOG_LEVEL || "info" });
logger.info({ event: "payment_processed", userId, amount, currency: "EUR" });
```

---

## 📚 Useful Resources

- Node.js: [https://nodejs.org/](https://nodejs.org/)
- Express: [https://expressjs.com/](https://expressjs.com/)
- Fastify: [https://fastify.dev/](https://fastify.dev/)
- React: [https://react.dev/](https://react.dev/)
- Vite: [https://vitejs.dev/](https://vitejs.dev/)
- ESLint: [https://eslint.org/](https://eslint.org/)
- Prettier: [https://prettier.io/](https://prettier.io/)
- Vitest: [https://vitest.dev/](https://vitest.dev/)
- Playwright: [https://playwright.dev/](https://playwright.dev/)
- Zod: [https://zod.dev/](https://zod.dev/)
- Prisma: [https://www.prisma.io/](https://www.prisma.io/)

---

## ⚠️ Important Notes

- **NUNCA ASUMAS NI ADIVINES**: si hay dudas, coméntalas claramente en la PR.
- **Verifica paths y nombres de módulos** antes de usar.
- **Mantén este CLAUDE.md actualizado** cuando cambien patrones o dependencias.
- **Testea tu código**: ninguna feature está completa sin tests.
- **Documenta decisiones** (ADR/Notion/README) para contexto futuro.

---

## 🔍 Search Command Requirements

**CRÍTICO**: Usa siempre `rg` (ripgrep) en vez de `grep/find`.

```bash
# ❌ No uses grepgrep -r "pattern" .
# ✅ Usa rgrg "pattern"

# ❌ No uses find -name
find . -name "*.js"
# ✅ Usa rgrg --files | rg "\.js$"
# o
rg --files -g "**/*.js"
```

**Reglas de enforcement (ejemplo de linter custom o pre-commit):**

```
(
  r"^grep\b(?!.*\|)",
  "Usa 'rg' (ripgrep) en vez de 'grep'"
),
(
  r"^find\s+\S+\s+-name\b",
  "Usa 'rg --files | rg pattern' o 'rg --files -g pattern'"
),
```

---

## 🔄 Git Workflow

### Branch Strategy

- `main` - Código listo para producción
- `develop` - Integración de features
- `feature/*` - Nuevas funcionalidades
- `fix/*` - Correcciones de bugs
- `docs/*` - Actualizaciones de docs
- `refactor/*` - Refactors
- `test/*` - Tests

### Commit Message Format

**Nunca** incluyas “claude code” ni “escrito por claude” en commits.

```
<type>(<scope>): <subject>
<body>
<footer>
```

Tipos: feat, fix, docs, style, refactor, test, chore

Ejemplo:

```
feat(auth): add two-factor authentication
- Implement TOTP generation and validation
- Add QR code generation for authenticator apps
- Update user model with 2FA fields
Closes #123
```

---

## 🚀 GitHub Flow Workflow Summary

`main` (protected) ←── PR ←── `feature/your-feature`

**Daily Workflow:**

1. `git checkout main && git pull origin main`
2. `git checkout -b feature/new-feature`
3. Cambios + tests
4. `git push origin feature/new-feature`
5. PR → Review → Merge to main

---

_Este documento es vivo. Actualízalo según evolucione el proyecto y surjan nuevos patrones._
