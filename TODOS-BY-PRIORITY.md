# All TODOs — Prioritized

## P0 — Launch Blockers (Do First)

| ID | Task | Repo | Category |
|----|------|------|----------|
| TODO-001 | Configure Stripe dashboard (products, prices, webhooks, test keys) | Stripe / hustlexp-ai-backend | Infrastructure |
| TODO-002 | Replace placeholder SSL pin hashes with real SPKI hashes | HUSTLEXPFINAL1 | Security |
| TODO-003 | Remove C7 rehearsal failure injection code from client.ts | HUSTLEXPFINAL1 | Security |
| TODO-004 | Run `npm audit fix` on backend (5 HIGH vulnerabilities in hono, node-forge, fast-xml-parser) | hustlexp-ai-backend | Security |
| TODO-005 | Add frontend CI/CD pipeline (GitHub Actions: lint, typecheck, test, build) | HUSTLEXPFINAL1 | Infrastructure |

## P0.5 — Critical Financial Patches (Do Immediately After P0)

| ID | Task | Repo | Category |
|----|------|------|----------|
| TODO-030 | Add idempotency keys to `StripeService.createTransfer()` and `createRefund()` (STOP-005) | hustlexp-ai-backend | Financial Safety |
| TODO-031 | Move post-commit side effects (revenue, insurance, XP) into transaction or outbox (STOP-006) | hustlexp-ai-backend | Financial Safety |
| TODO-032 | Refactor `SelfInsurancePoolService.payClaim()` to use StripeService instead of direct fetch (STOP-007) | hustlexp-ai-backend | Financial Safety |
| TODO-033 | Fix webhook idempotency TOCTOU race — atomic INSERT ON CONFLICT pattern (STOP-008) | hustlexp-ai-backend | Financial Safety |
| TODO-034 | Add production guard against `HX_STRIPE_STUB=1` (STOP-009) | hustlexp-ai-backend | Financial Safety |
| TODO-035 | Implement actual Stripe transfer in `issueReferralReward()` (STOP-010) | hustlexp-ai-backend | Financial Safety |
| TODO-036 | Change XP velocity check to fail closed (`suspicious: true` on error) (STOP-011) | hustlexp-ai-backend | Financial Safety |
| TODO-037 | Fix ASAP price bump / escrow amount mismatch (STOP-012) | hustlexp-ai-backend | Financial Safety |

## P1 — Financial Safety (Week 1-2)

| ID | Task | Repo | Category |
|----|------|------|----------|
| TODO-006 | Re-verify ARCH-001 (DisputeService direct escrow UPDATE) against real backend | hustlexp-ai-backend | Architecture |
| TODO-007 | Fix 3 excluded test files that reference non-existent StripeMoneyEngine module | hustlexp-ai-backend | Testing |
| TODO-008 | Fix 2 excluded test files that reference non-existent TaxComplianceService | hustlexp-ai-backend | Testing |
| TODO-009 | Create the missing TODO.md file referenced by dispute.ts:16 | hustlexp-ai-backend | Documentation |
| TODO-010 | Raise vitest coverage thresholds from 10% to 70% to prevent regression | hustlexp-ai-backend | Testing |

## P2 — Code Quality (Week 2-3)

| ID | Task | Repo | Category |
|----|------|------|----------|
| TODO-011 | Migrate 43 console.log/warn/error calls to pino logger in backend | hustlexp-ai-backend | Observability |
| TODO-012 | Consolidate 20+ scattered process.env reads into config.ts | hustlexp-ai-backend | Architecture |
| TODO-013 | Fix sentry.ts release tag: `hustlexp-api` → `hustlexp-ai-backend` | hustlexp-ai-backend | Observability |
| TODO-014 | Add client-side input validation to all 14 TextInput fields | HUSTLEXPFINAL1 | Security |
| TODO-015 | Enable Firebase Crashlytics in frontend | HUSTLEXPFINAL1 | Observability |
| TODO-016 | Enable structured logging in frontend | HUSTLEXPFINAL1 | Observability |
| TODO-017 | Remove 39 `as any` type assertions from frontend | HUSTLEXPFINAL1 | Type Safety |
| TODO-026 | Obtain embedding-capable API key for knowledge graph indexer (OpenAI recommended, ~$0.02/M tokens) | HUSTLEXP-DOCS / hustlexp-ai-backend | Infrastructure |
| TODO-027 | Set remaining GitHub secrets on HUSTLEXP-DOCS: `DATABASE_URL`, `OPENAI_API_KEY`, `OPENAI_BASE_URL`, `EMBEDDING_MODEL` | HUSTLEXP-DOCS | Infrastructure |
| TODO-028 | Retrieve Supabase database password and construct `DATABASE_URL` connection string | Infrastructure | Infrastructure |
| TODO-029 | Set `INDEXING_ENABLED=true` repo variable on HUSTLEXP-DOCS and verify workflow triggers | HUSTLEXP-DOCS | Infrastructure |

## P2.5 — Anti-Abuse & Fraud Hardening (Week 2-3)

