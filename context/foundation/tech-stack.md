---
starter_id: 10x-astro-starter
package_manager: npm
project_name: cross-stitch-draft-generator
hints:
  language_family: js
  team_size: solo
  deployment_target: cloudflare-pages
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: first-class
  path_taken: standard
  quality_override: false
  self_check_answers: null
  has_auth: false
  has_payments: false
  has_realtime: false
  has_ai: false
  has_background_jobs: false
---

## Why this stack

CrossStitchDraftGenerator is a solo, 3-week after-hours web app: image upload, in-browser color quantization, DMC palette mapping, physical size calculation, and local project persistence — no server-side compute required for the MVP. `10x-astro-starter` (Astro 6 + React 19 + TypeScript + Tailwind CSS 4 + Supabase + Cloudflare Pages) matches every dimension: TypeScript-first for type-safe image-processing logic, convention-based routing and component model, Cloudflare Pages for zero-config edge deploy, and Supabase available if project persistence outgrows browser storage in v2. The starter is agent-friendly out of the box — no AGENTS.md patching needed. Image processing (quantization, DMC mapping, pattern size) runs entirely in the browser as pure TypeScript; the starter's edge runtime adds no friction for MVP scope.
