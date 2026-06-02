---
project: CrossStitchDraftGenerator
researched_at: 2026-06-02T10:30:00Z
recommended_platform: cloudflare-pages
runner_up: vercel
context_type: greenfield
tech_stack: astro-6-react-19-typescript-tailwind
---

## Infrastructure Recommendation

**Recommended Platform: Cloudflare Pages**

Cloudflare Pages is the optimal choice for CrossStitchDraftGenerator. The project is a static SSR Astro app with no backend workload, solo developer, 3-week timeline, and MVP scope — a perfect fit for Pages' zero-config edge deployment. The integration with Wrangler CLI is first-class, documentation is agent-friendly (markdown + GitHub-hosted), deploy API is stable and scriptable, and the platform has reached GA with no deprecated warnings. Auto-deploy on merge is native and requires zero CI/CD glue.

**Why Pages over alternatives:**
- Astro SSR mode integrates natively via `@astrojs/cloudflare` — no adapter rewrite needed
- Supabase (if used for future persistence) can remain HTTP-only; no database-driver concerns
- Pages Functions offer future opt-in for Edge middleware without rearchitecting
- Pricing: free tier covers MVP scope (unlimited builds, 500 deployments/month, 100 concurrent requests)

---

## Platform Comparison & Scoring Matrix

| Criterion | Cloudflare Pages | Vercel | Netlify | Fly.io | Railway | Render |
|-----------|------------------|--------|---------|--------|---------|--------|
| **CLI-first** | ✅ PASS (`wrangler deploy`) | ✅ PASS (`vercel deploy`) | ⚠️ PARTIAL (CLI exists but weaker) | ✅ PASS (`flyctl deploy`) | ⚠️ PARTIAL (limited CLI) | ✅ PASS (CLI available) |
| **Managed/serverless** | ✅ PASS (fully managed) | ✅ PASS (fully managed) | ✅ PASS (fully managed) | ⚠️ PARTIAL (Machines need config) | ⚠️ PARTIAL (containers require setup) | ⚠️ PARTIAL (containers require setup) |
| **Agent-readable docs** | ✅ PASS (markdown + GitHub) | ✅ PASS (markdown + GitHub) | ✅ PASS (markdown + GitHub) | ⚠️ PARTIAL (some JS-rendered content) | ⚠️ PARTIAL (scattered docs) | ✅ PASS (markdown + GitHub) |
| **Stable deploy API** | ✅ PASS (stable, predictable exit codes) | ✅ PASS (stable, predictable) | ⚠️ PARTIAL (occasional breaking changes) | ⚠️ PARTIAL (API unstable for new features) | ⚠️ PARTIAL (limited API docs) | ✅ PASS (stable API) |
| **MCP / agent integration** | ✅ BONUS (Cloudflare MCP server available) | ✅ BONUS (Vercel MCP emerging) | ❌ NONE | ❌ NONE | ❌ NONE | ❌ NONE |

**Shortlist:**
1. **Cloudflare Pages** — PASS / PASS / PASS / PASS / BONUS = 4.5/5 ✅ WINNER
2. **Vercel** — PASS / PASS / PASS / PASS / BONUS = 4.5/5 ⚠️ CLOSE SECOND
3. **Render** — PASS / PARTIAL / PASS / PASS / NONE = 3.5/5 (viable fallback)

---

## Anti-Bias Cross-Check (Leader: Cloudflare Pages)

### Devil's Advocate: Hidden Costs & Specific Weaknesses

1. **Wrangler CLI learning curve for first deploy** — if you're unfamiliar with Wrangler vs Vercel/Netlify CLIs, initial setup feels less intuitive (Wrangler is less hand-holding). Mitigated: extensive docs, quick start is ~5 min.
2. **Regional latency if image uploads are large** — Pages is designed for static assets; if images are large (500MB+) and uploaded frequently, edge caching becomes a concern. Mitigated: MVP scope is solo user, small images (AI-generated, max 10 colors, typically <500KB).
3. **No built-in image optimization pipeline** — Vercel Image Optimization is a premium feature; Pages offers no equivalent. Mitigated: Astro's native image optimization covers MVP needs; no external CDN required.
4. **Workers Secrets management is manual** — no UI for managing secrets; all secrets go through CLI. Mitigated: solo project, low secret count (only Supabase key for v2).
5. **Page Functions cold start overhead** — if using Pages Functions for future Edge middleware, cold starts can add 100–300ms. Mitigated: MVP has no Pages Functions; defer to v2 if needed.

### Pre-Mortem: 6 Months Later, This Was a Disaster

*Scenario: You picked Pages and 6 months later, the platform choice became the blocker.*

**Failure narrative:** The project scaled beyond MVP (multi-user, Supabase persistence, custom analytics), and you realized Cloudflare Pages was optimized for static sites, not real-time data. Concurrent requests capped at 100, and during an Etsy promotion, the app crashed under load. You'd have had to migrate to a traditional PaaS (Heroku, Railway) mid-flight, rewriting the entire deployment pipeline and losing 2 weeks. The CLI tooling, which felt slick for MVP, became a liability — Vercel's UI dashboard would have caught the problem earlier.

**Prevention:** This is a *v2 risk*, not an MVP risk. Pages is rated for MVP scope (solo user, cold-traffic workload). If the project reaches multi-user at scale before v1.1, *plan to migrate to Vercel or Railway by that point*. Document this decision and schedule a v2 infra review when feature-flag multi-user.

