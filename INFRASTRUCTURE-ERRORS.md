# Infrastructure Errors & Issues

## CRITICAL: Stripe Dashboard Completely Empty (STOP-001)

- $0 balance
- 0 products
- 0 customers
- 0 payment intents
- 0 subscriptions
- Backend webhook handlers and Stripe service are fully wired, but the Stripe dashboard has zero configuration.

**Impact**: No payments can be processed. The entire escrow/payment flow is dead on arrival.
**Fix**: Configure Stripe products, prices, webhook endpoints, and test with live-mode test keys.

## REVISED: Database Provider (STOP-003)

- **Neon PostgreSQL**: ACTIVE production database. 103 tables, PostGIS enabled, Railway-hosted.
- **Supabase "aura-ventures"**: RESTORED to ACTIVE_HEALTHY as of 2026-04-02. Project ref: `xptvjwceoknmfringzju`, region: `us-west-1`. pgvector enabled, `doc_embeddings` table created for knowledge graph indexer. Now used as the vector store for HUSTLEXP-DOCS indexing pipeline.

**Action**: Supabase is confirmed NOT legacy. It serves as the vector embedding store for the knowledge graph indexer. Do NOT remove Supabase references.

## RESOLVED: Backend CI/CD (STOP-002)

4 GitHub Actions workflows confirmed active: ci.yml, deploy-aws.yml, deploy.yml, security.yml.

## NEW: Knowledge Graph Indexer Blocked — No Embedding Provider (STOP-004)

**Added**: 2026-04-02
**Repos**: HUSTLEXP-DOCS, hustlexp-ai-backend
**Severity**: HIGH — blocks knowledge graph feature entirely

**Problem**: The HUSTLEXP-DOCS knowledge graph indexer (`scripts/index-docs.ts`) requires an embeddings API to generate vector embeddings for document chunks. The Kimi Moonshot API key provided does NOT support embeddings — Moonshot only offers chat completion models (kimi-k2.5, moonshot-v1-*). There is no `/v1/embeddings` endpoint on `api.moonshot.cn`.

**What IS wired up (completed 2026-04-02)**:
- Supabase restored, pgvector enabled, `doc_embeddings` table created with ivfflat index
- `index-docs.ts` patched to accept `OPENAI_BASE_URL` and `EMBEDDING_MODEL` env vars (commit `7c481b6`)
- `index-docs.yml` workflow has `INDEXING_ENABLED` guard (commit `43fe9d4`)
- `CROSS_REPO_TOKEN` secret set on HUSTLEXP-DOCS

**What is still needed**:
- An embedding-capable API key (one of the following):
  - **Option A**: OpenAI API key (text-embedding-3-small, ~$0.02/M tokens — cheapest)
  - **Option B**: Patch `index-docs.ts` for Hugging Face Inference API (free tier, `BAAI/bge-small-en-v1.5`)
  - **Option C**: Any OpenAI-compatible provider that supports `/v1/embeddings`
- Set remaining GitHub secrets on HUSTLEXP-DOCS: `DATABASE_URL`, `OPENAI_API_KEY`, `OPENAI_BASE_URL`, `EMBEDDING_MODEL`
- Set `INDEXING_ENABLED=true` repo variable on HUSTLEXP-DOCS
- DATABASE_URL requires Supabase database password (not yet retrieved)

**Impact**: Knowledge graph search, semantic doc retrieval, and AI-assisted documentation features are all blocked until an embedding provider is configured.
**Fix**: Provide an OpenAI API key OR approve Hugging Face free-tier patch. Then set secrets and flip `INDEXING_ENABLED=true`.

---

## CRITICAL: Stripe Transfer Missing Idempotency Key — Double-Payout Risk (STOP-005)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/services/StripeService.ts` — `transfers.create()` call
**Severity**: CRITICAL — real money loss

**Problem**: `StripeService.createTransfer()` does NOT pass an `idempotencyKey` to `stripe.transfers.create()`. If a network timeout occurs and the caller retries, Stripe will create a **second transfer**, sending real money twice. Same issue exists on `stripe.refunds.create()` — double-refund risk.

**Impact**: Every escrow release or refund is vulnerable to double-payout on retry/timeout. One incident = direct financial loss.
**Fix**: Add `idempotencyKey: \`transfer_${escrowId}_${taskId}\`` to every `transfers.create()` and `idempotencyKey: \`refund_${paymentIntentId}\`` to every `refunds.create()` call. Estimated effort: 2 hours.

## CRITICAL: Post-Commit Side-Effect Failures in EscrowService (STOP-006)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/services/EscrowService.ts` — `releaseEscrow()` method
**Severity**: CRITICAL — silent revenue/XP loss

**Problem**: After the DB transaction commits (escrow marked RELEASED), `RevenueService.recordRevenue()`, `SelfInsurancePool.contribute()`, and `XPService.awardXP()` are called **outside** the transaction. If any of these fail, the escrow is already released but the platform fee, insurance contribution, or XP grant is silently lost. The try/catch swallows errors.

**Impact**: Revenue leakage, insurance pool underfunding, missing XP awards — all invisible until manual audit.
**Fix**: Move critical side effects into the transaction OR implement a transactional outbox pattern with retry. Estimated effort: 4-6 hours.

## CRITICAL: SelfInsurancePool Bypasses StripeService (STOP-007)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/services/SelfInsurancePoolService.ts` — `payClaim()` method
**Severity**: CRITICAL — bypasses all financial safeguards

