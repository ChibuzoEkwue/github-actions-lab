# GitHub Actions Mastery Checklist (50 Projects)



## Next.js Project Setup (use this as your base repo for most exercises)

```bash
# 1. Create a new Next.js app (TypeScript)
npx create-next-app@latest my-actions-lab --typescript --eslint --app --src-dir --import-alias "@/*"
cd my-actions-lab

# 2. Run it locally
npm run dev
# → http://localhost:3000

# 3. Init git + push to GitHub (needed before any workflow will run)
git init
git add .
git commit -m "chore: init next.js app for github actions lab"
gh repo create my-actions-lab --public --source=. --remote=origin
git push -u origin main
```

**Folder to create for all workflows:**
```
.github/
  workflows/
    ci.yml
  actions/            # for custom composite/JS actions later
```

**Common scripts to have in `package.json` (workflows will call these):**
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

**Quick sanity check before writing any workflow:**
```bash
npm run build   # must pass locally first
npm run lint    # must pass locally first
```
If these fail locally, your CI will fail too — always validate locally before pushing.

---

## Tier 1 — Foundations

- [x] **1. Hello World Workflow**
  A workflow triggered on `push` that echoes a message to the log.
  **MVP:** workflow file in `.github/workflows/`; triggers only on push to a specific branch; log clearly shows your custom message; visible green check in Actions tab.

- [ ] **2. Multi-Trigger Workflow**
  One workflow that responds to `push`, `pull_request`, and manual `workflow_dispatch`.
  **MVP:** all three triggers work independently; `workflow_dispatch` accepts at least one custom `input` and prints it in the log; Actions UI shows the correct "triggered by" event for each run.

- [ ] **3. Node.js CI Pipeline**
  Installs dependencies and runs the test suite on every push.
  **MVP:** uses `actions/setup-node`; runs `npm install` + `npm test`; caching configured with `actions/cache` (or `cache: 'npm'` in setup-node); a broken test fails the run.

- [ ] **4. Linting Workflow**
  Runs ESLint/Prettier checks on every PR.
  **MVP:** runs on `pull_request`; a deliberate lint violation fails the run; fixing it passes on the same PR; no lint errors are silently swallowed.

- [ ] **5. Matrix Build Across Node Versions**
  Tests the app against multiple Node versions in parallel.
  **MVP:** `strategy.matrix` with at least 3 Node versions; jobs run in parallel (visible as separate legs); one deliberately-broken version fails independently of the others.

- [ ] **6. Environment Variables & Secrets**
  Uses a stored secret safely inside a workflow.
  **MVP:** secret added via repo Settings, never hardcoded; referenced with `secrets.NAME`; confirmed masked (`***`) in logs; demonstrates scope override (step-level env beats job-level).

- [ ] **7. Conditional Steps**
  Steps that only run under specific conditions.
  **MVP:** at least one step gated by branch (`github.ref`) and one gated by prior step outcome (`success()`/`failure()`); both conditions independently verified by triggering true/false cases.

- [ ] **8. Artifact Upload/Download**
  Builds an app in one job, passes the build output to a second job.
  **MVP:** `actions/upload-artifact` in job 1; `actions/download-artifact` in job 2; `needs:` correctly links them; removing `needs:` breaks the download (proven, not assumed).

- [ ] **9. PR Auto-Labeler**
  Labels PRs automatically based on changed file paths.
  **MVP:** `labeler.yml` config with at least 2 path-based rules; a PR touching multiple areas gets multiple correct labels with zero manual tagging.

- [ ] **10. Auto-Comment Bot**
  Comments a welcome message on a contributor's first PR.
  **MVP:** uses `actions/github-script` or `octokit`; comment posts as `github-actions[bot]`; logic prevents duplicate comments on that same user's later PRs.

- [ ] **11. Scheduled Workflow (Cron)**
  Runs daily to check something (e.g. outdated deps).
  **MVP:** valid `on.schedule` cron expression; also supports `workflow_dispatch` as a manual fallback for testing; at least one real scheduled run observed in Actions history (not just theoretical).

