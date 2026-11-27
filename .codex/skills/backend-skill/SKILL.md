---
name: telegram-bot-skill
description: Builds and maintains a Telegram bot in Rust (teloxide, tokio, reqwest, serde, sqlx/PostgreSQL, Redis) that sells access via Telegram Stars and provisions users through the Remnawave API, deployed with podman compose on Ubuntu 24. Enforces KISS + YAGNI + SOLID, idempotent update and payment handling, safe migrations and lean verification. Use whenever the user works on this bot or any Telegram bot, mentions teloxide, BotFather, Telegram Stars/XTR, invoices, pre_checkout, Remnawave, VPN subscription bot, callback keyboards, dialogue state, Redis, podman compose or Containerfile, even if they do not say "Telegram bot" explicitly.
---

# Telegram Bot Skill: Rust / teloxide / Stars / Remnawave

## Stack (fixed, do not swap)
Rust, tokio, teloxide, reqwest, serde, sqlx + PostgreSQL, Redis, Telegram Stars (`XTR`), Remnawave API, podman compose, Ubuntu 24. Add a dependency only when a real feature needs it; check the current crate version instead of guessing, update `Cargo.lock`, note substantive choices in `project/DOCS.md` if it exists. No web framework by default: webhook (only if needed) uses teloxide's built-in listener behind a reverse proxy. Default to **long polling**.

## Principles (applied, not decorative)
- **KISS:** one binary, one crate until it hurts, plain functions and enums before traits and generics, no macros or abstractions to save five lines.
- **YAGNI:** build only the requested slice. No admin web UI, plugin system, event bus, microservices, config framework, or "future" tables/fields/modules. No empty scaffolding.
- **SOLID, pragmatically:** single responsibility per module; handlers depend on services, services depend on small traits **only at external boundaries** (Remnawave panel, Telegram sender if tests need it, clock). Create a trait when there is a second implementation or a real test need, never before. Open for extension via enums/match, not inheritance-style hierarchies.

## Layout
- `main.rs`: bootstrap, graceful shutdown. `config.rs`: sole env owner, fail fast on missing vars.
- `bot/`: `dispatcher.rs` (handler tree, error handler), `commands.rs`, `handlers/<feature>.rs` (thin adapters), `keyboards.rs`, `callbacks.rs` (typed, ≤64-byte callback data), `dialogue.rs` (state enum).
- `services/`: use cases (`billing`, `subscription`, ...): plain Rust, no teloxide types in signatures, transactions live here.
- `db/`: sqlx repositories, `models.rs` rows. `remnawave/`: `client.rs` (reqwest) + `dto.rs` (serde). `domain.rs`: pure rules (plans, prices, expiry math).
- Handlers never contain SQL, HTTP calls or business rules. Split a file at ~400 lines when meaningful. No global `utils.rs` dumps.

## Data ownership
- **PostgreSQL is the source of truth:** users, payments, subscriptions, provisioning state. Money/entitlement state is never only in Redis.
- **Redis is disposable:** dialogue state (teloxide Redis storage), update dedupe (`SET NX EX`), anti-flood/rate counters, short locks. The bot must stay correct if Redis is wiped. Always set TTLs.
- Ids: `BIGINT` for every Telegram id; identify users by `telegram_user_id`, never username.

## Update handling
- Dedupe by `update_id` (Redis TTL) before side effects; real idempotency for money lives in Postgres constraints.
- Always `answerCallbackQuery`, even on errors. Callback data: short codes + ids only.
- Throttle outgoing calls (use teloxide throttle adapter), honor `retry_after` on 429. Text ≤4096, caption ≤1024. Prefer HTML parse mode and escape all user text.
- `403 Forbidden`: mark user inactive. Handler errors never crash the dispatcher: log, friendly reply, continue.
- Set `allowed_updates` to only what the bot uses (`message`, `callback_query`, `pre_checkout_query` at minimum for payments).
- Webhook (if chosen): secret token header checked, random URL path (never the bot token). Never run polling and webhook on the same token.

