# MailTool Pro — Product Requirements Document (PRD)

**Version:** 1.2 (Draft)  
**Date:** September 9, 2026  
**Status:** Planning  
**Working title:** MailTool Pro (final product name is not confirmed)

---

## 1. Product Vision

MailTool Pro will be a focused platform containing only mail-related tools. It will support two intentional access paths:

1. **Open access:** basic tools work immediately without an account.
2. **Account access:** external login unlocks advanced tools, usage credits/tokens, and eligible private history.

> Start without friction. Create an account only when deeper capability becomes useful.

The complete product will share one distinctive visual language: unique styling, typography, components, motion, spacing, and tool interactions. It must not feel like a generic tools template or a copy of nexitool-pro.

---

## 2. Core Principles

| Principle | Product decision |
|---|---|
| Focus | Only mail-related tools |
| No forced login | Basic tools remain available to guests |
| Progressive access | Accounts unlock advanced tools, history, and usage credits |
| External authentication | Login and account creation occur through a separate account service |
| Consistent identity | Every page uses the same unique design system and typography direction |
| Multi-page structure | Homepage, tool directory, individual tools, privacy, and guidance are separate pages |
| Secure AI access | Provider keys and protected tool instructions remain server-side |
| Provider-neutral interface | Public product copy describes task-specific AI rather than advertising the underlying vendor |
| Configurable backend | Prompts, models, access levels, token costs, and limits change without frontend deployment |
| Privacy and honesty | Disclosures accurately explain third-party processing without misleading ownership claims |

---

## 3. Access Model

### 3.1 Guest access

- No account required
- Basic tools available immediately
- Per-tool guest request limits enforced server-side
- No saved account history
- Short-lived pseudonymous security metadata may be used for rate limiting
- Clear upgrade path when an advanced feature is selected

### 3.2 Account access

- Login and account creation handled by an external identity/account portal
- Advanced tools become available after verified authentication
- User receives a visible usage-credit balance, called **tokens** in the product UI
- Each advanced tool may consume a configurable number of tokens
- Eligible results can appear in private history
- Account-level limits and entitlements can differ from guest limits

### 3.3 Important token distinction

Two different concepts must not be mixed:

- **Access token:** short-lived signed credential returned by the identity system and verified by the backend. It is never displayed as spendable currency.
- **Usage tokens/credits:** product balance shown to the user and deducted when eligible advanced tools are used.

The frontend must never be trusted to declare its own balance, deduction, identity, or entitlement. All checks and deductions happen atomically on the backend.

---

## 4. Information Architecture

The production website will not place every subject on one long homepage.

| Route/page | Purpose |
|---|---|
| Home | Short positioning, featured tools, and account-value overview |
| Tools | Complete mail-tool directory with open/account access labels |
| Individual tool pages | One focused task and interface per URL |
| How it works | Plain-language explanation of the experience |
| Privacy | Accurate processing, account, retention, and third-party disclosures |
| External login | Separate account portal owned by the authentication system |
| External create account | Separate account-registration flow |
| Account/history | Signed-in area for token balance, advanced access, and eligible history |

Every tool must have a dedicated indexable URL rather than opening all tools on the homepage.

---

## 5. Scope

### In scope

- Existing mail tools moved or rebuilt from nexitool-pro
- New mail tools added over time
- Open and account-only tool access levels
- External login and account creation
- Usage-token balance, pricing rules, atomic deduction, and ledger
- Private history for eligible signed-in requests
- Supabase database and Edge Functions
- Server-side custom prompt/tool-instruction selection
- Per-tool guest and account request limits
- Secure provider-key management
- Domain/origin restrictions and abuse protection
- Unique responsive design created in Figma and implemented consistently
- SEO-friendly individual tool pages

### Out of scope for the first version

- Image, PDF, or unrelated general-purpose tools
- Native mobile applications
- Storing guest mail content as history
- Handling user passwords directly inside the mail-tools frontend

---

## 6. AI Tool Architecture

### 6.1 Central idea

The frontend contains neither provider credentials nor full protected tool instructions. Supabase acts as the control layer for every AI-powered mail tool.

Each AI tool has a stable public identifier such as:

- `mail_writer`
- `reply_generator`
- `subject_generator`
- `tone_rewriter`

The frontend sends `tool_id` plus validated user input to a Supabase Edge Function. The backend loads the correct protected instructions, model configuration, access tier, token cost, and request-limit rules. It calls the configured AI service using server-only credentials and returns only the structured result.

### 6.2 Request flow

1. User opens an individual tool page and submits input.
2. Frontend sends an HTTPS request containing tool ID, required input, anti-abuse proof when enabled, and an optional signed access credential.
3. Edge Function validates origin, method, content type, schema, payload size, and tool status.
4. If an access credential exists, the backend verifies identity and entitlements.
5. Backend loads the tool configuration.
6. Backend enforces either guest or account rate limits.
7. For account-only tools, backend confirms balance and reserves/deducts the configured usage tokens atomically.
8. Backend combines protected instructions with sanitized input and invokes the configured AI service.
9. Backend returns a structured result. Eligible signed-in requests may be written to private history under retention rules.
10. Failures return clear error codes; deductions must be reversed when an eligible provider failure prevents delivery.

### 6.3 Tool-configuration table

Suggested table: `ai_tool_configs`