- [ ] **12. Manual Approval Gate**
  Pauses a workflow until a human approves.
  **MVP:** GitHub Environment with required reviewers configured; workflow visibly halts and shows "Review deployments"; job only proceeds after explicit approval, confirmed by timestamp gap in logs.

## Tier 2 — Real CI/CD

- [ ] **13. Next.js + MongoDB CI Pipeline**
  Runs integration tests against a real MongoDB instance spun up in CI.
  **MVP:** `services:` block running `mongo`; health check configured; tests that hit the DB fail if the service block is removed (proves it's actually required).

- [ ] **14. Publish an npm Package**
  Auto-publishes a package to npm on new version tags.
  **MVP:** triggered by git tag push; uses `NODE_AUTH_TOKEN` secret; version on npmjs.com updates; workflow fails if version wasn't bumped.

- [ ] **15. Deploy to Vercel via Actions**
  Triggers a Vercel deployment from a workflow, not Vercel's native integration.
  **MVP:** Vercel CLI called with a deployment token secret; deployment URL printed in logs and is live/reachable; a failing build step blocks deployment entirely.

- [ ] **16. Reusable Workflow**
  Shared CI logic called from multiple repos/workflows via `workflow_call`.
  **MVP:** one reusable workflow file with defined `inputs`/`secrets`; called from at least 2 different workflows or repos; a change to the shared file updates behavior in both callers.

- [ ] **17. Composite Action**
  Bundles setup + install + lint into one reusable custom action.
  **MVP:** `action.yml` in `.github/actions/your-action/`; referenced via `uses: ./.github/actions/your-action`; produces identical output to the original multi-step version; at least one configurable `input`.

- [ ] **18. Semantic Release Automation**
  Auto version-bump and changelog from Conventional Commits.
  **MVP:** `semantic-release` configured; `fix:` bumps patch, `feat:` bumps minor (both tested); `CHANGELOG.md` auto-committed; `chore:` commits don't trigger a release.

- [ ] **19. Auto Dependency Updates**
  Dependabot PRs auto-merge after CI passes, for patch-level bumps only.
  **MVP:** `dependabot.yml` configured; auto-merge workflow gated on CI status; a broken test blocks the auto-merge; a major-version bump is NOT auto-merged.

- [ ] **20. Slack/Discord Notification Pipeline**
  Sends build status to a channel on failure.
  **MVP:** incoming webhook secret configured; message includes branch, commit, and run link; only fires `if: failure()` (no spam on success, unless intentional).

- [ ] **21. Stripe Webhook Test Harness**
  Spins up the app and replays Stripe test webhook events in CI.
  **MVP:** app started as background process; workflow polls/health-checks before firing events (no fragile fixed `sleep`); confirms 200 response from your handler in logs.

- [ ] **22. Email Test Pipeline with Resend**
  Sends and verifies a test transactional email in CI.
  **MVP:** Resend test/sandbox API key stored as scoped secret; workflow fails on non-success API response; log clearly states what was sent and to whom (test address).

- [ ] **23. PR Preview Deployments**
  Every PR gets its own live preview URL, posted as a comment.
  **MVP:** unique URL per PR (no overwriting); bot comment updates in place on new commits (not duplicated); URL is confirmed reachable, not just printed.

- [ ] **24. Monorepo Path-Based Triggers**
  Only runs a package's pipeline when its files change.
  **MVP:** `paths:` filter or `dorny/paths-filter` configured per package; a change to package A does not trigger package B's job; verified with a real side-by-side test.

- [ ] **25. Turborepo/Nx Affected Builds**
  Builds/tests only affected packages in a monorepo.
  **MVP:** Turbo/Nx configured with remote or local caching; tool output explicitly shows cache hits/skips for unaffected packages; a shared-dependency change correctly marks downstream packages affected.

- [ ] **26. License & Dependency Compliance Check**
  Fails PRs introducing disallowed (e.g. GPL) licenses.
  **MVP:** `license-checker` (or similar) run as a CI step; a deliberately added GPL package fails the build with a clear named error; removing it passes.

- [ ] **27. Code Coverage Enforcement**
  Fails builds below a coverage threshold; uploads to Codecov.
  **MVP:** coverage threshold defined and enforced (e.g. 80%); dropping below it fails CI; Codecov PR comment or dashboard reflects the actual diff in coverage.

## Tier 3 — Advanced Engineering

- [ ] **28. Custom JavaScript Action**
  A standalone JS action using the Actions Toolkit.
  **MVP:** `action.yml` with `runs.using: node20`; reads at least one `input` via `core.getInput()`; sets at least one `output` via `core.setOutput()`; fails properly via `core.setFailed()` on error (test this).

- [ ] **29. Custom Action with Outputs Consumed Downstream**
  An action's output is consumed by a later job.
  **MVP:** job-level `outputs:` correctly maps a step output; a second job reads it via `needs.<job>.outputs.<name>`; output value visible in the run summary; deliberately breaking the output mapping demonstrates the failure mode.

- [ ] **30. Publish Action to Marketplace**
  Publish a custom action publicly with proper versioning.
  **MVP:** `action.yml` includes branding (icon/color); tagged with semver (`v1`, `v1.1`); listed on GitHub Marketplace; a repo pinned to `@v1` isn't broken by a later `v1.1` release.

- [ ] **31. Self-Hosted Runner Setup**
  A non-containerized VM registered as a self-hosted runner.
  **MVP:** runner shows "Idle"/"Active" in repo settings; a job targeted with `runs-on: self-hosted` actually executes on your machine (verify via unique hostname); you can explain the public-repo security risk in your own words.

- [ ] **32. Dynamic Matrix Generation from JSON**
  Matrix values generated at runtime instead of hardcoded.
  **MVP:** a JSON source file feeds `fromJson()` into `strategy.matrix`; changing the JSON changes matrix legs with zero workflow edits; malformed JSON fails clearly at the generation step.

- [ ] **33. OIDC Auth to AWS (No Long-Lived Keys)**
  Deploys to AWS using OIDC, no stored access keys.
  **MVP:** `aws-actions/configure-aws-credentials` with `role-to-assume`; zero AWS keys in repo secrets; deployment succeeds and shows in CloudTrail; IAM trust policy scoped to your specific repo/branch.

- [ ] **34. Multi-Target Deployment Matrix**
  Deploys the same app to 2–3 different non-Docker targets via matrix.
  **MVP:** matrix with `include`/`exclude`; each leg deploys to a distinct, verifiable live target; one target's failure doesn't block the others (unless configured otherwise, and you can toggle it).

- [ ] **35. Blue-Green Deployment Workflow**
  Deploy to staging, smoke test, then swap to production.
  **MVP:** staging deploy → smoke test → production swap, in that strict order via `needs:`; a failing smoke test blocks production; a separate rollback job actually reverts to the prior state (tested).

- [ ] **36. Canary Release Pipeline**
  Gradual traffic shift with monitoring between stages.
  **MVP:** at least 2 distinct traffic-percentage stages with a pause/check between them; a simulated error spike triggers an automatic halt or rollback; final stage requires an explicit passing check.

- [ ] **37. Security Scanning Suite (No Container Scanning)**
  Combines CodeQL, `npm audit`, and Gitleaks.
  **MVP:** all three tools running in one workflow; findings appear in the repo Security tab (SARIF upload for CodeQL); a planted dummy secret is caught by Gitleaks; `npm audit --audit-level=high` fails the build on a known vulnerable package.

## Tier 4 — Expert / Platform-Level

- [ ] **38. Supply Chain Attestation (SLSA)**
  Generates build provenance for release artifacts.
  **MVP:** `actions/attest-build-provenance` used on a real build artifact; attestation visible attached to the release; verified successfully via `gh attestation verify`.

- [ ] **39. Load Testing in CI**
  k6/Artillery run against a preview deploy with a latency budget.
  **MVP:** load test step targets a real deployed preview URL; p95 latency threshold enforced as a pass/fail gate; artificially slowing an endpoint proves the gate actually fails.

- [ ] **40. Cross-Repo Workflow Orchestration**
  Repo A's success triggers a workflow in Repo B.
  **MVP:** `repository_dispatch` used with a fine-grained PAT (not default `GITHUB_TOKEN` — prove the default token can't do this first); Repo B's run history shows the correct trigger source and payload data.

- [ ] **41. Full Release Pipeline to a VPS (via SSH, No Docker)**
  Build, test, and deploy to a plain Linux VPS over SSH.
  **MVP:** SSH private key stored only as a secret (confirmed masked); deployment gated behind passing tests; app restarts with minimal/no visible downtime (e.g. via PM2 reload).

- [ ] **42. Multi-Tenant SaaS Deploy Pipeline**
  Per-tenant env vars/migrations deployed safely via matrix.
  **MVP:** matrix of tenants, each using its own environment-scoped secret; one tenant's secret is unreachable from another tenant's job (verified); adding a tenant requires only config, not workflow logic changes.

- [ ] **43. Database Migration Safety Gate**
  Migrations tested against a staging snapshot before production approval.
  **MVP:** migration runs against an ephemeral/staging DB first; production migration blocked until staging step passes; a rollback job demonstrably reverts staging to its prior state.

- [ ] **44. Feature Flag Rollout Automation**
  Toggles a feature flag post-deploy and monitors before full rollout.
  **MVP:** flag state changed via API call in the workflow (e.g. Upstash Redis), verified by a manual read; rollout happens in defined percentage steps, not one jump; a simulated error spike halts/reverts the flag.

- [ ] **45. Cost-Aware CI (Concurrency Control)**
  Cancels superseded runs to save billed minutes.
  **MVP:** `concurrency.group` + `cancel-in-progress: true` scoped correctly to PR branches only; rapid pushes to the same PR show a cancelled prior run; `main` branch deploys are proven NOT affected by the same setting.

- [ ] **46. Full Test Pyramid Pipeline**
  Unit → integration → E2E (Playwright) → visual regression, orchestrated as a job graph.
  **MVP:** job graph in the Actions UI reflects intended order/parallelism; a Playwright failure produces a downloadable trace/screenshot artifact; test report hosted (e.g. GitHub Pages) and linked back to the PR.

- [ ] **47. Reusable Org-Wide Workflow Library**
  Centralized `.github` repo of reusable workflows for multiple repos.
  **MVP:** at least 2 separate repos call the same central reusable workflow; version pinning (`@v1`) proven to isolate a repo from a later breaking change (`@v2`); adopting the shared workflow in a new repo takes only a few lines.

- [ ] **48. Compliance & Audit Trail Pipeline**
  Every production deploy generates a signed audit record.
  **MVP:** audit record captures real triggering actor, event, and approvals; stored append-only (e.g. as a release note or immutable artifact); "who approved this and when" is answerable from the record alone.

- [ ] **49. Incident Response Automation**
  Production error webhook opens an issue and can trigger rollback.
  **MVP:** simulated webhook payload creates a correctly detailed GitHub issue; rollback workflow can be triggered from the same pipeline and actually reverts to last-good deploy; duplicate errors in a short window don't spam multiple issues.

- [ ] **50. Full Production-Grade Release Train**
  Semantic versioning + changelog + multi-env promotion + notifications + security scans + rollback, all orchestrated together.
  **MVP:** a single merge to `main` flows dev → staging → prod with zero manual file edits mid-pipeline; each gate (test, security, smoke test) independently proven to block promotion when broken; you can explain the full pipeline end-to-end in under 5 minutes using only job/workflow names.

---

**Progress tracker:** `0 / 50` — update this line manually as you tick boxes off.