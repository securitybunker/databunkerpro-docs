# Databunker Documentation

Source for the official [Databunker](https://databunker.org/) documentation, published at **[docs.databunker.org](https://docs.databunker.org/)**.

Databunker is a secure vault for personal data (PII, PHI, PCI, KYC) — it encrypts sensitive records and replaces them in your database with safe, random tokens. The docs cover two products:

- **Databunker Pro** (`pro/`) — the commercial, self-hosted PII vault and tokenization engine: guides, installation, administration, concepts, comparisons, and the `/v2` API reference.
- **Databunker OSS** (`oss/`) — the open-source edition: quickstart, installation, architecture, and the `/v1` API reference.

The site is built with [Mintlify](https://mintlify.com/). Navigation is defined in `docs.json`; API references are generated from `pro/api/openapi.yml` and `oss/api/openapi.yml`.

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

## Related repositories

- [securitybunker/databunker](https://github.com/securitybunker/databunker) — Databunker OSS source code
- [databunkerpro-js](https://github.com/securitybunker/databunkerpro-js) · [databunkerpro-python](https://github.com/securitybunker/databunkerpro-python) · [databunkerpro-php](https://github.com/securitybunker/databunkerpro-php) · [databunkerpro-java](https://github.com/securitybunker/databunkerpro-java) — Databunker Pro SDKs
