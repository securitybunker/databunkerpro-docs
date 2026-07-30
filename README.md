# Databunker Documentation

Source for the official [Databunker](https://databunker.org/) documentation, published at **[docs.databunker.org](https://docs.databunker.org/)**.

Databunker is a secure vault for personal data (PII, PHI, PCI, KYC) — it encrypts sensitive records and replaces them in your database with safe, random tokens. The docs cover two products:

- **Databunker Pro** (`pro/`) — the commercial, self-hosted PII vault and tokenization engine: guides, installation, administration, concepts, comparisons, and the `/v2` API reference.
- **Databunker OSS** (`oss/`) — the open-source edition: quickstart, installation, architecture, and the `/v1` API reference.

The site is built with [Mintlify](https://mintlify.com/). Navigation is defined in `docs.json`; API references are generated from `pro/api/openapi.yml` and `oss/api/openapi.yml`.

## Repository structure

```
docs.json                 Navigation, theme, navbar, footer. Every page must be listed here.
favicon.svg  logo/        Site branding.

pro/                      Databunker Pro (commercial, self-hosted)
  get-started/            Overview, PII vault, architecture, security, performance, FAQ
  installation/           Docker Compose, Kubernetes/Helm, unattended, Oracle backend, credentials
  administration/         Master key, key rotation, Shamir shares, multi-tenancy, access control
  concepts/               Tokenization, file vault, search, versioning, sub-accounts, deployment
  comparisons/            vs AWS Cognito, vs HashiCorp Vault, vs building it yourself
  howtos/                 Short task-focused operational guides
  developer-tools/        SDK index
  api/                    overview / authentication / errors / pagination + openapi.yml (/v2)

oss/                      Databunker OSS (open source)
  get-started/            Overview, quickstart, architecture, examples
  installation/           Installation guide
  api/                    overview + openapi.yml (/v1)
```

Images live next to the page that uses them (for example `pro/get-started/databunker-diagram.png`)
and are referenced by absolute site path: `/pro/get-started/databunker-diagram.png`.

## Local development

Install the [Mintlify CLI](https://www.npmjs.com/package/mint):

```
npm i -g mint
```

Then run from the repo root (where `docs.json` lives):

```
mint dev
```

View the local preview at `http://localhost:3000`.

## Publishing

Changes merged to `main` are deployed to production automatically via the Mintlify GitHub app.

## Docs improvement plan (CTO audience)

The primary buying audience for Databunker Pro is the CTO or engineering lead who has to approve
running a PII vault in production. This section benchmarks the site against 13 developer-tool
documentation sites (examined July 2026) and lists the prioritised gaps.

### Done so far

| Change | Closes |
| ------ | ------ |
| **[Licensing and limits](pro/get-started/licensing.mdx)** — new page: what the licence controls (record count and expiry, never features), what counts as a record, licence scope per deployment, behaviour at the cap and at expiry, checking usage via `SystemGetSystemStats` | Question 6 |
| **[Wrapping key rotation](pro/administration/key-rotation.mdx)** — rewritten as a runbook: prerequisites, the `SystemGenerateWrappingKey` call, storing the returned key before restarting, verification, and a *What is recoverable* failure matrix | Half of question 5 |
| **[Master key](pro/administration/master-key.mdx)** — documents the two independent wrappings (`encryptedkey` / `recoverykey`), which is what makes rotation O(1) in vault size and keeps Shamir shares valid across rotations | Half of question 5 |
| **[Shamir keys](pro/administration/shamir-keys.mdx)** — rewritten around share custody: hex format, distribution across custodians *and* systems, audit cadence, and what happens below the three-share threshold | Half of question 5 |
| **[Database-level tenant isolation](pro/administration/multi-tenancy.mdx)** — the PostgreSQL `mtenant` / `madmin` role model, RLS policies, covered tables, why `xtokens` is excluded, and how the cross-tenant bypass is constrained | Question 7 (partial) |
| **[Security overview](pro/get-started/security-overview.mdx)** — corrected an unqualified "Shamir shares recover a lost wrapping key" claim, added the no-key-copy/no-escrow position, and added encryption service availability | Question 7 (partial) |
| Supporting fixes — the 14-day trial window (previously undocumented, a silent day-15 failure), all licence-key paths routed through the portal, `record-versioning` note that versions do not multiply record count, `errors.mdx` cross-links, and the Pro `openapi.yml` licence corrected from MIT to commercial terms | — |

**Corrections that came out of this work.** Writing the licensing page surfaced two code
deltas in `LicenseGetLimitations` (`databunkerpro/src/license_api.go`): the trial cap returned
10,000 records instead of 1,000, and the trial period was 30 days instead of 14. Both are fixed on
the `fix-trial-license-limits` branch. Documenting a mechanism against its source tends to find
these; it is worth doing deliberately.

**Still open on question 5:** the threat model, and the failure modes beyond key loss (database loss,
root-token compromise). **Still the highest-value remaining gap:** the users-table migration guide
(P0.3), followed by the Pro quickstart (P0.2) and the production checklist (P0.4).

### Sites benchmarked

| Tool | Category | The pattern worth stealing |
| --- | --- | --- |
| **Stripe** | payments | Dual entry on the hub: *by use case* ("Accept payments online") **and** *by getting-started step*. Plus an agent-first block: install the CLI, load the `stripe-docs` skill. |
| **Twilio** | comms API | Dedicated **error-codes reference**, **changelog**, and **pricing** as first-class nav items — not buried in prose. |
| **Supabase** | database/BaaS | **11 framework quickstarts** as a logo grid above the fold ("Connect a framework"). AI prompt + CLI install in the first screen. |
| **Clerk** | auth | Quickstarts split by **frontend framework (11)**, **backend framework (8)**, **community SDKs (8)**, plus a **"How Clerk works"** architecture page and a **"Securing your application"** page. |
| **Neon** | Postgres | `npx neon@latest init` as *the* first step. **30+ framework guides**. A whole **"AI tools"** section (Cursor, Claude Code, Codex, Copilot). |
| **ClickHouse** | database | Four hub cards: **Get started / Guides / Reference / Solutions** — with *Migration guides* and *Best practices* inside Get started. MCP server + "Ask AI" in the hero. |
| **Temporal** | workflow infra | **"Production Deployment"** is a top-level section, equal in weight to Quickstart and Developer Guide. Self-host and cloud share one path. |
| **HashiCorp Vault** | secrets (competitor) | An explicit **OPERATIONS** section: configuration → deployment → **upgrades** → sysadmin. Plus **Internals** (architecture), **FIPS/HSM hardening**, a **glossary**, and **11 prior versions** of docs kept online. |
| **Tailscale** | security infra | IA split by *role*: **Get started / Manage your tailnet / Expand / Resources & reference**, with **security, privacy & compliance** as its own resource area so it doesn't clutter the product nav. |
| **PostHog** | analytics | Docs-as-marketing. Plain, casual voice. Multi-surface: web, CLI, MCP, IDE. |
| **Infisical** | secrets, open-core | Clean **self-hosting vs cloud** split, feature-module nav, `llms.txt` index. |
| **Skyflow** | PII vault (closest competitor) | **Fundamentals** (get started + *usage patterns* for PII / PCI / LLM privacy / analytics + core concepts), then **Governance**, **Tokenization**, **Connections**, **SDKs**, **API Reference**, **Release Notes**. |
| **Basis Theory** | tokenization/PCI vault | Nav organised by **use case** — including **Migrations** as a use case — and a **Platform** section containing **Testing** and an interactive **Production Checklist** (account, PCI, access controls, operations, code). |

#### Five patterns all 13 share that this repo is missing

1. **A hub page.** Every one of them has a landing page with a card grid. `docs.json` here has no root page — `docs.databunker.org` drops the reader straight into `pro/get-started/overview`, 59 lines of prose.
2. **A time-boxed quickstart above the fold.** Pro has none. OSS has one (`docker run` + 3 curls). For Pro the shortest path to a first API call is: clone a repo → generate env files → `docker compose up` → tail logs for a 6-digit code → open a web UI → generate admin credentials. That's two pages and a browser detour before any code.
3. **Per-language code, everywhere.** Stripe/Twilio/Clerk/Supabase/Neon lead with SDK snippets and keep a language selector. Here: `CodeGroup` is used on **zero** pages, `openapi.yml` has **zero** `x-codeSamples` across 92 endpoints, and `pii-vault.mdx` hand-writes raw `axios` and `requests` calls — actively teaching readers to bypass the four SDKs that exist.
4. **An operations story.** Vault has upgrades + sysadmin; Temporal has Production Deployment; Basis Theory has a Production Checklist. Here there is no page for backup/DR, monitoring, upgrades, or go-live hardening. Grep: `changelog` → 0 hits, `Prometheus` → 0, `upgrade` → 1 (in a comparison page), `SLA` → 0.
5. **Proof of life.** Changelog / release notes / version selector. All 13 have at least one. This repo has none — which, for self-hosted software a CTO must commit to running, reads as *abandoned or immature*.

### Score against the ten questions a CTO asks

| # | Question | Today | Where it stands |
| --- | --- | --- | --- |
| 1 | What is this, do I need it? | ⚠️ | Positioning exists but is marketing-voiced; no hub, no Pro-vs-OSS decision page. |
| 2 | Can my team integrate it in one sprint? | ❌ | No Pro quickstart, no framework guides, SDK page is 4 links to GitHub (32 lines, no install command, no example). |
| 3 | Will it hold at our scale? | ✅✅ | `pro/get-started/performance.mdx` is genuinely best-in-class — measured, honest, with a sizing table and a *Confidence* column. Better than anything the 13 sites have on this axis. |
| 4 | How do we run it in production? | ❌ | No production checklist, no HA/scaling guide, no monitoring, no upgrade path. |
| 5 | What breaks, and how badly? | ⚠️ | Key-loss failure modes and the recovery matrix are now documented (see [Done so far](#done-so-far)). Still missing: a threat model, and the other failure modes — database loss, root-token compromise. |
| 6 | What does it cost; what does the licence gate? | ✅ | [Licensing and limits](pro/get-started/licensing.mdx) covers record caps, what counts as a record, licence scope, and behaviour at the cap and at expiry. Prices stay on the marketing site. |
| 7 | Will it survive procurement and audit? | ⚠️ | Excellent material — SOC 2 in progress, no product-level IRAP, inherits the cloud IRAP boundary, FIPS 140-2 detail, an honest disclosure of non-cryptographic MD5 use — all **buried in FAQ answers 6–8**. |
| 8 | Migration path in (and out)? | ❌ | Nothing. This is the #1 blocking question for anyone with an existing `users` table. |
| 9 | Is the project alive? | ❌ | No changelog, no releases, no version support policy. |
| 10 | How does it compare to alternatives? | ✅ | Three comparison pages exist (Cognito, Vault, build-your-own). Good — but filed under *Guides*, where an evaluator won't look. |

**Read across that column: the docs are strong where an engineer has already committed, and empty
exactly where a CTO decides.**

### Priority 0 — the five pages that change the outcome

#### P0.1 A hub page (`index.mdx`)

Card grid, no prose wall. Four cards mirroring ClickHouse: **Get started · Evaluate · Integrate · Operate**.
Below it: the three sandbox demos (user-table replacement, PCI tokenization, consent management) — today
they appear on exactly one page. Below that: "Pro or OSS?" and the headline benchmark number.

#### P0.2 `pro/get-started/quickstart.mdx` — "First token in 5 minutes"

Non-negotiable. One page, copy-pasteable, no web-UI detour:

```
docker compose up  →  grab access code  →  create root token  →  UserCreate  →  UserGet
```

Show it in four tabs (`curl`, JS, Python, Java) using the actual SDKs. End with three "next step"
links (search, tokenize a card, migrate your users table). Put a **"⏱ 5 minutes"** badge and a
prerequisites `<Note>` at the top — every one of the 13 sites does this.

#### P0.3 `pro/integrate/migrate-users-table.mdx` — "Migrate an existing users table"

The single highest-value missing page. Basis Theory makes *Migrations* a top-level use case;
ClickHouse puts *Migration guides* inside Get Started. Every ingredient already exists here:

- `UserCreateBulk` at ~5,800 rec/s measured (and the honest "the database becomes the bottleneck" caveat)
- The published loader scripts in `databunkerpro-python`
- The fact — currently buried in `pii-vault.mdx` §2 — that **tokens are stable, deterministic join keys within a tenant**, so a SQL join on `email` becomes a join on `user_token` with no vault round-trip

Structure it as: assess → dual-write → backfill → verify → cut over `email` column to `user_token` →
rollback plan → decommission. Include the downtime estimate straight from the sizing table
(10 M ≈ 28 min, 100 M ≈ 2.3 h).

#### P0.4 `pro/deploy/production-checklist.mdx`

Everything an infra lead needs before go-live, currently scattered across `security-overview`,
`architecture`, `master-key`, `shamir-keys`, `select-security` and the FAQ. Steal Basis Theory's
six-bucket shape: **Keys & secrets · Database · Network/TLS · Access control · Backup & recovery · Observability**.
Make it a literal checklist with links out to the deep pages.

#### P0.5 `pro/deploy/backup-and-recovery.mdx`

For a vault, this is the fear that kills deals. State plainly:

- What must be backed up: database **+ wrapping key + licence** (all three, or the backup is useless)
- **If the wrapping key is lost, the data is unrecoverable.** Say it in bold. Then point at Shamir 3-of-5.
- Restore runbook, RPO/RTO guidance, a "test your restore" drill
- **The sanctioned full-export path.** `select-security` deliberately blocks bulk extraction — so document how a *legitimate* exit/portability export is performed. "Can I get my data out if I stop paying?" is existential for a vault and is currently unanswered.

### Priority 1 — evaluation and integration surface

#### P1.1 Promote the buried gold: `pro/evaluate/security-and-compliance.mdx`

Lift FAQ answers 6–8 into a real page: framework-by-framework control mapping (GDPR Art. 25/32,
PCI DSS scope reduction via tokenization, HIPAA §164.312, DPDPA, RBI localisation, ISO 27001, SOC 2),
the FIPS 140-2 detail, current attestation status, and the IRAP boundary-inheritance argument.
**Keep the MD5 disclosure.** That kind of candour is what makes a security reviewer trust the rest
of the document — it is a feature, not a liability, and right now it's hidden in a Q&A.

#### P1.2 `pro/evaluate/threat-model.mdx` — including what Databunker does *not* protect

No page currently says: *if your application server is compromised, an attacker holding a valid token
can call the API.* Say it, then show the mitigations already shipped — masking policies, RBAC,
`select-security` bulk-export limits, rate limiting, audit trail, per-tenant isolation. Vendors who
state their limits get believed about their strengths.

#### ~~P1.3 `pro/evaluate/licensing-and-limits.mdx`~~ — done

Shipped as [`pro/get-started/licensing.mdx`](pro/get-started/licensing.mdx), placed in *Get started*
rather than a new *Evaluate* group so it is reachable before that restructure happens.

#### P1.4 Rewrite `developer-tools/overview.mdx` into a real SDK page

Today: 32 lines, four cards pointing at GitHub. It should carry, per language: install command,
a 10-line create-then-read example, current version, and a link to the reference. Then add framework
guides — Express, Next.js, Django, Spring Boot, Laravel — following Supabase/Clerk/Neon. Also give
`@databunker/store` and `@databunker/session-store` real pages; they're mentioned once, in step 5 of
the OSS quickstart.

#### P1.5 Add `x-codeSamples` to the OpenAPI spec

92 endpoints, zero code samples, and `servers:` lists only `http://localhost:3000`. Add JS/Python/PHP/Java
samples to the ~12 endpoints that matter (`UserCreate`, `UserGet`, `UserUpdate`, `UserDelete`,
`UserCreateBulk`, format-preserving token create/get, `AppdataCreate`, agreement accept, session create),
and add a `https://databunker.internal.example.com` server entry so the reference doesn't look like a
localhost toy.

#### P1.6 Ops pages: `high-availability.mdx`, `monitoring.mdx`, `upgrades.mdx`

- **HA/scaling** — the facts already exist (stateless app tier, shared Redis, database is the bottleneck past ~12 K rec/s); turn them into a deployment topology page.
- **Monitoring** — health endpoint, what to alert on (database CPU ≥ 70%, index cache-hit < 97%, 403 rate, audit-write failures). Zero coverage today.
- **Upgrades & version support** — container tag policy, whether upgrades run schema migrations, rollback, supported-version window. Vault keeps 11 versions of docs online; there is no upgrade page here at all.

#### P1.7 `changelog.mdx`

Start with the last five releases. Mintlify renders `<Update>` blocks for this. Its absence is read as
a signal about the project, not about the docs.

### Priority 2 — voice, structure, and polish

#### Fix the voice (this is the "appealing to a CTO" lever)

The repo contains two distinct registers, and the wrong one is on the entry pages.

**Standardise on the `performance.mdx` register.** It states measurements, names the bottleneck,
concedes weaknesses ("Oracle writes ~2.4× slower", "~3× the on-disk size"), and marks projections
as projections. A CTO reading that page trusts the product. Apply it everywhere.

**Delete the other register.** Concretely:

- `pii-vault.mdx` opens *"In today's data-driven world…"*; `architecture.mdx`'s description opens *"In today's digital landscape…"*. Cut both — open with what the thing does.
- Drop the closing sales blocks: *"Why Choose Databunker Pro?"*, *"🎯 Conclusion"*, *"The Bottom Line"*, *"Ready to eliminate PII exposure…"*. Mid-document CTAs signal vendor content. Replace with a "Next steps" link list, and put a single "get a trial key / talk to an engineer" card at the end of *evaluation* pages only.
- Remove emoji headings (`🔐 ⚠️ ⚙️ 💻 🛡️ 🎯`, `💣`, `🚀`). None of the 13 reference sites uses them in headings.
- Cut unfalsifiable adjectives — "enterprise-grade", "next-generation", "robust", "business imperative".

#### Single-source the duplicated facts

"AES-256 / RLS multi-tenancy / Shamir / stateless" is restated in `architecture.mdx`,
`security-overview.mdx`, `faq.mdx`, and `pii-vault.mdx`. The multi-tenancy + error-code tables are
duplicated verbatim between `pro/api/overview.mdx` and `openapi.yml`'s `info.description`. Pick one
home for each fact; link to it. Four copies will drift.

#### Restructure the navigation

`Concepts` is currently a flat nine-item list mixing search features (fuzzy, partial), data model
(versioning, shared records, sub-accounts), security (select-security), and deployment
(global-deployment). Regroup by reader intent:

```
Databunker Pro / Guides
  Start here      overview · quickstart(NEW) · pro-vs-oss(NEW) · architecture · how it works
  Evaluate        performance ⭐ · security & compliance(NEW) · threat model(NEW) ·
                  licensing & limits(NEW) · vs Cognito · vs Vault · vs build-your-own · FAQ
  Integrate       SDKs · framework guides(NEW) · store & retrieve PII · search (exact/fuzzy/partial) ·
                  card tokenization · files · sessions · consent & legal basis ·
                  migrate your users table(NEW) ⭐
  Deploy          docker compose · helm · unattended · oracle · admin credentials ·
                  production checklist(NEW) · HA & scaling(NEW) · backup & DR(NEW) ⭐ ·
                  monitoring(NEW) · upgrades(NEW)
  Operate         master key · key rotation · shamir · multi-tenancy · access control ·
                  bulk export controls · how-tos
  Changelog(NEW)
```

Three specific moves: **comparisons out of Guides into Evaluate** (evaluators don't browse guides);
**performance to position 2 in Start here** (it's the strongest asset — lead with it); and collapse
the single-page `developer-tools` group into `Integrate`.

Also add a **"Pro or OSS?"** page. With two Mintlify `products`, a reader who lands on Pro never
learns OSS exists, and vice versa. OSS is the top of the funnel — give it an explicit upgrade path page.

#### Visual and mechanical

- `architecture.mdx` sets `style={{ backgroundColor: '#FFF' }}` on the diagram — that breaks in dark mode. Export diagrams with transparent backgrounds and wrap them in `<Frame>`.
- Add `icon:` frontmatter to group landing pages; Mintlify renders them in the sidebar and it reads as a maintained site.
- Convert the FAQ's 18 answers to `<Accordion>` — it's currently an unscannable wall.
- Add `⏱ time` + prerequisites callouts to every install page (only some have them).
- Consider a docs-wide `<Tabs>`/`<CodeGroup>` language convention so a reader who picks Python once sees Python throughout.

#### AI/agent surface — already ahead, finish the job

`contextual.options` already exposes copy/view/ChatGPT/Claude/Perplexity/MCP/Cursor/VS Code, and
`api/overview.mdx` links `llms-full.txt`. That's ahead of most of the 13. What's missing is what
Stripe and Neon do: surface it on the **hub page** ("point your coding agent at our MCP server / feed
it `llms-full.txt`") and add a short *AI-assisted integration* page. For a self-hosted vault, "your
agent will wire the SDK correctly on the first try" is a real CTO time-saver, and it's cheap to offer
because the plumbing already exists.

### Quick wins (under an hour each)

1. Add `index.mdx` hub with a card grid.
2. Move `performance` to position 2 under *Get started*.
3. Strip the "In today's …" openers and the "Why choose / Bottom line / Ready to…" closers from `pii-vault.mdx` and `architecture.mdx`.
4. Remove emoji from all headings.
5. Add a non-localhost entry to `openapi.yml` `servers:`.
6. Add `⏱`/prerequisites callouts to the four install pages.
7. Wrap the FAQ in `<Accordion>`.
8. Fix the hardcoded white background on the architecture diagram.
9. Delete the duplicated multi-tenancy/error tables from `openapi.yml` `info.description`, linking to `/pro/api/overview` instead.
10. Publish a `changelog.mdx` with the last five releases.

### Suggested sequence

| Wave | Contents | Why this order |
| --- | --- | --- |
| **1** | Hub page · Pro quickstart · all ten quick wins | Fixes first impression and the 5-minute path. Cheapest, most visible. |
| **2** | Migrate-users-table · production checklist · backup & DR | The three pages that unblock an actual buying decision. |
| **3** | Security & compliance · threat model · licensing & limits · comparisons moved to *Evaluate* | Survives procurement and security review. |
| **4** | SDK page rewrite · `x-codeSamples` · framework guides · HA/monitoring/upgrades · changelog | Makes integration and long-term ownership credible. |
| **5** | Nav restructure · voice pass across all pages · single-sourcing · AI-integration page | Consolidation, best done once the page set is stable. |

**Summary:** the docs are already excellent at the hard technical question (`performance.mdx` beats
every competitor site examined) and near-silent on the five commercial ones — *how do I get in, how do
I run it, what breaks, what does it cost, is it alive*. Close those five and the marketing register on
the entry pages, and the site starts reading like infrastructure a CTO can commit to rather than a
product being sold to them.

## Related repositories

- [securitybunker/databunker](https://github.com/securitybunker/databunker) — Databunker OSS source code
- [databunkerpro-js](https://github.com/securitybunker/databunkerpro-js) · [databunkerpro-python](https://github.com/securitybunker/databunkerpro-python) · [databunkerpro-php](https://github.com/securitybunker/databunkerpro-php) · [databunkerpro-java](https://github.com/securitybunker/databunkerpro-java) — Databunker Pro SDKs
