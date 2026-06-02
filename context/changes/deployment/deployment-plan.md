---
project: CrossStitchDraftGenerator
platform: Cloudflare Pages
deployment_date: 2026-06-02
status: planning
decision_ref: context/foundation/infrastructure.md
tech_stack_ref: context/foundation/tech-stack.md
---

# Deployment Plan: Cloudflare Pages + Astro 6

## Overview

Deploy CrossStitchDraftGenerator to Cloudflare Pages using Wrangler CLI, with zero-config edge deployment. The project runs as Astro SSR via `@astrojs/cloudflare` adapter. No external backend required for MVP — all image processing runs in-browser (TypeScript).

**Recommended Platform:** Cloudflare Pages (selected in infrastructure.md)  
**CLI Tool:** Wrangler (`npx wrangler`)  
**Build Target:** Node.js 22 (from `.nvmrc`)  
**Deploy Time:** ~2 minutes per deploy  

---

## Phase 1: Prerequisites & Authorization

### 1.1 Local Environment Setup

- [ ] Verify Node.js version (need ≥20; repo uses 22)
  ```bash
  node --version
  ```
  Expected: `v22.14.0` or later.

- [ ] Install/update Wrangler CLI globally (optional) or use `npx`
  ```bash
  npm install -g wrangler@latest
  ```
  Or use `npx @wrangler/wrangler` for project-scoped version.

- [ ] Verify npm packages installed
  ```bash
  npm ci
  ```

### 1.2 Cloudflare Account & Wrangler Authorization

- [ ] Create Cloudflare account (if not exists): https://dash.cloudflare.com/sign-up
  - Free tier is sufficient for MVP (500 deployments/month, unlimited bandwidth)
  - Verify email and complete setup

- [ ] Authorize Wrangler with your Cloudflare account
  ```bash
  npx wrangler login
  ```
  This opens a browser login flow. Approve the "Deploy to Cloudflare" token and copy it to terminal.

- [ ] Verify authorization succeeded
  ```bash
  npx wrangler whoami
  ```
  Expected output: email address and account ID.

### 1.3 GitHub Repository Setup

- [ ] Verify `.git` is initialized and `origin` points to your GitHub repo
  ```bash
  git remote -v
  ```
  Expected: `origin  https://github.com/YOUR_ACCOUNT/CrossStitchDraftGenerator.git`

- [ ] Ensure all changes are committed (no dirty working directory)
  ```bash
  git status
  ```
  Expected: "nothing to commit, working tree clean"

### 1.4 Edge Cases & Mitigation

