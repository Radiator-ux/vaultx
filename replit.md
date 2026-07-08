# VaultX — Password Manager SaaS

A production-ready, full-stack Password Manager SaaS built with React, Express, and PostgreSQL. Features AES-256 vault encryption, JWT auth with refresh tokens, a beautiful dark/light glassmorphism UI, and a full-featured password vault.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080, proxied at /api)
- `pnpm --filter @workspace/password-manager run dev` — run the frontend (port 24311, proxied at /)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string (auto-provisioned)
- Required env: `JWT_SECRET`, `JWT_REFRESH_SECRET`, `ENCRYPTION_KEY` — auto-generated at setup

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- **Frontend**: React 18 + Vite, TailwindCSS v4, Framer Motion, Recharts, Wouter, React Hook Form, Sonner
- **API**: Express 5, Helmet, express-rate-limit, pino logger
- **DB**: PostgreSQL + Drizzle ORM
- **Auth**: JWT access tokens (15min) + refresh tokens (30d) in HttpOnly cookies, bcrypt
- **Vault encryption**: AES-256-CBC (crypto module), key derived from ENCRYPTION_KEY
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)

## Where things live

- `artifacts/api-server/src/routes/` — Express route handlers (auth, vault, generator, dashboard)
- `artifacts/api-server/src/lib/auth.ts` — JWT signing/verification + AES-256 encrypt/decrypt
- `artifacts/api-server/src/middlewares/authenticate.ts` — Bearer token middleware
- `lib/db/src/schema/` — Drizzle ORM schema (users, vault_entries)
- `lib/api-client-react/src/generated/` — Auto-generated React Query hooks (do not edit)
- `artifacts/password-manager/src/pages/` — Frontend pages (login, register, dashboard, vault, generator, settings)
- `artifacts/password-manager/src/contexts/auth-context.tsx` — Auth state + token getter management
- `artifacts/api-server/src/openapi.yaml` — Single source of truth for the API contract

## Architecture decisions

- **JWT with Bearer header**: Auth context sets `setAuthTokenGetter` so all generated API hooks automatically attach `Authorization: Bearer <token>`. The getter is set synchronously before React state update to prevent race conditions.
- **Access + Refresh tokens**: 15-minute access tokens returned in response body; 30-day refresh tokens in HttpOnly cookies via `/api/auth/refresh`.
- **AES-256-CBC encryption**: All vault passwords are encrypted at rest using a key derived from `ENCRYPTION_KEY`. Decryption only happens on explicit `GET /api/vault/:id/decrypt` requests.
- **Contract-first API**: OpenAPI spec drives Orval codegen for type-safe React Query hooks and Zod schemas. Run codegen after any spec change.
- **Trust proxy**: `app.set("trust proxy", 1)` is set so rate limiting works correctly behind Replit's proxy.

## Product

- **Auth**: Register, login, logout, forgot/reset/change password
- **Vault**: CRUD password entries with AES-256 encryption, search, filter by category, favorites, archive, tags, reveal/copy password, strength assessment
- **Generator**: Cryptographically secure password generation with configurable options (length 4–128, charsets, exclusions), entropy display
- **Dashboard**: Security score, total/weak/reused/favorite counts, strength breakdown chart, category pie chart, recent entries
- **Settings**: Profile update, theme switcher (dark/light), change password, vault export (JSON), account deletion

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Run `pnpm --filter @workspace/api-spec run codegen` after any OpenAPI spec change before editing frontend pages
- The `useGetMe` query requires `queryKey: getGetMeQueryKey()` in its options (required by Orval's generated typing)
- `useGetRecentEntries` takes `params` as the first argument directly (not wrapped in `{ params: ... }`)
- `setAuthTokenGetter` must be called before `setAccessTokenState` in the login function to avoid race conditions with React Query
- API server must be rebuilt (`pnpm run build`) after any route changes since it uses esbuild

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
