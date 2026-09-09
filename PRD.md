# MailTool Pro — Product Requirements Document (PRD)

**Version:** 1.1 (Draft)  
**Date:** September 9, 2026  
**Status:** Planning  
**Working title:** MailTool Pro (final product name is not confirmed)

---

## 1. Product Vision

MailTool Pro will be a focused website containing **only mail-related tools**. It will have no user accounts, no login, and no unrelated image, PDF, or general-purpose tools.

> Open the website, choose a mail tool, enter the required information, and get a useful result immediately.

The complete product will share one distinctive visual language: unique styling, typography, components, motion, spacing, and tool interactions. It must not feel like a generic tools template or a copy of nexitool-pro.

---

## 2. Core Principles

| Principle | Product decision |
|---|---|
| Focus | Only mail-related tools |
| No login | No account, signup, or user profile |
| Consistent identity | Every page uses the same unique design system and font direction |
| Fast | Lightweight, responsive, and optimized for strong Core Web Vitals |
| Mobile-first | Every tool must be easy to use on a phone |
| Secure AI access | AI keys and custom prompts remain on the server |
| Configurable backend | Prompts, models, and limits can be changed without editing frontend code |
| Abuse protection | Every server-side tool has a configurable request limit |
| Privacy-first | Collect and retain only the minimum technical data required for security and operation |

---

## 3. Scope

### In scope

- Mail tools already built in nexitool-pro, moved or rebuilt in this dedicated repository
- New mail tools added over time
- AI-powered mail tools, such as a mail writer, using one shared AI provider when appropriate
- Supabase database and Edge Functions
- Server-side custom prompt selection
- Per-tool, per-client rate limiting
- Secure provider-key management
- Domain/origin restrictions and abuse protection
- Unique responsive design created in Figma and implemented consistently
- SEO-friendly pages for individual tools

### Out of scope for the first version

- Login, signup, or user accounts
- Image, PDF, or unrelated general-purpose tools
- Payments or subscriptions
- Native mobile applications

---

## 4. AI Tool Architecture

### 4.1 Central idea

The frontend must not contain AI provider keys or full custom prompts. Supabase will be the control layer for every AI-powered mail tool.

Each AI tool will have a stable public identifier such as:

- `mail_writer`
- `reply_generator`
- `subject_generator`
- `tone_rewriter`

When a user uses Mail Writer, the frontend sends the `tool_id` plus validated user input to a Supabase Edge Function. The backend uses that ID to load the correct private prompt, model settings, and rate-limit rules. It then calls the configured AI provider, such as Gemini, and returns only the final result.

The same Gemini API integration can therefore serve many tools while each tool behaves differently because its backend configuration and custom prompt are different.

### 4.2 Request flow

1. User opens a tool page and submits input.
2. Frontend sends an HTTPS request containing:
   - public `tool_id`
   - tool-specific user inputs
   - anti-abuse token when enabled
3. Supabase Edge Function validates:
   - request origin/domain
   - HTTP method and content type
   - payload schema and size
   - whether the tool is active
4. Backend finds the tool configuration using `tool_id`.
5. Backend checks that client's limit for that specific tool.
6. If allowed, backend combines the private system prompt with sanitized user input.
7. Backend calls the configured AI provider using a server-only API key.
8. Backend returns the structured result to the frontend.
9. If the limit is exceeded, backend returns HTTP `429` with a friendly retry time.

### 4.3 Initial tool-configuration table

Suggested table: `ai_tool_configs`

| Field | Purpose |
|---|---|
| `id` | Internal UUID primary key |
| `public_id` | Stable unique ID sent by frontend, e.g. `mail_writer` |
| `name` | Human-readable tool name |
| `provider` | AI provider, e.g. `gemini` |
| `model` | Provider model used by this tool |
| `system_prompt` | Private custom prompt used only by backend |
| `prompt_version` | Prompt version for safe updates and debugging |
| `requests_per_minute` | Configurable per-client limit, e.g. 3, 4, or 5 |
| `requests_per_day` | Optional daily safety limit |
| `max_input_chars` | Maximum accepted input size |
| `max_output_tokens` | Maximum generated response size |
| `temperature` | Tool-specific model setting |
| `is_active` | Enables or disables the tool without a deployment |
| `created_at` / `updated_at` | Audit timestamps |

The frontend may know `public_id`, tool name, and public UI information. It must never be able to read `system_prompt`, provider credentials, or internal security settings directly.

### 4.4 Supporting backend data