**Problem**: `payClaim()` makes a direct `fetch()` call to Stripe's REST API (`https://api.stripe.com/v1/transfers`), completely bypassing `StripeService`, the circuit breaker, idempotency logic, the `HX_STRIPE_STUB` guard, and all logging/audit infrastructure.

**Impact**: Insurance claim payouts have zero idempotency protection, zero circuit breaker protection, and zero audit trail through the standard financial pipeline. Double-claim payouts possible.
**Fix**: Refactor to use `StripeService.createTransfer()` (after that service gets idempotency keys per STOP-005). Estimated effort: 2 hours.

## HIGH: Webhook Idempotency TOCTOU Race (STOP-008)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/services/StripeService.ts` — `processWebhookEvent()` method
**Severity**: HIGH — duplicate webhook processing

**Problem**: Webhook deduplication does SELECT (check if event exists) → run handler → INSERT (mark event processed), but these are NOT in the same transaction. Two concurrent deliveries of the same webhook can both pass the SELECT check before either inserts, leading to double-processing.

**Impact**: Duplicate escrow releases, duplicate XP grants, duplicate refunds on concurrent webhook delivery.
**Fix**: Wrap in a transaction with `INSERT ... ON CONFLICT DO NOTHING` returning whether the insert succeeded, THEN process only if insert won. Estimated effort: 2 hours.

## HIGH: HX_STRIPE_STUB Env Var Bypasses All Stripe Calls (STOP-009)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/services/StripeService.ts`
**Severity**: HIGH — if set in production, all payments silently no-op

**Problem**: When `HX_STRIPE_STUB === '1'`, every Stripe call returns a fake success response. This is intended for testing but there is no guard ensuring it cannot be set in production. If deployed with this env var, the platform will accept tasks, "fund" escrows, and "release" payments — all as no-ops — while users believe real money is moving.

**Impact**: Complete payment system bypass in production if misconfigured.
**Fix**: Add startup assertion: `if (process.env.NODE_ENV === 'production' && process.env.HX_STRIPE_STUB === '1') throw new Error('FATAL: HX_STRIPE_STUB cannot be enabled in production')`. Estimated effort: 15 minutes.

## HIGH: Referral Rewards Never Actually Paid (STOP-010)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/routers/referral.ts` — `issueReferralReward()` function
**Severity**: HIGH — trust/legal issue

**Problem**: `issueReferralReward()` sets `referrer_reward_paid = TRUE` in the database but **never creates a Stripe transfer** to actually send the $5 reward. The referrer sees "reward paid" but receives nothing.

**Impact**: Users are told they earned referral rewards but never receive money. Trust destruction + potential legal liability for false advertising.
**Fix**: Add `StripeService.createTransfer()` call with idempotency key before marking paid. Estimated effort: 2 hours.

## HIGH: XP Velocity Check Fails Open (STOP-011)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/services/XPService.ts` — `checkVelocity()` method
**Severity**: HIGH — fraud detection bypass

**Problem**: If the velocity check query throws an error (Redis down, DB timeout), the catch block returns `{ suspicious: false }` — meaning the system assumes "not suspicious" when it actually has no data. An attacker could deliberately trigger errors to bypass fraud detection.

**Impact**: XP farming attacks succeed whenever Redis or the velocity check DB query is under load.
**Fix**: Fail closed: return `{ suspicious: true, reason: 'velocity_check_unavailable' }` on error. Estimated effort: 30 minutes.

## HIGH: ASAP Price Bump / Escrow Mismatch (STOP-012)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/services/DynamicPricingService.ts` — `bumpASAPPrice()` method
**Severity**: HIGH — financial inconsistency

**Problem**: `bumpASAPPrice()` updates the task's displayed price (+$3 per bump, up to 3 bumps = +$9) but the escrow amount stays at the original funded value. The worker sees a $59 task but the escrow only holds $50.

**Impact**: Worker expects $59 payout but escrow can only release $50. $9 shortfall per ASAP-bumped task.
**Fix**: Either bump escrow amount in same transaction, or collect ASAP surcharge as separate payment intent. Estimated effort: 3-4 hours.

## MEDIUM: BackgroundCheckService Never Calls Checkr API (INFO-001)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/services/BackgroundCheckService.ts` — `initiateBackgroundCheck()` method
**Severity**: MEDIUM — feature is stub

**Problem**: `initiateBackgroundCheck()` generates a fake background check ID and never calls the Checkr/Sterling/GoodHire API. The webhook handler exists but no webhook route is registered. Background check status is always manually set.

**Impact**: Background check feature is non-functional. Tasks >$500 are gated by a check that can never actually complete through normal flow.
**Fix**: Integrate Checkr API (sandbox first), register webhook route, implement earned verification unlock payment collection. Estimated effort: 8-12 hours.

## MEDIUM: No Sybil / Mass Account Creation Defense (INFO-002)

**Added**: 2026-04-02
**Repo**: hustlexp-ai-backend
**File**: `backend/src/trpc.ts` — `ensureUserRowForFirebaseUid()` function
**Severity**: MEDIUM — abuse vector

**Problem**: Firebase free-tier allows unlimited account creation. `ensureUserRowForFirebaseUid()` auto-provisions a DB row for every new Firebase UID with no rate limiting, no device fingerprinting, and no phone verification requirement. `FraudDetectionService` pattern types are defined but detection logic is stub/TBD.

**Impact**: Attacker can create hundreds of accounts to farm referral bonuses ($5 × 20 cap × N accounts), manipulate ratings, or collude on tasks.
**Fix**: Phase 1: Rate-limit account creation per IP/device. Phase 2: Require phone verification. Phase 3: Implement graph-based collusion detection. Estimated effort: 6-10 hours total.
