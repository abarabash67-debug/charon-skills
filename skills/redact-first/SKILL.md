---
id: "vibe-redact-first"
name: "RedactFirst: Pre-Publish IP Decoupling Guard"
version: "1.0.0"
description: "Anonymizes code, configs, and prompts before public release — strips client identifiers, internal endpoints, and personal data that would leak IP or violate NDAs."
category: "Security"
sub_category: "Agent Protection"
price_model: "free"
author: "a.barabash67"
tags: ["anonymization", "ip-protection", "pre-publish", "redaction", "nda-compliance", "open-source-safety"]
---

# RedactFirst: Pre-Publish IP Decoupling Guard

Strips client identifiers, internal endpoints, personal data, and proprietary naming from code, configs, and prompts before public release. Built for developers who want to publish their work — to open source, to a marketplace, to a blog post — without leaking their employer's data, their clients' names, or their own infrastructure.

A publish-ready artifact must not contain: real domains, real API keys, real employee names, real client names, real internal URLs, or anything that could be used to re-identify the source.

## What it does

- Scans code, configs, and prompts for direct identifiers: client names, employee names, internal domains, real URLs, API keys, tokens.
- Replaces identifiers with consistent, neutral placeholders that preserve structure (`acme-corp` → `client-a`, `api.internal.acme.com` → `api.vendor.example`).
- Flags indirect identifiers: unique project names, unusual tech-stack combinations, dated commit messages, regional references that narrow the source.
- Checks for secrets that would survive anonymization (embedded tokens, hashes, private keys) and escalates them separately.
- Produces a re-identification risk score: how easy it would be to trace the sanitized artifact back to the original author.
- Never removes code logic — only identifiers.

## When to use

- Publishing code to GitHub, an open-source repo, or a public gist.
- Uploading a skill, template, or boilerplate to a marketplace.
- Writing a blog post or tutorial that includes real config or logs.
- Sharing a project in a portfolio, resume, or job application.
- The user says: "sanitize this before I publish", "remove client info", "anonymize this repo", "check for leaks before open-sourcing".

## When NOT to use

- Internal-only repositories that will never be shared outside the organization.
- Code already published with attribution — removing identifiers post-publication is not the goal.
- Content that must keep attribution intact (academic citations, licensed code) — the skill flags but does not remove those.

## Prerequisites

- Read access to the files to be sanitized: source code, configs, prompts, docs.
- Optional: a list of known client names, internal domains, or personal identifiers to prioritize.
- Git history access if commit messages must also be sanitized.

## How it works

### Step 1 — Identify direct identifiers

Scan for and list every occurrence of:

1. **Client / employer names** — company names, product names, brand names.
2. **Internal domains** — `*.internal.*`, `*.corp.*`, private subdomains.
3. **Real URLs** — staging, production, or admin endpoints.
4. **Personal identifiers** — employee names, emails, phone numbers, usernames.
5. **Secrets** — API keys, tokens, passwords, private keys, connection strings.
6. **Regional references** — city names, country codes, timezone offsets that narrow the source.

### Step 2 — Replace with consistent placeholders

For each identifier, replace with a neutral placeholder and keep it consistent across the entire artifact:

| Original | Placeholder |
|---|---|
| `acme-corp` | `client-a` |
| `api.internal.acme.com` | `api.vendor.example` |
| `john.doe@acme.com` | `user@example.com` |
| `sk-live-abc123...` | `[REDACTED_SECRET]` |
| `Frankfurt DC` | `region-1` |

Consistency is critical: if `acme-corp` becomes `client-a` in one place and `company-x` in another, the anonymization fails.

### Step 3 — Flag indirect identifiers

Direct identifiers are easy. Indirect ones are what get authors caught:

- **Unusual tech-stack combinations** — "FastAPI + Redpanda + Trino + DuckDB in the same project" is more identifying than any domain name.
- **Dated commit messages** — "fix for the 2026-08-14 outage" narrows the search.
- **Unusual variable names** — a project-specific function name can be Googled.
- **License headers** — copyright notices with real company names.
- **Comments referencing internal tools** — "the internal Jira ticket PROJ-1234".

Flag these separately. The author decides whether to remove them.

### Step 4 — Escalate secrets separately

Secrets that survive anonymization (because they are embedded in binary, hashed, or encoded) are a different category. The skill does not try to remove them — it escalates them:

```text
🚨 SECRET SURVIVOR DETECTED — manual intervention required.
Location: <file>:<line>
Type: <token | key | hash | connection-string>
Action: rotate the secret at the source before publication.
```

Anonymized secrets are still leaked secrets. Rotation is the only correct fix.