- `rate_limit_buckets` — atomic counters by tool and pseudonymous client key
- `usage_events` — minimal operational events without storing mail content by default
- `allowed_origins` or secure environment configuration — production and approved preview domains
- Optional prompt history table for rollback and controlled prompt changes

---

## 5. Rate Limiting Without Login

Because there are no accounts, the backend cannot identify an exact human user. “Per-user” limiting will therefore mean a privacy-conscious approximation based on a short-lived pseudonymous client key, such as a salted hash derived from IP and limited request context. A browser identifier may be added only if required and disclosed appropriately.

### Required behavior

- Limit is configured separately for every tool in `ai_tool_configs`.
- Example: Mail Writer can allow 3 requests/minute while Subject Generator allows 5.
- Counter updates must be atomic and enforced by the backend, never by frontend JavaScript.
- Raw IP addresses should not be retained longer than required; prefer salted hashes with expiry.
- Return `429 Too Many Requests` and `Retry-After` when blocked.
- Show a clear frontend message instead of a broken tool.
- Add payload-size limits, timeouts, and provider-budget safeguards.
- Consider Cloudflare Turnstile or a similar challenge if automated abuse appears.

Supabase Postgres plus an atomic database function can support the initial fixed/sliding-window limiter. A dedicated edge rate-limiting service can be considered later if traffic becomes very high.

---

## 6. Frontend-to-Backend Security

### Important rule

A permanent secret placed in browser code is **not secret**. Anyone can inspect it and reuse it. Therefore, domain + a permanent frontend secret alone will not be treated as secure authentication.

### Approved security model

- AI provider key is stored only in **Supabase project secrets**.
- Supabase service-role key is never included in frontend code.
- Custom prompts are readable only by trusted backend code; Row Level Security denies direct public access.
- Frontend calls a narrowly scoped Edge Function endpoint.
- Edge Function uses an exact production-origin allowlist and strict CORS rules.
- Origin/domain checks reduce casual misuse but are not the only protection because headers can be spoofed outside a browser.
- Server-side rate limits, payload validation, budget caps, and optional Turnstile provide the real abuse protection.
- If stronger request authentication is later required, use a short-lived signed token issued by a trusted server—not a permanent browser secret.
- The Supabase anon key may exist in the frontend because it is public by design, but it grants only tightly restricted access under RLS and is not the AI secret.

---

## 7. Design Direction

- One unique visual identity across the homepage and every tool
- Original typography direction with properly licensed web fonts
- Consistent design tokens for colors, type scale, spacing, radii, shadows, borders, and motion
- Mobile-first forms, touch-friendly controls, and strong visual hierarchy
- Fast interactions with helpful loading, success, empty, error, and rate-limit states
- Figma Pro used for the design system, responsive screens, components, and developer handoff
- Final product name and logo remain open decisions; repo name is temporary

---

## 8. Performance, Accessibility, and SEO

- Target Lighthouse scores of 90+ for Performance, Accessibility, Best Practices, and SEO
- Optimize Core Web Vitals and avoid unnecessary client-side JavaScript
- Semantic HTML, keyboard navigation, visible focus states, and WCAG-friendly contrast
- Every tool gets a dedicated indexable URL, title, description, canonical URL, and social metadata
- Do not expose private prompts, secrets, or internal error traces in browser responses
- Mail content should not be saved by default; any future retention feature requires an explicit product decision

---

## 9. Open Decisions

1. Final website/product name
2. Exact list of existing mail tools to migrate first
3. Frontend framework and hosting platform
4. Gemini model and expected monthly AI budget
5. Exact per-minute and daily limits for each tool
6. Whether Turnstile is enabled at launch or only after abuse appears
7. Data-retention duration for pseudonymous rate-limit records and operational logs

---

## 10. Delivery Phases

1. **Planning:** finalize tools, architecture, security rules, stack, and product name
2. **Audit:** inspect nexitool-pro and map every existing mail tool and dependency
3. **Design system:** create the unique Figma direction, tokens, components, and responsive screens
4. **Foundation:** initialize frontend, Supabase schema, RLS, Edge Functions, and environments
5. **AI gateway:** implement tool-ID routing, private prompt lookup, Gemini integration, validation, and structured responses
6. **Rate limiting:** implement atomic per-tool counters, friendly `429` states, and abuse controls
7. **Tool migration:** port and test mail tools in priority order
8. **Quality and launch:** mobile QA, accessibility, security review, performance, SEO, and deployment

---

*This is a living document and will be updated as product decisions are confirmed.*
