# Repository Guidelines

CrossStitchDraftGenerator converts AI-generated images into cross-stitch patterns with DMC palette mapping. Built with Astro 6 SSR, React 19, TypeScript strict, Tailwind 4, and Supabase auth, deployed to Cloudflare Workers.

## Hard Rules

- **Never commit secrets.** `SUPABASE_URL` and `SUPABASE_KEY` must live in `.env` / `.dev.vars` only. Both files are gitignored.
- **Always enable RLS on new Supabase tables** with granular per-operation, per-role policies. Tables without RLS leak data.
- **No Next.js directives** ("use client", "use server") in React components. This is Astro, not Next.js.
- **Use `cn()` helper for conditional classes.** Never concatenate class strings manually — always use `cn()` from `@/lib/utils` (clsx + tailwind-merge).
- **API routes must export uppercase `GET`, `POST`.** Lowercase `get`, `post` will not route.

## Security & Configuration

- Local Supabase: `npx supabase start`, copy credentials to `.env` + `.dev.vars`.
- Deploy: `npx wrangler deploy`, set secrets via `npx wrangler secret put`.
- Email confirmation: disable in Supabase dashboard during local dev.

## Project Structure

- `src/pages/` — Astro pages (file-based routing) + `api/` endpoints
- `src/components/` — Astro/React UI; `ui/` holds shadcn components
- `src/lib/` — services, helpers, utilities
- `src/middleware.ts` — auth guard + user resolution

Path alias: `@/*` → `./src/*`

## Build, Test, and Development Commands

- `npm run dev` — start dev server (Cloudflare workerd runtime)
- `npm run build` — production build (SSR via @astrojs/cloudflare)
- `npm run preview` — preview production build
- `npm run lint` — ESLint with strict type-checked rules
- `npm run lint:fix` — auto-fix lint issues
- `npm run format` — Prettier (includes Astro + Tailwind plugins)

Pre-commit: husky + lint-staged auto-runs `eslint --fix` on `*.{ts,tsx,astro}` and `prettier --write` on `*.{json,css,md}`.

## Coding Style & Naming Conventions

TypeScript strict (`@tsconfig.json`), ESLint strict type-checked (`@eslint.config.js`), Prettier with 2-space indent, 120-char width (`@.prettierrc.json`).

- Components: PascalCase `.astro` / `.tsx`. Astro for static, React for interactivity only.
- Services/helpers: camelCase in `src/lib/`.
- Types: shared in `src/types.ts`.
- Migrations: `supabase/migrations/YYYYMMDDHHmmss_description.sql`.
- shadcn: install via `npx shadcn@latest add [name]`, never hand-author in `ui/`.

## Testing Guidelines

⚠️ **No test runner configured yet.** Set up Vitest before making algorithmic changes (color quantization, DMC mapping, pattern size calculations). See @context/foundation/health-check.md for setup instructions.

## Commit & Pull Request Guidelines

Pre-commit hooks enforce lint + format. CI (`.github/workflows/ci.yml`) runs `npm run lint` and `npm run build` on every push and PR to master. Build requires `SUPABASE_URL` and `SUPABASE_KEY` repository secrets.

Commit messages: no enforced convention yet (repo not yet initialized as git). After `git init`, follow conventional commits (`feat:`, `fix:`, `docs:`, etc.).

## Architecture Overview

Full SSR (`output: "server"`). All pages server-rendered; API routes need `export const prerender = false`. Auth in `src/middleware.ts` resolves user, attaches to `context.locals.user`, redirects unauthenticated from `PROTECTED_ROUTES`.

See @context/foundation/prd.md and @context/foundation/tech-stack.md for deeper context.