| ID | Task | Repo | Category |
|----|------|------|----------|
| TODO-038 | Rate-limit account creation per IP/device fingerprint (Sybil defense phase 1) (INFO-002) | hustlexp-ai-backend | Security |
| TODO-039 | Require phone verification for account activation (Sybil defense phase 2) | hustlexp-ai-backend | Security |
| TODO-040 | Implement graph-based collusion detection (FraudDetectionService stub → real logic) | hustlexp-ai-backend | Security |
| TODO-041 | Add streak timezone normalization (prevent UTC midnight arbitrage) | hustlexp-ai-backend | Gamification |
| TODO-042 | Add rejection counter to `TaskService.rejectProof()` — max 3 rejections then auto-escalate to dispute | hustlexp-ai-backend | Financial Safety |
| TODO-043 | Add `idle_in_transaction_session_timeout` to DB pool config (prevent hung transaction pool exhaustion) | hustlexp-ai-backend | Infrastructure |
| TODO-044 | Fix Redis/PG XP daily cap drift — DECRBY on transaction rollback | hustlexp-ai-backend | Financial Safety |

## P3 — Testing & Coverage (Week 3-4)

| ID | Task | Repo | Category |
|----|------|------|----------|
| TODO-018 | Fix 2 excluded test files with wrong db abstraction (sql vs db.query) | hustlexp-ai-backend | Testing |
| TODO-019 | Fix 1 excluded test file with wrong Redis mock | hustlexp-ai-backend | Testing |
| TODO-020 | Write E2E tests covering full payment flow (login → task → escrow → pay → XP) | HUSTLEXPFINAL1 | Testing |
| TODO-021 | Verify GoogleService-Info.plist is gitignored and not leaking credentials | HUSTLEXPFINAL1 | Security |

## P4 — Architecture Hardening (Week 4+)

| ID | Task | Repo | Category |
|----|------|------|----------|
| TODO-022 | Evaluate refactoring god services (ExpertiseSupplyService 1621L, TaskService 1487L, NotificationService 1430L) | hustlexp-ai-backend | Architecture |
| ~~TODO-023~~ | ~~Resolve Supabase status — confirm legacy, remove references if dead~~ **RESOLVED 2026-04-02**: Supabase restored to ACTIVE_HEALTHY, now serves as vector store for knowledge graph indexer | Infrastructure | Architecture |
| TODO-024 | Implement Phase 2 native SSL pinning (react-native-ssl-pinning) | HUSTLEXPFINAL1 | Security |
| TODO-025 | Replace 16 frontend console.log calls with structured logger | HUSTLEXPFINAL1 | Observability |
| TODO-045 | Integrate Checkr API (sandbox → production) for BackgroundCheckService (INFO-001) | hustlexp-ai-backend | Feature |
| TODO-046 | Register BackgroundCheck webhook route + implement status callback handler | hustlexp-ai-backend | Feature |
| TODO-047 | Implement earned verification unlock payment collection ($1 fee via Stripe) | hustlexp-ai-backend | Feature |
| TODO-048 | Build 5-tier verification ladder (email → phone → ID → background → expertise) | hustlexp-ai-backend | Feature |
| TODO-049 | Connect background check CLEAR status to trust tier promotion in TrustService | hustlexp-ai-backend | Architecture |
| TODO-050 | Add trust tier decay for inactive users (>90 days no completed tasks) | hustlexp-ai-backend | Architecture |
| TODO-051 | Implement dual-sided referral rewards ($5 referrer + $3 new user credit) | hustlexp-ai-backend | Growth |
| TODO-052 | Add social share deeplinks with OG meta tags for completed task celebrations | HUSTLEXPFINAL1 | Growth |
| TODO-053 | Build "Invite Contacts" flow with pre-populated referral codes | HUSTLEXPFINAL1 | Growth |
| TODO-054 | Implement neighborhood-level task feed (geo-fenced discovery for network density) | hustlexp-ai-backend | Growth |
| TODO-055 | Add insurance pool sustainability controls (min pool balance, claim frequency limits, reinsurance trigger) | hustlexp-ai-backend | Financial Safety |
| TODO-056 | Implement surge pricing manipulation guard (cap demand signal changes per 15-min window) | hustlexp-ai-backend | Financial Safety |
| TODO-057 | Fix dispute window inconsistency (48h in DisputeService vs 6h in EscrowService) | hustlexp-ai-backend | Financial Safety |
| TODO-058 | Add admin auth cache invalidation on role revocation (current 5-min stale window) | hustlexp-ai-backend | Security |

---

## Changelog

- **2026-04-02**: Added TODO-026 through TODO-029 (knowledge graph indexer pipeline). Resolved TODO-023 (Supabase confirmed active, not legacy). Added STOP-004 to INFRASTRUCTURE-ERRORS.md — Kimi Moonshot API does not support embeddings, blocking indexer activation.
- **2026-04-02 (stress test)**: Added STOP-005 through STOP-012 to INFRASTRUCTURE-ERRORS.md (8 new findings from adversarial stress test loops). Added TODO-030 through TODO-058 (29 new items): P0.5 critical financial patches (030-037), P2.5 anti-abuse hardening (038-044), P4 Checkr integration, viral growth, insurance sustainability, and architecture fixes (045-058). Source: vault pages 14-15 (Stress-Test-Worst-Outcomes + Optimization-Playbook).