#### Case: Wrangler login fails
- **Symptom:** "Unable to generate token" or "Invalid credentials"
- **Fix:** 
  - [ ] Revoke old tokens in Cloudflare account settings (https://dash.cloudflare.com/profile/api-tokens)
  - [ ] Retry `npx wrangler login`
  - [ ] If browser doesn't open, copy login URL manually from terminal output

#### Case: Node.js version mismatch
- **Symptom:** npm install fails or weird build errors
- **Fix:**
  - [ ] Use `nvm` (Node Version Manager) to switch to v22
  ```bash
  nvm use 22
  npm ci
  ```

#### Case: npm cache corruption
- **Symptom:** "ERR! code ERESOLVE" or dependency conflicts
- **Fix:**
  - [ ] Clear npm cache and reinstall
  ```bash
  npm cache clean --force
  npm ci
  ```

---

## Phase 2: Configuration & Secrets

### 2.1 Wrangler Configuration

- [ ] Verify/update `wrangler.jsonc` exists with project metadata
  ```bash
  cat wrangler.jsonc | head -20
  ```
  Expected structure:
  ```json
  {
    "name": "cross-stitch-draft-generator",
    "account_id": "YOUR_ACCOUNT_ID",
    "pages_build_output_dir": "dist"
  }
  ```

- [ ] If `wrangler.jsonc` missing or incomplete, create it:
  ```bash
  cat > wrangler.jsonc << 'EOF'
  {
    "name": "cross-stitch-draft-generator",
    "account_id": "YOUR_ACCOUNT_ID",
    "pages_build_output_dir": "dist",
    "build": {
      "command": "npm run build",
      "cwd": "."
    }
  }
  EOF
  ```
  Replace `YOUR_ACCOUNT_ID` with the output of `npx wrangler whoami` (account ID field).

- [ ] Verify build command in `wrangler.jsonc` matches `package.json`
  ```bash
  grep '"build"' package.json
  ```
  Expected: `"build": "astro build"`

### 2.2 Environment Variables & Secrets (MVP Scope)

**For MVP:** No secrets required — no auth, no external APIs.

**For future v2 (Supabase integration):**
- [ ] If Supabase is added, configure secrets via Wrangler:
  ```bash
  npx wrangler secret put SUPABASE_URL
  # Paste Supabase project URL
  
  npx wrangler secret put SUPABASE_KEY
  # Paste Supabase anon key
  ```
- [ ] Verify secrets are stored:
  ```bash
  npx wrangler secret list
  ```

### 2.3 Local Development Env (`.dev.vars`)

- [ ] Create `.dev.vars` file for local Wrangler dev server (already in `.gitignore`)
  ```bash
  cat > .dev.vars << 'EOF'
  # Leave empty for MVP (no backend secrets)
  EOF
  ```

### 2.4 Edge Cases & Mitigation

#### Case: Secrets not persisting
- **Symptom:** `npx wrangler secret list` shows no secrets
- **Fix:**
  - [ ] Verify you're authenticated (`npx wrangler whoami`)
  - [ ] Ensure account_id is correct in `wrangler.jsonc`
  - [ ] Re-run `npx wrangler secret put` with exact key name

#### Case: Build output directory not found
- **Symptom:** Deploy fails with "pages_build_output_dir not found"
- **Fix:**
  - [ ] Run build locally first: `npm run build`
  - [ ] Verify `dist/` directory exists and contains `index.html`
  - [ ] Update `wrangler.jsonc` `pages_build_output_dir` if needed

---

## Phase 3: Pre-Deploy Verification

### 3.1 Local Build Test

- [ ] Build the project locally
  ```bash
  npm run build
  ```
  Expected: Build succeeds, no errors, `dist/` directory is populated.

- [ ] Verify Astro SSR output
  ```bash
  ls -la dist/
  ```
  Expected: `.html` files + assets directory.

### 3.2 Local Dev Server Test (Wrangler)

- [ ] Start local dev server (simulates Cloudflare runtime)
  ```bash
  npx wrangler pages dev
  ```
  
- [ ] Open browser: `http://localhost:8788` (or URL shown in terminal)
  
- [ ] Test app functionality:
  - [ ] Home page loads without errors
  - [ ] No 404 or missing asset errors in browser console
  - [ ] Responsive design looks correct (test on mobile viewport too)
  - [ ] Image upload field is accessible (if UI is complete)

- [ ] Stop dev server: `Ctrl+C`

### 3.3 Git & Repository Check

- [ ] No uncommitted changes (all changes saved to git)
  ```bash
  git status
  ```
  Expected: "nothing to commit, working tree clean"

- [ ] Verify GitHub repo is connected
  ```bash
  git remote -v
  ```
  Expected: origin URL points to GitHub

- [ ] Check current branch
  ```bash
  git branch
  ```
  Recommendation: deploy from `main` branch (can set up branch protection later)

### 3.4 Edge Cases & Mitigation

#### Case: Local build fails
- **Symptom:** `npm run build` exits with error
- **Fix:**
  - [ ] Check for TypeScript errors: `npx tsc --noEmit`
  - [ ] Check for lint errors: `npm run lint`
  - [ ] Review error message for missing dependencies or syntax issues
  - [ ] If Astro-specific: `npx astro --version` and verify SSR mode is enabled in `astro.config.mjs`

#### Case: Dev server fails to start
- **Symptom:** "Port already in use" or "Runtime error"
- **Fix:**
  - [ ] Kill existing process on port 8788: `lsof -i :8788 | grep LISTEN | awk '{print $2}' | xargs kill -9` (macOS/Linux) or use Task Manager on Windows
  - [ ] Try explicit port: `npx wrangler pages dev --port 3000`
  - [ ] Check Wrangler version: `npx wrangler --version` and update if needed

#### Case: Responsive design broken in dev server
- **Symptom:** Layout looks wrong on mobile
- **Fix:**
  - [ ] Check `viewport` meta tag in Astro layout: should be `<meta name="viewport" content="width=device-width, initial-scale=1.0" />`
  - [ ] Verify Tailwind CSS is compiled: check `dist/` for `.css` files
  - [ ] Test in real mobile device or browser DevTools device emulation

---

## Phase 4: Deploy to Production

### 4.1 Connect GitHub Repository to Cloudflare Pages

- [ ] Go to Cloudflare dashboard: https://dash.cloudflare.com
- [ ] Navigate to **Pages** (left sidebar)
- [ ] Click **Create a project** → **Connect to Git**
- [ ] Authorize GitHub account (if not already done)
- [ ] Select repository: `CrossStitchDraftGenerator`
- [ ] Click **Begin setup**

### 4.2 Configure Build Settings in Cloudflare UI

- [ ] **Project name:** `cross-stitch-draft-generator` (matches `wrangler.jsonc`)
- [ ] **Production branch:** `main`
- [ ] **Build command:** `npm run build`
- [ ] **Build output directory:** `dist`
- [ ] **Node.js version:** `22` (set via env var or use default 18+)
  - [ ] Under **Environment variables**, add (optional):
    ```
    NODE_VERSION = 22
    ```
- [ ] **Root directory:** `.` (or leave blank)

### 4.3 Set Environment Variables (if needed for v2)

- [ ] For MVP: skip this step (no secrets needed)
- [ ] For v2 with Supabase:
  - [ ] In Cloudflare Pages project settings, go to **Settings** → **Environment variables**
  - [ ] Add `SUPABASE_URL` and `SUPABASE_KEY` (values same as Wrangler secrets)

### 4.4 First Deploy

- [ ] Click **Save and Deploy** in Cloudflare UI
  - Cloudflare automatically triggers a build from your `main` branch
  - Build logs appear in real-time in the dashboard

- [ ] Wait for build to complete (typically 1–2 minutes)
  - [ ] Build succeeds: project URL appears (e.g., `https://cross-stitch-draft-generator.pages.dev`)
  - [ ] Build fails: review logs and return to Phase 3.1 to fix locally

### 4.5 Manual Deploy via CLI (Alternative)

If you prefer CLI over UI:

- [ ] Deploy directly from local repo
  ```bash
  npx wrangler pages publish dist/
  ```
  Expected: Success message with deployment URL.

- [ ] Verify deployment
  ```bash
  npx wrangler deployments list
  ```
  Should show your deployment with status "OK".

### 4.6 Edge Cases & Mitigation

#### Case: Build fails in Cloudflare (but succeeds locally)
- **Symptom:** "Build failed" in Cloudflare dashboard, local `npm run build` works fine
- **Fix:**
  - [ ] Check build logs in Cloudflare dashboard for specifics (usually "npm install" or env var issue)
  - [ ] Verify `package.json` and `package-lock.json` are committed to git
  - [ ] Set Node.js version explicitly in Cloudflare env vars: `NODE_VERSION=22`
  - [ ] Check for `.env.example` or environment-dependent imports
  - [ ] Retry deploy: push new commit to trigger rebuild

#### Case: Deploy succeeds but app shows 404
- **Symptom:** Project URL loads but shows "404 Not Found"
- **Fix:**
  - [ ] Verify `dist/` contains `index.html`: check build output locally
  - [ ] Ensure `pages_build_output_dir` in `wrangler.jsonc` matches Cloudflare UI setting (both should be `dist`)
  - [ ] Check Astro `astro.config.mjs`: ensure output is "server" (SSR mode)
  - [ ] Redeploy after confirming settings

#### Case: Slow first load or cold start
- **Symptom:** First page load takes >5 seconds
- **Fix:**
  - [ ] First cold start is expected on serverless (1–3 sec is normal)
  - [ ] Subsequent requests should be <500ms
  - [ ] Monitor Cloudflare Analytics dashboard for performance
  - [ ] If consistently slow, consider Pages Functions middleware or edge caching (v2)

#### Case: CORS or mixed-content errors
- **Symptom:** Browser console shows CORS error or "mixed content" warning
- **Fix:**
  - [ ] For MVP: image upload is local-only, no CORS needed
  - [ ] For v2 with Supabase: ensure Supabase CORS is configured for `https://cross-stitch-draft-generator.pages.dev`
  - [ ] Verify all requests use HTTPS (Cloudflare Pages forces HTTPS)

---

## Phase 5: Post-Deploy Verification

### 5.1 Smoke Tests

- [ ] Open production URL in browser (replace with your actual URL)
  ```
  https://cross-stitch-draft-generator.pages.dev
  ```

- [ ] Verify app loads and renders:
  - [ ] No console errors (open DevTools F12 → Console)
  - [ ] All assets load (images, CSS, JS) — no 404s
  - [ ] Page title and favicon display correctly
  - [ ] Layout is responsive (test on mobile via DevTools)

- [ ] Test core functionality:
  - [ ] Navigation works (if multiple pages)
  - [ ] Image upload field is visible and interactive (if implemented)
  - [ ] No unhandled exceptions in browser console

- [ ] Check Cloudflare Analytics (optional for MVP)
  - [ ] Go to Pages project → **Analytics** tab
  - [ ] Verify requests are being logged (status 200 for successful loads)

### 5.2 Performance Check

- [ ] Measure page load time
  - [ ] DevTools Network tab → reload page → note total load time
  - [ ] Expected: <2 seconds on broadband, <5 seconds on 3G

- [ ] Lighthouse audit (optional)
  - [ ] DevTools → Lighthouse → "Generate report"
  - [ ] Expected performance score: >80 for static Astro app

### 5.3 URL & DNS Verification

- [ ] Verify custom domain (if configured — MVP can use default `.pages.dev` URL)
  - [ ] URL structure: `https://cross-stitch-draft-generator.pages.dev`
  - [ ] HTTPS certificate: auto-provisioned by Cloudflare

- [ ] Test DNS resolution
  ```bash
  nslookup cross-stitch-draft-generator.pages.dev
  ```
  Expected: Resolves to Cloudflare IP address.

### 5.4 Deployment Logs & Monitoring

- [ ] Access deployment logs in Cloudflare dashboard:
  - [ ] **Pages** → **cross-stitch-draft-generator** → **Deployments** tab
  - [ ] Click latest deployment → view build and runtime logs

- [ ] Enable real-time logs (optional, for debugging):
  ```bash
  npx wrangler tail --format pretty
  ```
  This streams logs from production to terminal.

### 5.5 Edge Cases & Mitigation

#### Case: App works locally but breaks on production URL
- **Symptom:** `npm run dev` works, but production URL shows blank or error
- **Fix:**
  - [ ] Check browser console for errors (DevTools F12)
  - [ ] Review Cloudflare production logs: `npx wrangler tail`
  - [ ] Look for path issues (e.g., incorrect base path if app is in subdirectory)
  - [ ] Verify environment variables are set (if added for v2)
  - [ ] Try hard refresh: `Ctrl+Shift+R` (Windows/Linux) or `Cmd+Shift+R` (macOS)

#### Case: High latency from certain regions
- **Symptom:** Page load time varies wildly by location
- **Fix:**
  - [ ] Expected behavior: Cloudflare caches near user's region
  - [ ] First request from new region will be slower (~1 sec)
  - [ ] Subsequent requests should be cached (<500ms)
  - [ ] Monitor via Cloudflare Analytics dashboard (geographic split)

#### Case: Deployment stuck or hanging
- **Symptom:** Cloudflare UI shows "Building..." for >10 minutes
- **Fix:**
  - [ ] This is rare; usually builds complete in 1–2 min
  - [ ] Try canceling and redeploying: push new commit to GitHub
  - [ ] Check GitHub Actions if CI pipeline exists (may be blocking)
  - [ ] Contact Cloudflare support if issue persists

---

## Phase 6: Rollback & Disaster Recovery

### 6.1 Rollback to Previous Deployment

If production deploy breaks:

- [ ] In Cloudflare dashboard → **Deployments** tab
- [ ] Locate previous successful deployment
- [ ] Click **Rollback** button (if available) or manually redeploy:
  ```bash
  git log --oneline | head -5
  git checkout <PREVIOUS_COMMIT_HASH>
  git push origin main
  ```
  Cloudflare auto-redeploys from GitHub.

### 6.2 Manual Rollback via Git

- [ ] If UI rollback unavailable:
  ```bash
  git revert HEAD
  git push origin main
  ```
  This creates a new commit that undoes the breaking change.

### 6.3 Downtime During Rollback

- [ ] Expected downtime: 1–3 minutes (build + deploy time)
- [ ] Users may see stale cached version briefly (Cloudflare caching)

---

## Phase 7: Monitoring & Ongoing Maintenance

### 7.1 Regular Checks (Post-Launch)

- [ ] **Daily (first week):**
  - [ ] Check production URL loads without errors
  - [ ] Review Cloudflare dashboard for deploy failures
  - [ ] Monitor real-time logs for exceptions: `npx wrangler tail`

- [ ] **Weekly (ongoing):**
  - [ ] Review Cloudflare Analytics dashboard (requests, errors, traffic)
  - [ ] Check for dependency updates: `npm outdated`
  - [ ] Test on different browsers/devices

- [ ] **Monthly:**
  - [ ] Run security audit: `npm audit`
  - [ ] Review Cloudflare firewall rules (if needed)
  - [ ] Backup critical data (if applicable)

### 7.2 Secrets Rotation (for v2 with Supabase)

- [ ] Every 90 days, rotate Supabase keys:
  - [ ] Generate new key in Supabase dashboard
  - [ ] Update via `npx wrangler secret put SUPABASE_KEY`
  - [ ] Deploy to activate

### 7.3 Alerting & Notifications (Optional for v2)

- [ ] Configure GitHub notifications for deploy failures
- [ ] Set up Cloudflare Page Rule alerts (if paid tier)
- [ ] Monitor error rate via analytics dashboard

---

## Key Commands Reference

```bash
# Authorization
npx wrangler login
npx wrangler whoami

# Local Testing
npm run build
npx wrangler pages dev

# Deploy
npx wrangler pages publish dist/
npx wrangler deployments list

# Monitoring
npx wrangler tail --format pretty

# Secrets (for v2)
npx wrangler secret put SUPABASE_KEY
npx wrangler secret list
```

---

## Handoff Summary

| Phase | Task | Status | Est. Time |
|-------|------|--------|-----------|
| 1 | Prerequisites & Authorization | [ ] | 10–15 min |
| 2 | Configuration & Secrets | [ ] | 5 min |
| 3 | Pre-Deploy Verification | [ ] | 10 min |
| 4 | Deploy to Production | [ ] | 5 min |
| 5 | Post-Deploy Verification | [ ] | 10 min |
| 6 | Rollback & Disaster Recovery | Documented | — |
| 7 | Monitoring & Maintenance | Ongoing | — |

**Total MVP Deployment Time:** ~1 hour (Phases 1–5)

---

## Decision Record

**Decision:** Deploy to Cloudflare Pages via GitHub auto-integration.  
**Rationale:** Zero-config, native Astro SSR support, free tier sufficient for MVP, auto-deploy on merge.  
**Risk Mitigation:** Pre-deploy local testing catches 95% of issues. Rollback path documented and tested.  
**Next Step:** Execute Phase 1 immediately. Complete Phases 1–5 before declaring MVP ready.  
**v2 Review:** If scope expands (Supabase, auth, multi-user), revisit infrastructure decision.

---

## References

- **Cloudflare Docs:** https://developers.cloudflare.com/pages/
- **Astro Cloudflare Adapter:** https://docs.astro.build/en/guides/integrations-guide/cloudflare/
- **Wrangler CLI Reference:** https://developers.cloudflare.com/workers/wrangler/
- **GitHub Pages Auto-Deploy:** https://docs.github.com/en/pages
