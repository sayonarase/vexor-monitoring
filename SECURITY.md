# Security Policy

## Supported versions

The Vexor monitoring platform follows a rolling-release model. Only the
**latest** released version (current `main` HEAD or most recent tag) is
supported with security fixes.

| Version | Supported |
|---------|-----------|
| latest  | ✅ |
| older   | ❌ |

## Reporting a vulnerability

**Do not file public GitHub issues for security problems.**

Send vulnerability reports privately by one of these channels:

1. **GitHub Security Advisory** — use the **Security** tab → *Report a
   vulnerability* on this repository. This creates a private advisory only
   maintainers can see.
2. **Email** — `security@vexor.local` (PGP key in `docs/security-pgp.asc`
   if/when published).

Include:

- the repository + commit / version,
- a clear description of the issue and impact,
- a proof-of-concept (PoC) request, payload, or reproduction steps if possible,
- your suggested fix or workaround if you have one.

We aim to:

- **acknowledge** new reports within **3 business days**,
- **triage** and confirm impact within **7 business days**,
- **fix or mitigate** confirmed criticals within **30 days** (HIGH within 60,
  MED within 90).

We credit reporters in the GitHub Security Advisory unless they request
anonymity.

## Scope

In scope: any code or configuration in this repository, including
workflows, scripts, and documentation that references credentials or
infrastructure.

Out of scope: third-party dependencies (file with the upstream project),
self-hosted instances misconfigured by operators (we document the secure
defaults; misconfiguration is an operational issue), and informational
findings such as missing security headers on local-only endpoints.
