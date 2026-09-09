# MailTool Pro — Product Requirements Document (PRD)

**Version:** 1.0 (Draft)
**Date:** September 9, 2026
**Status:** Planning
**Repository:** Mailtool-pro

---

## 1. Overview

MailTool Pro is a **free online mail tools website** — redesigned from scratch, evolving from `nexitool-pro` into a **mail-tools-only** platform.

The existing mail tools already work (currently inside nexitool-pro). This project gives them a dedicated home with a completely new design, modern stack, and a focused identity.

---

## 2. Vision

> Sirf aur sirf mail tools. Ek focused, fast, beautiful website — jo sirf ek kaam kare aur wo best kare.

A single-purpose tools site that feels modern, loads instantly, and works flawlessly on mobile.

---

## 3. Core Principles

| Principle | Decision |
|-----------|----------|
| **Focus** | Only mail tools. Nothing else. |
| **No login** | Zero accounts, zero signups. Open the site → use the tool. |
| **Privacy-first** | Minimal data collection; no user accounts means no personal data liability. |
| **Abuse protection** | Server-side rate limiting so no user can spam requests. |
| **Speed** | Fast load times, lightweight pages. |
| **Mobile-first** | Designed for mobile screens first, scaled up to desktop. |
| **Unique design** | Not a template look — a distinctive, current-generation visual identity. |

---

## 4. Scope

### 4.1 In Scope
- Mail tools only (ported from nexitool-pro + new ones)
- Server-side **rate limiting**
- **Supabase** backend
- Mobile-friendly, responsive UI
- Fresh, unique design (new fonts, colors, layout — everything)

### 4.2 Out of Scope
- User accounts / login system
- Image, PDF, or AI tools (those stay on nexitool-pro)
- Payments / monetization (for now)

---

## 5. Tools (Draft List — To Finalize)

Existing mail tools from nexitool-pro will be ported first. Candidate tool ideas for the platform:

1. **Email Validator** — syntax + domain (MX) check
2. **Temp Mail / Disposable Inbox** — receive emails without an account
3. **Email Header Analyzer** — parse and explain raw headers
4. **SPF / DKIM / DMARC Checker** — domain authentication lookup
5. **MX Record Lookup**
6. **Mailto Link Generator**
7. **Email Signature Generator**
8. **Subject Line Tester**
9. **Disposable Email Detector** — is this address a temp mail?
10. **Email Extractor** — pull emails out of any text

> Final launch list to be decided in Phase 1.

---

## 6. Technical Architecture

| Layer | Decision |
|-------|----------|
| **Backend** | Supabase (database, edge functions, rate-limit logic) |
| **Rate limiting** | Server-side (Supabase Edge Functions) — per-IP request caps |
| **Frontend** | TBD — must be lightweight & fast (static-first preferred) |
| **Hosting** | TBD (Vercel / Netlify / Cloudflare Pages — decide in Phase 1) |
| **Design** | Figma (Pro plan available) |

### Rate Limiting (planned behavior)
- Per-IP request limits per tool (e.g., X requests / minute)
- Graceful error states when limit is hit (no broken UI)
- Server-enforced — cannot be bypassed from the browser

---

## 7. Non-Functional Requirements

- **Performance:** fast first paint; target Lighthouse score 90+
- **Mobile:** fully responsive, touch-friendly, thumb-zone aware
- **SEO:** each tool gets its own indexable page with proper meta/OG tags
- **Reliability:** tools must fail gracefully (clear errors, no dead ends)
- **Accessibility:** semantic HTML, keyboard navigable, readable contrast

---

## 8. Design Direction

- **Unique identity** — no generic template feel
- Modern typography & color system (chosen for this era, not recycled from nexitool-pro)
- Design work in **Figma Pro** → then implemented in code
- Design tokens (colors, spacing, type scale) documented before build

---

## 9. Open Questions

| # | Question | Owner |
|---|----------|-------|
| 1 | Final website name? (repo name is a working title) | Lio |
| 2 | Which tools launch first? (pick 3–5 from the draft list) | Lio |
| 3 | Frontend stack — plain HTML/JS or a framework (Astro / Next.js)? | Together |
| 4 | Hosting platform? | Together |
| 5 | Monetization later (ads / pro tools)? | Lio |

---

## 10. Phases

- **Phase 1 — Planning (now):** PRD ✔ → finalize tool list, stack, hosting, name
- **Phase 2 — Design:** Figma designs for homepage + tool pages
- **Phase 3 — Build:** core tools, one by one
- **Phase 4 — Backend:** Supabase setup + server-side rate limiting
- **Phase 5 — Polish & Launch:** performance, SEO, QA on mobile

---

*This is a living document — updated as decisions are made.*
