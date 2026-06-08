# Gusturi Autentice

Romanian traditional food e-commerce marketplace with roles: Admin, Vanzator (Seller), Cumparator (Buyer).

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/food-store run dev` — run the frontend (port from $PORT)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React 19 + Vite 7 + TailwindCSS 4 + shadcn/ui + Wouter routing
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (zod/v4), drizzle-zod
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/api-server/src/` — Express backend (routes/, middlewares/, db/)
- `artifacts/api-server/src/db/schema.ts` — DB schema (users, products, offers, sales)
- `artifacts/food-store/src/` — React frontend
- `artifacts/food-store/src/pages/` — all pages (home, login, register, seller/*, admin/*, sales, my-offers)
- `artifacts/food-store/src/context/AuthContext.tsx` — auth state
- `lib/api-spec/openapi.yaml` — OpenAPI spec (source of truth for API contract)
- `lib/api-client-react/src/generated/api.ts` — generated Orval hooks (do not edit manually)

## Architecture decisions

- Cookie-based sessions (not JWT): userId stored in signed cookie, checked by sessionMiddleware on protected routes
- Contract-first API: all routes defined in openapi.yaml, Orval generates typed React Query hooks
- Vite must dedupe `@tanstack/react-query` (and react, react-dom) to prevent multiple copies when lib imports it directly
- Offers auto-rejected server-side if proposedPrice < minimumPrice (buyer never sees the minimum)
- On purchase: product + all its offers are deleted from active tables, sale record created

## Product

- **Marketplace** (/) — All approved products with search + type filter; unauthenticated users see "login" CTA
- **Buyer** — Can buy fixed-price products directly or submit offers on negotiable products; views offer status at /my-offers; completes purchase after offer approved
- **Seller** — Manages products (/seller/products); reviews/approves/rejects incoming offers (/seller/offers); must be approved by admin
- **Admin** — Dashboard with stats (/admin); manages seller accounts (/admin/sellers: approve/disable/reactivate); views all sales (/sales)

## Demo accounts

- Admin: admin@email.com / admin
- Seller (approved): maria@email.com / maria123, gheorghe@email.com / gheorghe123
- Buyer: ion@email.com / ion123

## Gotchas

- Google Fonts @import must be first line in index.css before @import "tailwindcss" (PostCSS requirement)
- Express 5 params: always `parseInt(String(req.params.id))` — params are `string | string[]`
- `useLogout().mutateAsync()` takes no args (void) — do NOT pass `{}`
- `useGetMe` requires `queryKey` in query options
- Vite `resolve.dedupe` must include `@tanstack/react-query` alongside react/react-dom

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._