## Telegram Stars
- Invoice: currency `XTR`, empty provider token, a single price item, payload ≤128 bytes holding an internal order id (never secrets or prices to trust).
- `pre_checkout_query`: validate order exists, unpaid, amount matches **server-side**, answer within the time limit.
- `successful_payment`: insert payment with `UNIQUE(telegram_payment_charge_id)` in the same transaction that marks the order paid; replays become no-ops. Store the charge id (needed for refunds via `refundStarPayment`).
- Never grant access on user claims or callbacks, only after a confirmed `successful_payment`. Provide a refund/support path.

## Remnawave integration
- **Do not guess endpoints or fields.** Read the current Remnawave API docs/OpenAPI of the target panel version (web search or ask for the version) before writing the client; record the version assumed in docs.
- `remnawave/client.rs`: one `reqwest::Client` reused, base URL + API token from env, connect/request timeouts, bounded retries with backoff **only for idempotent calls**, typed serde DTOs (tolerate unknown fields), errors mapped to a small domain error enum. Never log the token or full responses with user links.
- Provisioning is a state machine in Postgres: `paid → provisioning → active | failed`. Make create/extend idempotent (look up the panel user by a stable username derived from `telegram_user_id` before creating). A background reconcile pass retries `provisioning/failed`; the user is never charged without eventual delivery or a refund path.
- Subscription links are sensitive: send only to their owner in private chat, do not log them.

## Migrations (mandatory)
1. Inspect existing schema; add a **new numbered SQL migration** in `migrations/`, never edit an applied one.
2. Prefer expand/backfill/contract so old and new bot versions can coexist. Constraints and indexes only for real queries (`UNIQUE` charge id, user id, status).
3. Apply automatically only to disposable dev/test DB. **Production migrations run through the user-approved deployment step** (e.g. a one-shot `migrate` compose service), not in agent sessions. Never drop/truncate/reset production data without explicit approval; no `sqlx migrate revert` on real data.
4. Verify with one targeted check when the schema changes. Keep offline query metadata (`.sqlx`) consistent if used.

## Containers: podman compose on Ubuntu 24
- Multi-stage `Containerfile` (build → slim runtime), non-root user, no secrets baked into images, pinned base image tags.
- `compose.yaml`: services `bot`, `postgres`, `redis`; healthchecks, `restart: unless-stopped`, named volumes for Postgres, internal network, **do not publish Postgres/Redis ports**. Secrets via `env_file` outside git (or podman secrets), never inline values. Redis: password set, persistence optional since it is disposable.
- Write for **rootless podman** (no docker.sock, no privileged, no assumptions about Docker-only features); if systemd autostart is requested use Quadlet/systemd user units, otherwise skip (YAGNI).
- Provide `.env.example` with names only: `BOT_TOKEN`, `DATABASE_URL`, `REDIS_URL`, `REMNAWAVE_URL`, `REMNAWAVE_TOKEN`, `ADMIN_IDS` (+ webhook vars only if used).

## Security
Bot token and panel token only from env; never in code, logs, errors or chat. Don't log full updates/message text (PII): log update id, user id, handler, outcome. Validate and bound all input, parameterized SQL, timeouts on every outbound call, no blocking calls in async code. No `unwrap`/`expect` on update or payment paths. Admin commands authorized server-side from `ADMIN_IDS`/DB role on every call. If a token leaks, say it must be revoked in BotFather / the panel.

## Workflow and checks
Inspect touched code → smallest vertical slice (migration → repository → service → handler/keyboard) → check `git diff` → verify. `cargo fmt`, targeted `cargo check`/Clippy, and a focused test **only when relevant**: pricing/expiry rules, callback parsing, payment idempotency, provisioning state transitions, authorization. Test services with a fake Remnawave trait impl, no live token or panel needed. Do not run all-target Clippy + full tests after every change; no coverage inflation. Use `testing-skill` LIGHT/STANDARD/DEEP if present.

MUST NOT: secrets/tokens/subscription links in logs or chat, SQL or HTTP calls in handlers, granting access before confirmed Stars payment, non-idempotent payment or provisioning, Redis as sole source of truth, guessing Remnawave endpoints, traits/generics/layers without a real need, features nobody asked for, published DB/Redis ports, destructive production migrations, irrelevant refactors, unapproved external deployment.