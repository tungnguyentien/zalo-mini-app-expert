---
name: zalo-mini-app-expert
description: This skill bundle consolidates the knowledge base in references/ to support Zalo Mini App development: ZMP SDK (client-side), Open APIs (server-side), ZaUI Components, app configuration, permissions, payment, devtools, deployment, compatibility, troubleshooting, best practices, and end-to-end use cases.
---

# Zalo Mini App (Skill Bundle)

## Purpose

This skill bundle consolidates the knowledge base in `references/` to support Zalo Mini App development: ZMP SDK (client-side), Open APIs (server-side), ZaUI, app configuration, permissions, payment, devtools, deployment, compatibility, troubleshooting, best practices, and end-to-end use cases.

## When to use

- Look up **client-side APIs** (ZMP SDK).
- Implement **server-side integrations** (Open APIs, webhooks, token verification, token decoding).
- Build **UI with ZaUI** (React components, layout, router, feedback patterns).
- Standardize **configuration**, **permissions**, **payment**, **devtools/deployment**, **compatibility**, and **troubleshooting**.

## Knowledge sources

All detailed content lives under `references/`, organized by topic for fast retrieval.

## Non-negotiable rules (high-signal)

- **Never call Open APIs / Graph APIs from the client**. Anything requiring secrets or API keys must run server-side.
- **Always check compatibility**. APIs may require minimum SDK/Zalo versions and/or approved permissions.
- **Always handle errors**. Use error codes to drive user-facing fallbacks and recovery paths.
- **Request permissions contextually**. Do not prompt on app launch; explain why the permission is needed with an appropriate UX.

## References index

See the topic files in `references/` (migrated from the legacy `skills/*.md` flat layout).