### Step 5 — Produce the re-identification risk score

Output a categorical verdict:

- **CLEAR** (0–20): no direct identifiers remain, no indirect markers flagged.
- **WARNING** (21–50): indirect identifiers remain — author review required.
- **BREACH** (51–100): secrets survived, or direct client identifiers remain, or the artifact is trivially re-identifiable.

### Step 6 — Emit the sanitized artifact

Return the sanitized version alongside the report. Do not silently overwrite the original — the author must diff before publishing.

## Example

**User Request:** "Sanitize this config before I publish it to GitHub."

**Config to audit:**

```yaml
# Acme Corp production config — John Doe, 2026-08-14
database:
  host: db.internal.acme.com
  user: john.doe
  password: "sk-live-4f8a2b9c1d3e5f7a"
api:
  endpoint: https://api.acme-corp.com/v1
  token: "ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

**RedactFirst Action:**

```text
🔍 REDACTFIRST SCAN COMPLETE — BREACH
```

```json
{
  "risk_assessment": {
    "overall_status": "BREACH",
    "weighted_score": 94,
    "highest_risk_level": "HIGH"
  },
  "direct_identifiers": {
    "status": "HIGH",
    "findings": [
      { "type": "client_name", "detail": "acme-corp / Acme Corp" },
      { "type": "internal_domain", "detail": "db.internal.acme.com" },
      { "type": "personal_name", "detail": "John Doe" },
      { "type": "real_url", "detail": "api.acme-corp.com" }
    ]
  },
  "indirect_identifiers": {
    "status": "MEDIUM",
    "findings": [
      { "type": "dated_comment", "detail": "2026-08-14" }
    ]
  },
  "secret_survivors": {
    "status": "HIGH",
    "findings": [
      { "type": "api_key", "detail": "sk-live-4f8a2b9c1d3e5f7a — ROTATE BEFORE PUBLISHING" },
      { "type": "github_token", "detail": "ghp_xxxx... — ROTATE BEFORE PUBLISHING" }
    ]
  },
  "sanitized_artifact": "```yaml\n# Client-A production config — user@example.com, <date-redacted>\ndatabase:\n  host: db.vendor.example\n  user: user\n  password: \"[REDACTED_SECRET_ROTATE_AT_SOURCE]\"\napi:\n  endpoint: https://api.vendor.example/v1\n  token: \"[REDACTED_SECRET_ROTATE_AT_SOURCE]\"\n```",
  "compliance_signals": [
    {
      "module": "direct_identifiers",
      "trigger": "Client name and internal domain present in config",
      "risk": "HIGH",
      "action_required": "Replace with placeholders before publication. Consistent replacement required — do not vary placeholder names."
    },
    {
      "module": "secret_survivors",
      "trigger": "Two live credentials embedded in config",
      "risk": "HIGH",
      "action_required": "ROTATE both secrets at the provider before publishing. Anonymized secrets are still leaked secrets."
    }
  ]
}
```

## Output format

```json
{
  "risk_assessment": {
    "overall_status": "CLEAR | WARNING | BREACH",
    "weighted_score": 0,
    "highest_risk_level": "LOW | MEDIUM | HIGH"
  },
  "direct_identifiers": { "status": "...", "findings": [] },
  "indirect_identifiers": { "status": "...", "findings": [] },
  "secret_survivors": { "status": "...", "findings": [] },
  "sanitized_artifact": "...",
  "compliance_signals": []
}
```

- On BREACH: do not publish — rotate secrets and replace direct identifiers first.
- On WARNING: author review required — indirect identifiers remain.
- On CLEAR: safe to publish.
- On secret survivor: always escalate to rotation — never silently redact and move on.

## Guardrails

- Never silently overwrite the original artifact — always return a separate sanitized version.
- Never remove logic or code structure — only identifiers.
- Never claim that anonymization is complete if indirect identifiers remain.
- Never treat a redacted secret as safe — rotation is the only correct fix.
- Never vary placeholder names for the same original identifier — consistency is what makes anonymization work.
- Never sanitize a file without showing the diff to the author.
- Never assume the author knows which identifiers are sensitive — flag everything and let them decide.

## Continue with paid skills

If you find this useful, the following security skills extend it:

- **SecretSanitize ($19)** — purge leaked credentials from Git history and rotate them at the source.
- **ShadowAPIGuard ($19)** — audit AI-generated code for hidden API calls and copyleft license conflicts.
- **DataProxy ($149)** — prevent indirect prompt injection and exfiltration in third-party AI pipelines.
- **ZeroDayShield ($149)** — universal supply chain defense against zero-day, WAF-bypass, and post-exploitation artifacts.
