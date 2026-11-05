# Databunker Documentation

Source for the official [Databunker](https://databunker.org/) documentation, published at **[docs.databunker.org](https://docs.databunker.org/)**. The developer hub is at [databunker.org/developers](https://databunker.org/developers/).

Databunker is a secure vault for personal data (PII, PHI, PCI, KYC) — it encrypts sensitive records and replaces them in your database with safe, random tokens. The docs cover two products:

- **Databunker Pro** (`pro/`) — the commercial, self-hosted PII vault and tokenization engine: guides, installation, administration, concepts, comparisons, and the `/v2` API reference.
- **Databunker OSS** (`oss/`) — the open-source edition: quickstart, installation, architecture, and the `/v1` API reference.

The site is built with [Mintlify](https://mintlify.com/). Navigation is defined in `docs.json`; API references are generated from `pro/api/openapi.yml` and `oss/api/openapi.yml`.

## Repository structure

```
docs.json                 Navigation, theme, navbar, footer. Every page must be listed here.
favicon.svg  logo/        Site branding.

pro/                      Databunker Pro (commercial, self-hosted)
  get-started/            Overview, quickstart, performance, PII vault, architecture,
                          security, licensing, FAQ
  installation/           Docker Compose, Kubernetes/Helm, unattended, Oracle backend,
                          credentials, production checklist
  administration/         Master key, key rotation, Shamir shares, backup & recovery,
                          monitoring, multi-tenancy, access control
  concepts/               Tokenization, file vault, search, versioning, sub-accounts, deployment
  comparisons/            vs AWS Cognito, vs HashiCorp Vault, vs building it yourself
  migrations/             Moving existing users in: SQL users table, AWS Cognito
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
| **[Quickstart](pro/get-started/quickstart.mdx)** — new page: one `docker run … demo` on in-memory SQLite, then create a user, store and retrieve an encrypted file. No database server, no setup wizard; ephemeral by design, so the fixed `DEMO` key cannot leak into production | Question 2 (partial) |
| **[Licensing and limits](pro/get-started/licensing.mdx)** — new page: what the licence controls (record count and expiry, never features), what counts as a record, licence scope per deployment, behaviour at the cap and at expiry, checking usage via `SystemGetSystemStats` | Question 6 |
| **[Wrapping key rotation](pro/administration/key-rotation.mdx)** — rewritten as a runbook: prerequisites, the `SystemGenerateWrappingKey` call, storing the returned key before restarting, verification, and a *What is recoverable* failure matrix | Half of question 5 |
| **[Master key](pro/administration/master-key.mdx)** — documents the two independent wrappings (`encryptedkey` / `recoverykey`), which is what makes rotation O(1) in vault size and keeps Shamir shares valid across rotations | Half of question 5 |
| **[Shamir keys](pro/administration/shamir-keys.mdx)** — rewritten around share custody: hex format, distribution across custodians *and* systems, audit cadence, and what happens below the three-share threshold | Half of question 5 |
| **[Database-level tenant isolation](pro/administration/multi-tenancy.mdx)** — the PostgreSQL `mtenant` / `madmin` role model, RLS policies, covered tables, why `xtokens` is excluded, and how the cross-tenant bypass is constrained | Question 7 (partial) |
| **[Security overview](pro/get-started/security-overview.mdx)** — corrected an unqualified "Shamir shares recover a lost wrapping key" claim, added the no-key-copy/no-escrow position, and added encryption service availability | Question 7 (partial) |
| **[Production checklist](pro/installation/production-checklist.mdx)** — new page: a go-live gate covering keys, database, network, access control, backup, licence capacity, and monitoring, linking out to the deep pages | Question 4 (partial) |
| **[Backup and recovery](pro/administration/backup-and-recovery.mdx)** — new page: the three artefacts a backup needs (database + wrapping key + licence), restore runbook, RPO/RTO, a restore drill, and the sanctioned `BulkListAllUsers` export path | Question 5, question 8 (out) |
| **Voice pass** — every `## Conclusion` / `## Why Choose` sales block removed site-wide, marketing openers cut, emoji headings gone, and **Next steps** added to 8 dead-end pages. Nine pages now route readers to the [quickstart](pro/get-started/quickstart.mdx), up from three | Question 1 (partial) |
| **[Monitoring](pro/administration/monitoring.mdx)** — new page: Prometheus scrape config and alert rules with thresholds from the benchmarks, plus licence usage and database health, which `/metrics` does not report | Question 4 (partial) |
| **[Migrations](pro/migrations/overview.mdx)** — new nav group after *Comparisons*: an overview of the shared pattern, plus guides for a [SQL users table](pro/migrations/sql-users-table.mdx) and [AWS Cognito](pro/migrations/aws-cognito.mdx) | Question 8 |
| **[SDK page](pro/developer-tools/overview.mdx)** — install commands and a create-a-user example for JavaScript, TypeScript, Python, PHP and Java; the Java client is documented via JitPack, since it is not on Maven Central | Question 2 (partial) |
| **[Session management](pro/howtos/session-management.mdx)** — new how-to: server-side sessions with sliding and absolute expiry, logout, log out everywhere, cookie settings. Writing it surfaced Redis-mode bugs in delete, list and tenant isolation, fixed in databunkerpro #61 | Question 2 (partial) |
| **[Developer hub](https://databunker.org/developers/)** — card-grid landing page on the marketing site: first API call in cURL and four SDKs, the round trip to a masked `UserGet`, benchmark strip, API reference and OpenAPI spec | Question 1 (partial) |

The highest-value remaining gaps are a changelog (P1.7) — the only thing between question 9 and a
tick — and the threat model (P1.2).

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

#### Patterns all 13 share that this repo is still missing

1. **Per-language code, everywhere.** Stripe/Twilio/Clerk/Supabase/Neon lead with SDK snippets and keep a language selector. Here `openapi.yml` has **zero** `x-codeSamples` across 92 endpoints, and `pii-vault.mdx` hand-writes raw `axios` and `requests` calls.
2. **Upgrades / version support.** Vault and Temporal both document them; there is no upgrade page here.
3. **Proof of life.** Changelog / release notes / version selector. All 13 have at least one. This repo has none — which, for self-hosted software a CTO must commit to running, reads as *abandoned or immature*.

### Score against the ten questions a CTO asks

| # | Question | Today | Still missing |
| --- | --- | --- | --- |
| 1 | What is this, do I need it? | ⚠️ | A Pro-vs-OSS decision page. |
| 2 | Can my team integrate it in one sprint? | ⚠️ | Framework guides. |
| 3 | Will it hold at our scale? | ✅✅ | — `performance.mdx` beats every site examined. |
| 4 | How do we run it in production? | ⚠️ | Upgrades and version support. |
| 5 | What breaks, and how badly? | ⚠️ | A threat model, and root-token compromise. |
| 6 | What does it cost; what does the licence gate? | ✅ | — |
| 7 | Will it survive procurement and audit? | ⚠️ | Compliance material is **buried in FAQ answers 6–8** — see P1.1. |
| 8 | Migration path in (and out)? | ✅ | — |
| 9 | Is the project alive? | ❌ | Changelog, releases, version support policy. |
| 10 | How does it compare to alternatives? | ✅ | Comparisons are filed under *Guides*, where an evaluator won't look. |

### Priority 1 — evaluation and integration surface

#### P1.1 Promote the buried gold: `pro/get-started/security-compliance.mdx`

The material already exists — framework list, inherited-certification argument, SOC 2 status, the
IRAP boundary-inheritance argument — but it is inside two FAQ accordions, where no security reviewer
will find it. Lift it into a page, placed next to `security-overview.mdx`.

**The split that keeps the two pages from duplicating each other:** `security-overview.mdx` explains
*how the product works* (AES-256, RLS, the key model) for an engineer. `security-compliance.mdx` maps
those mechanisms onto *what a framework requires* and states attestation status, for a reviewer. The
new page never re-explains a mechanism; it links.

Proposed shape:

1. **Databunker Pro is a control, not a certificate.** Say it up front — it provides technical
   controls frameworks require; it does not make you compliant. Vendors who blur this get caught.
2. **A shared responsibility table** — the centrepiece, because it answers the question behind every
   one of those FAQ answers and nothing states it directly today:

   | | Databunker Pro | Your cloud | You |
   | --- | --- | --- | --- |
   | Encryption at rest | AES-256 per record | disk encryption | key custody |
   | Tenant isolation | PostgreSQL RLS | — | tenant design |
   | Audit trail | per-record, per-field | — | retention, review |
   | Physical / infra controls | — | SOC 2 · ISO · IRAP | — |
   | Organisational controls | — | — | your own SOC 2 / ISO |

3. **Attestation status**, honestly tabulated: SOC 2 (in progress, managed portal only), ISO 27001
   (roadmapped), IRAP (not assessed — boundary inheritance), FIPS 140-2 (algorithms compliant).
4. **Framework control mapping** — GDPR Art. 25/32, PCI DSS scope reduction via tokenization,
   HIPAA §164.312, DPDPA, RBI localisation. Each row: requirement → feature → link.
5. **What to hand an auditor** — which pages are evidence, and what the audit trail can produce.

**Keep the MD5 disclosure.** That candour is what makes a reviewer trust the rest of the document.

**Two blockers before writing.** *(a)* The FAQ says SOC 2 is "expected within ~2–3 months", written
around July 2026 — that window has closed, and a stale forward-looking date is worse than none on a
page auditors read. Confirm current status first. *(b)* FERPA appears in one FAQ list and nowhere
else; either it belongs in the mapping or it should be dropped rather than carried forward as an
unsupported claim.

Give this page a **`last reviewed` date** in frontmatter — compliance is the one area where readers
need to know how current the claim is.

#### P1.2 `pro/get-started/threat-model.mdx` — including what Databunker does *not* protect

No page currently says: *if your application server is compromised, an attacker holding a valid token
can call the API.* Say it, then show the mitigations already shipped — masking policies, RBAC,
`select-security` bulk-export limits, rate limiting, audit trail, per-tenant isolation. Vendors who
state their limits get believed about their strengths.

#### P1.5 Add `x-codeSamples` to the OpenAPI spec

92 endpoints, still zero code samples. Add JS/Python/PHP/Java samples to the ~12 endpoints that matter
(`UserCreate`, `UserGet`, `UserUpdate`, `UserDelete`, `UserCreateBulk`, format-preserving token
create/get, `AppdataCreate`, agreement accept, session create).

#### P1.6 Upgrades and version support

Container tag policy, whether upgrades run schema migrations, rollback, supported-version window.
Vault keeps 11 versions of docs online; there is no upgrade page here at all.

#### P1.7 `changelog.mdx`

Start with the last five releases. Mintlify renders `<Update>` blocks for this. Its absence is read as
a signal about the project, not about the docs.

### Priority 2 — voice, structure, and polish

#### Voice follow-ups

The target register is `performance.mdx`: state measurements, name the bottleneck, concede weaknesses,
and mark projections as projections. Two follow-ups remain: a sweep of the unfalsifiable adjectives
(`advanced` ×5, `comprehensive` ×3, `robust`/`seamless` ×2 each), and cost-or-limits statements on
`tokenization.mdx` and `record-versioning.mdx`, which still make capability claims with no numbers.

#### Restructure the navigation

Pages have been placed into the existing groups deliberately (production checklist in *Installation*,
backup & recovery in *Administration*, licensing in *Get started*, migrations in their own group) —
don't undo that with a blanket restructure. What is still worth doing:

- **`Concepts` is a flat nine-item list** mixing search features (fuzzy, partial), data model
  (versioning, shared records, sub-accounts), security (select-security), and deployment
  (global-deployment). Split it by reader intent.
- **Move the comparisons out of *Guides*.** Evaluators do not browse a guides list.
- **Collapse the single-page `developer-tools` group.**
- **Add a "Pro or OSS?" page.** With two Mintlify `products`, a reader who lands on Pro never learns
  OSS exists, and vice versa. OSS is the top of the funnel — give it an explicit upgrade path.

#### Visual and mechanical

- `architecture.mdx` sets `style={{ backgroundColor: '#FFF' }}` on the diagram — that breaks in dark mode. Export diagrams with transparent backgrounds and wrap them in `<Frame>`.
- Add `icon:` frontmatter to group landing pages; Mintlify renders them in the sidebar and it reads as a maintained site.
- Consider a docs-wide `<Tabs>`/`<CodeGroup>` language convention so a reader who picks Python once sees Python throughout.

#### AI/agent surface — already ahead, finish the job

`contextual.options` already exposes copy/view/ChatGPT/Claude/Perplexity/MCP/Cursor/VS Code, and
`api/overview.mdx` links `llms-full.txt`. What's missing is what Stripe and Neon do: surface it on the
hub ("point your coding agent at our MCP server / feed it `llms-full.txt`") and add a short
*AI-assisted integration* page.

### Quick wins (under an hour each)

1. Add `⏱`/prerequisites callouts to the four install pages.
2. Fix the hardcoded white background on the architecture diagram.
3. Publish a `changelog.mdx` with the last five releases.

### Suggested sequence

| Wave | Contents | Why this order |
| --- | --- | --- |
| **1** | Security & compliance (P1.1) · threat model (P1.2) · comparisons out of *Guides* | Survives procurement and security review. |
| **2** | `x-codeSamples` (P1.5) · framework guides · upgrades (P1.6) · changelog (P1.7) | Makes integration and long-term ownership credible. |
| **3** | `Concepts` split · AI-integration page · quick wins | Consolidation, best done once the page set is stable. |

**Summary:** the docs are already excellent at the hard technical question (`performance.mdx` beats
every competitor site examined) and near-silent on the five commercial ones — *how do I get in, how do
I run it, what breaks, what does it cost, is it alive*. Close those five and the marketing register on
the entry pages, and the site starts reading like infrastructure a CTO can commit to rather than a
product being sold to them.

## Related repositories

- [securitybunker/databunker](https://github.com/securitybunker/databunker) — Databunker OSS source code
- [databunkerpro-js](https://github.com/securitybunker/databunkerpro-js) · [databunkerpro-python](https://github.com/securitybunker/databunkerpro-python) · [databunkerpro-php](https://github.com/securitybunker/databunkerpro-php) · [databunkerpro-java](https://github.com/securitybunker/databunkerpro-java) — Databunker Pro SDKs