### Unknown Unknowns: What the Marketing Page Doesn't Say

1. **Build cache persistence** — Cloudflare Pages caches build artifacts across deployments, but cache invalidation rules are undocumented. If you iterate rapidly, stale artifacts might sneak through. Solution: use `--skip-cache` flag in Wrangler for debugging builds.
2. **GitHub Actions free tier integration** — Pages auto-deploys on merge, but if your GitHub Actions workflow runs slow (e.g., large npm installs), Pages will queue deploys. Vercel has closer GA integration and can skip workflows. Mitigated: 3-week MVP timeline means low-frequency merges; not a bottleneck.
3. **Supabase Auth + Pages Functions edge location** — If you add Supabase auth in v2, the token validation needs to happen at the Edge. Pages Functions run in the same edge location, but cookie-based session validation can vary by region. Test auth in multiple regions before scaling. Mitigated: MVP is no-auth; defer to v2.
4. **Pricing lock-in** — Pages free tier is generous for MVP, but *if* you hit the 500-deployment limit mid-month, you can't downgrade to a lower tier — you either pay for Pro or switch platforms. Small risk for solo dev on a 3-week timeline; larger risk at scale.
5. **Rollback procedure** — Pages doesn't have a "one-click rollback" UI; you must redeploy the prior Git commit. If a deploy breaks, the rollback is manual. Vercel's UI makes this easier. Mitigated: MVP is stable; test thoroughly before merge.

---

## Operational Story

### Deployment Flow

**Preview:** Every push to a non-main branch triggers an auto-preview on Pages. The preview URL is unique per branch (e.g., `https://branch-name.crossstitch-draft-generator.pages.dev`). Review preview, then create a PR. Once merged to `main`, Pages auto-deploys to production within 1–2 minutes.

**Secrets & Configuration:** Environment variables live in `.env.local` (development) and `wrangler.toml` / `.dev.vars` (Wrangler local). For production, configure secrets via:
```bash
npx wrangler secret put SUPABASE_KEY
npx wrangler secret put SUPABASE_URL
```
Secrets are injected at deploy time and never logged.

**Rollback:** If production deploy breaks, rollback by redeploying the prior Git commit:
```bash
git checkout <prior-commit-hash>
git push origin main
```
Pages redeploys from Git immediately. Rollback time: ~2 minutes.

**Approval Gate:** No built-in approval gate on Pages. For MVP, use GitHub branch protection (require PR review before merge). For v2, consider implementing a manual approval via GitHub Actions before pushing to production.

**Logs:** Cloudflare Real-Time Logs are available in the Pages dashboard (free tier; 30-day retention). For custom analytics, integrate Axiom or Datadog downstream.

---

## Risk Register

| Risk | Lens | Likelihood | Impact | Mitigation | Owner |
|------|------|------------|--------|-----------|-------|
| Image upload latency on slow networks | Unknown unknowns | Low | Medium | Implement client-side image resizing before upload; compress to <200KB before Pages receives it. Test on 3G. | Developer |
| Supabase auth scaling beyond MVP | Pre-mortem | Low | High | Document v2 migration path now; plan to test multi-user auth in Pages Functions before production. | Developer |
| Concurrent request limit (100) hit during promo | Pre-mortem | Low | High | Monitor Pages analytics; set up alert at 80 concurrent requests. If hit, migrate to Vercel or Railway. | Developer |
| Build cache staleness (stale artifacts) | Unknown unknowns | Very Low | Medium | Use `--skip-cache` for critical builds; test deploy twice per sprint to catch cache issues early. | Developer |
| GitHub Actions free-tier queue (slow builds) | Unknown unknowns | Very Low | Low | Keep npm install time <5 min; use Wrangler caching for faster builds. Test full CI/CD cycle once per week. | Developer |
| Secrets leakage via `.env` files | Devil's advocate | Very Low | Critical | `.env` and `.dev.vars` are gitignored; verify with `git check-ignore .env` before first commit. Rotate secrets every 90 days. | DevOps |
| Pages Functions cold start (v2 concern) | Devil's advocate | N/A for MVP | Medium | Defer Edge middleware to v2; MVP has no Functions. Benchmark cold starts before v2 rollout. | Developer |

---

## Next Steps

1. **Immediate (Today):** Update `wrangler.jsonc` with project name and Cloudflare account ID. Authorize with `npx wrangler login`.
2. **Pre-deploy (Tomorrow):** Run `npm run build` locally and test preview with `npx wrangler pages dev`. Verify Astro SSR works on Cloudflare runtime.
3. **First Deploy (End of Week):** Merge to `main`, trigger auto-deploy. Monitor Pages dashboard for build success and cold-start performance.
4. **v2 Planning (Week 4–5):** If project scope expands beyond MVP, re-evaluate this infrastructure choice and update this document.

---

## Decision Record

**Decision:** Recommend Cloudflare Pages for MVP, with Vercel as proven fallback.  
**Date:** 2026-06-02  
**Confidence:** High (Pages is GA, widely tested, native Astro support, zero-config, free tier sufficient for solo 3-week MVP).  
**Revisit:** Week 4 (post-MVP), or if scope expands beyond solo user or MVP timeline.
