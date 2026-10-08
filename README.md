# Tanshi Agent Skills

[![skills.sh](https://skills.sh/b/tanshi-digital/tanshi-skills)](https://skills.sh)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A curated suite of specialized AI agent skills developed by **Tanshi Digital Solutions** for design, fintech, and modern web engineering.

These skills are designed to work across AI coding agents, including **Antigravity**, **Claude Code**, **Cursor**, **Codex**, and any tool supporting the open [Agent Skills](https://skills.sh) standard.

---

## Skills Included

| Skill | Description | Primary Use Case |
|---|---|---|
| **`lipila`** | Zambian payment gateway integration for Mobile Money (MTN, Airtel, Zamtel), Cards, and Bank payouts. | E-commerce, SaaS, billing, and checkout flows in Zambia. |
| **`paper-ui-ux`** | Tactile, skeuomorphic "Paper & Ink" design system (warm paper textures, desk items, Post-it notes, stamps). | Storybook websites, bookstores, creative portfolios, and tactile UI. |
| **`svg-personality`** | Anti-generic visual design guide using SVG filters (noise, wobbly edges, line boil, hand-drawn underlines, text-on-path). | Eliminating bland, template-looking UI with lightweight vector character. |

---

## Installation

You can install these skills directly into your project or globally on your machine using `npx skills`.

### Install all skills:
```bash
# In your project:
npx skills add <your-username>/tanshi-skills

# Globally (for all projects):
npx skills add <your-username>/tanshi-skills -g
```

### Install individual skills:

```bash
# Lipila Payment Gateway Skill
npx skills add <your-username>/tanshi-skills --skill lipila

# Paper & Ink UI/UX Design System Skill
npx skills add <your-username>/tanshi-skills --skill paper-ui-ux

# SVG for Personality Skill
npx skills add <your-username>/tanshi-skills --skill svg-personality
```

---

## Skill Details

### 1. `lipila`
Full integration runbook for Lipila, the Zambian payment gateway.
- **Telecom Normalization:** Formats Zambian phone numbers into international MSISDN (`260XXXXXXXXX`) and auto-detects telco networks (097/077/057 Airtel, 096/076/056 MTN, 095/075/055 Zamtel).
- **Collections:** USSD Mobile Money push prompts, hosted 3D-Secure card checkout, and status polling.
- **Disbursements:** Mobile money payouts and bank transfers.
- **Webhook Security:** Cryptographically verifies incoming `v1` HMAC-SHA256 signatures with 300-second timestamp tolerance and replay attack mitigation.
- **Security:** Credentials (`LIPILA_API_KEY`, `LIPILA_WEBHOOK_SECRET`) are configured via local `.env` variables and are never hardcoded.

### 2. `paper-ui-ux`
A design system inspired by school exercise books, cloth-bound library volumes, Post-it notes with tape, and aged paper on a wooden desk.
- Warm cream background palette (`#FBF6E9`) and fountain pen navy (`#1E2B4A`).
- Analog rotations, wobbly sketch lines, torn page borders, and library checkout slips.
- Literary serif typography and handwriting accents.

### 3. `svg-personality`
A practical guide to escaping generic rounded-box templates and AI-generated SaaS defaults.
- **Materials via Filters:** Uses `feTurbulence` and displacement maps for wobbly lines, paper grain, wood grain, deboss, and liquid goo.
- **Hand-drawn Accents:** Animated path-draw underlines, circles, and emphasis markers.
- **Shapes that Aren't Boxes:** Curving section edges, cut-out masks, and halftone hatching.
- **Restraint:** Focuses personality on 3–7 key touchpoints while keeping performance and accessibility tight.

---

## Security & Secrets

None of these skills contain private API keys, client secrets, or live wallet IDs. When using skills like `lipila`, the agent will guide you to configure your credentials inside your local `.env.local` or environment secrets manager:

```env
LIPILA_ENV=sandbox # or "live"
LIPILA_BASE_URL=https://api.lipila.dev/api/v1
LIPILA_API_KEY=your_key_here
LIPILA_WEBHOOK_SECRET=your_signing_secret_here
```

---

## License

MIT License. Created with  by Tanshi Digital Solutions.