| Field | Purpose |
|---|---|
| `id` | Internal UUID primary key |
| `public_id` | Stable frontend tool identifier |
| `name` | Human-readable tool name |
| `provider` | Internal provider routing value; never required in public UI copy |
| `model` | Internal model configuration |
| `system_prompt` | Protected tool instructions readable only by trusted backend code |
| `prompt_version` | Version for controlled changes and rollback |
| `access_tier` | `guest`, `account`, or another future entitlement |
| `token_cost` | Usage-credit cost for an eligible account request |
| `guest_requests_per_minute` | Guest limit for this tool |
| `account_requests_per_minute` | Signed-in limit for this tool |
| `requests_per_day` | Optional daily safety limit |
| `max_input_chars` | Maximum accepted input size |
| `max_output_tokens` | Maximum generated response size |
| `temperature` | Tool-specific generation setting |
| `save_history` | Whether eligible signed-in results may be stored |
| `is_active` | Enables/disables the tool without deployment |
| `created_at` / `updated_at` | Audit timestamps |

### 6.4 Supporting data

- `user_profiles` — external identity mapping and product status, without passwords
- `token_wallets` — current usage-credit balance
- `token_ledger` — immutable grants, deductions, reversals, and references
- `tool_entitlements` — optional advanced access overrides
- `request_history` — eligible signed-in outputs under retention rules
- `rate_limit_buckets` — atomic counters by tool and client/account key
- `usage_events` — minimal operational events without storing guest mail content by default
- `allowed_origins` — approved production and preview domains
- Optional prompt history table for rollback

---

## 7. External Authentication and Security

- The mail-tools frontend redirects login and registration to an external account portal.
- The identity system returns a short-lived signed access credential through a secure callback flow.
- The backend verifies signature, issuer, audience, expiry, and subject before granting account access.
- Refresh credentials use secure, HTTP-only cookie handling where supported and must not be exposed to frontend JavaScript.
- This site never receives or stores the user's password.
- AI provider keys and Supabase service-role credentials remain in server-side secrets.
- Row Level Security prevents direct public reads of protected prompts, wallets, ledger entries, or another user's history.
- Usage-token deductions and advanced-tool authorization occur in one atomic backend operation.
- Domain/origin allowlists and CORS reduce browser misuse but do not replace authentication, validation, rate limits, and anti-bot controls.

---

## 8. Rate Limiting

### Guest requests

- Enforced per tool using a short-lived pseudonymous client key
- Raw IP addresses should not be retained longer than required
- Return HTTP `429` and `Retry-After` when blocked
- Show a clear frontend message

### Account requests

- Enforced per verified account and tool
- Can use different limits from guest access
- Token balance is not a substitute for burst protection
- Add daily budget and provider-cost safeguards

Supabase Postgres plus atomic database functions can support the initial limiter and token ledger. A dedicated edge-limiting service can be considered if traffic becomes very high.

---

## 9. Public AI Wording and Transparency

### Product interface wording

Recommended wording:

> “Your submitted information is sent securely to an AI configured specifically for this task.”

Avoid vendor branding on marketing pages and avoid implying that every tool uses a generic chatbot.

### Accuracy requirement

- Do not claim the underlying model was built, trained, or owned by this product unless that is true.
- The formal Privacy Policy must accurately explain that a third-party AI service may process submitted information.
- Applicable processors should be identified wherever law, contract, platform rules, or user expectations require it.
- Internal technical configuration may route providers without changing general product UI copy.

---

## 10. Design Direction

- Preserve the approved editorial-utility visual concept
- One shared design system across separate pages
- Original typography direction with properly licensed web fonts
- Consistent colors, type scale, spacing, borders, motion, and interaction states
- Mobile-first forms and minimum 44px interactive targets
- Clear visual distinction between open and account-only tools
- Visible token balance and token cost before advanced requests
- Account prompts should feel optional and useful, never like a forced wall around basic tools
- Figma Pro used for design system, responsive screens, components, and developer handoff

---

## 11. Performance, Accessibility, and SEO

- Target Lighthouse scores of 90+ across core categories
- Optimize Core Web Vitals and avoid unnecessary client-side JavaScript
- Semantic HTML, keyboard navigation, focus states, and WCAG-friendly contrast
- Dedicated title, description, canonical URL, and social metadata for each tool
- Never expose protected prompts, secrets, raw provider errors, or internal traces
- Guest mail content is not saved as history by default

---

## 12. Open Decisions

1. Final website/product name
2. External account/identity service and final login URLs
3. Which existing tools are open versus account-only
4. Initial token grant and token cost per advanced tool
5. History retention duration and deletion controls
6. Frontend framework and hosting platform
7. Provider budget and account/guest request limits
8. Whether anti-bot challenges are enabled at launch

---

## 13. Delivery Phases

1. Finalize access tiers, token economics, identity provider, stack, and name
2. Audit and classify existing mail tools
3. Complete Figma design system and separate page designs
4. Initialize frontend, Supabase schema, RLS, Edge Functions, and environments
5. Integrate external authentication and secure callback verification
6. Implement tool-ID routing, protected instruction lookup, and structured outputs
7. Implement atomic rate limits, token wallets, ledger, and reversals
8. Implement eligible private history and retention controls
9. Migrate tools in priority order
10. Complete mobile QA, accessibility, security review, performance, SEO, and launch

---

*This is a living document and will be updated as product decisions are confirmed.*
