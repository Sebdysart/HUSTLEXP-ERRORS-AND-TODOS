# HUSTLEXP-ERRORS-AND-TODOS

Central tracking repository for all known errors, vulnerabilities, TODO items, and architectural issues across the HustleXP platform.

**Last Updated**: 2026-04-02
**Audit Baseline**: Full source-level platform audit + adversarial stress test, April 2026

---

## Overview

This repo consolidates findings from live source-level audits and adversarial stress testing across the HustleXP ecosystem. Every finding is grounded to specific files, line numbers, and code patterns with actionable remediation steps.

**Repos covered:**
- **hustlexp-ai-backend** — Backend API (Hono + tRPC, 68 services, 38 routers, 290+ procedures)
- **HUSTLEXPFINAL1** — React Native + Swift iOS client
- **omni-link-hustlexp** — Multi-repo engineering control plane
- **HUSTLEXP-DOCS** — Documentation authority (316 markdown files)
- **HustleXP-Vault** — Obsidian knowledge vault (16 audit pages)

---

## Quick Stats (as of 2026-04-02)

| Category | Count |
|----------|-------|
| STOP Errors (Infrastructure) | 12 (3 CRITICAL, 4 HIGH, 2 MEDIUM, 1 RESOLVED, 1 REVISED, 1 NEW) |
| INFO Findings | 2 (BackgroundCheck stub, Sybil defense gap) |
| TODOs (All Priority) | 64 across 7 priority tiers |
| Backend Code Errors | 30+ |
| Frontend Code Errors | 15+ |
| Security Findings | 15 |
| Financial Invariants | 8 verified (1 known violation under re-verification) |

---

## Priority Breakdown (64 TODOs)

| Priority | IDs | Count | Focus |
|----------|-----|-------|-------|
| **P0 — Launch Blockers** | TODO-001 to 005 | 5 | Stripe config, SSL pins, npm audit, CI/CD |
| **P0.5 — Critical Financial** | TODO-030 to 037 | 8 | Stripe idempotency, post-commit failures, stub guard, referral payout, XP velocity, ASAP/escrow mismatch |
| **P1 — Financial Safety** | TODO-006 to 010 | 5 | DisputeService verification, excluded tests, coverage thresholds |
| **P2 — Code Quality** | TODO-011 to 017, 026-029 | 11 | Logging, config consolidation, knowledge graph pipeline |
| **P2.5 — Anti-Abuse** | TODO-038 to 044 | 7 | Sybil defense, collusion detection, streak timezone, rejection counter, Redis/PG drift |
| **P3 — Testing** | TODO-018 to 021 | 4 | Test fixes, E2E payment flow, credential audit |
| **P4 — Architecture** | TODO-022 to 025, 045-064 | 24 | Checkr API, viral growth, insurance sustainability, notification reliability, outbox performance |

---

## STOP Errors (Infrastructure Blockers)

### CRITICAL (Real Money Risk)
| ID | Issue | File | Impact |
|----|-------|------|--------|
| STOP-005 | Stripe `transfers.create()` missing idempotency key | StripeService.ts | Double-payout on retry |
| STOP-006 | Post-commit side effects (revenue, XP, insurance) outside transaction | EscrowService.ts | Silent revenue/XP loss |
| STOP-007 | SelfInsurancePool direct `fetch()` bypasses StripeService | SelfInsurancePoolService.ts | Zero financial safeguards on claims |

### HIGH (Must Fix Before Beta)
| ID | Issue | Impact |
|----|-------|--------|
| STOP-001 | Stripe dashboard completely empty — $0 balance, 0 products | Payment flow dead on arrival |
| STOP-008 | Webhook idempotency TOCTOU race (SELECT then INSERT, not atomic) | Duplicate webhook processing |
| STOP-009 | `HX_STRIPE_STUB=1` bypasses all Stripe calls with no prod guard | Complete payment bypass if misconfigured |
| STOP-010 | Referral rewards marked paid but no Stripe transfer created | Users never receive $5 reward |
| STOP-011 | XP velocity check returns `suspicious: false` on error | Fraud detection bypass under load |
| STOP-012 | ASAP price bump updates task price but not escrow amount | Worker payout $9 short on bumped tasks |

### MEDIUM / INFO
| ID | Issue |
|----|-------|
| STOP-004 | Knowledge graph indexer blocked — no embedding provider |
| INFO-001 | BackgroundCheckService never calls Checkr API (stub) |
| INFO-002 | No Sybil defense on account creation (rate limiting, phone verification) |

### RESOLVED
| ID | Resolution |
|----|-----------|
| STOP-002 | Backend CI/CD confirmed active (4 GitHub Actions workflows) |
| STOP-003 | Database provider revised — Neon PostgreSQL (primary) + Supabase (vector store) |

---

## Detailed Tracking Files

| File | Contents |
|------|----------|
| [INFRASTRUCTURE-ERRORS.md](./INFRASTRUCTURE-ERRORS.md) | 12 STOP errors + 2 INFO findings with full root cause analysis |
| [TODOS-BY-PRIORITY.md](./TODOS-BY-PRIORITY.md) | 64 TODOs across P0–P4 with repo, category, and effort estimates |
| [BACKEND-ERRORS.md](./BACKEND-ERRORS.md) | 30+ backend code-level errors and vulnerabilities |
| [FRONTEND-ERRORS.md](./FRONTEND-ERRORS.md) | 15+ frontend errors, security gaps, quality issues |
| [FINANCIAL-INVARIANT-STATUS.md](./FINANCIAL-INVARIANT-STATUS.md) | 8 financial invariants verification status |
| [SECURITY-FINDINGS.md](./SECURITY-FINDINGS.md) | 15 cross-repo security audit findings |

---

## Stress Test Findings (April 2026)

Seven adversarial stress test loops were run against the full backend source code (April 2, 2026). Key areas tested:

| Loop | Focus | Source (Vault Page) |
|------|-------|-------------------|
| 1 | Escrow, Stripe, XP, disputes, auth, AI, DB | [[14-Stress-Test-Worst-Outcomes]] |
| 2 | Checkr retention play, earned verification ladder | [[15-Optimization-Playbook]] |
| 3 | Gamification exploits (streaks, badges, referrals, viral K-factor) | [[15-Optimization-Playbook]] |
| 4 | Sybil attacks, graph-based collusion | [[15-Optimization-Playbook]] |
| 5 | Economic model (insurance pool, surge pricing, IC compliance) | [[15-Optimization-Playbook]] |
| 6 | Growth bottlenecks, viral coefficient, mega-viral formula | [[15-Optimization-Playbook]] |
| 7+ | Workers, notifications, queues, outbox (deep source review) | [[16-Deep-Stress-Test-Workers-Notifications-Queues]] |

**Result**: 4 CRITICAL, 11 HIGH, 10+ MEDIUM findings. 7 confirmed architectural strengths in the worker/queue layer (atomic claims, HMAC signing, Zod validation, optimistic locking, triple-layer deduplication, DLQ monitoring).

**Key insight**: The financial worker layer (PaymentWorker, EscrowActionWorker) is production-ready. The service layer (EscrowService, StripeService, SelfInsurancePool) is where the real money risk lives — fix STOP-005 through STOP-007 first.

---

## Patch Priority Order

**Week 1 (Critical financial patches — ~15 hours):**
1. STOP-005: Add idempotency keys to Stripe transfers/refunds (2h)
2. STOP-006: Move post-commit side effects into transaction or outbox (4-6h)
3. STOP-007: Refactor insurance pool to use StripeService (2h)
4. STOP-009: Add production guard against HX_STRIPE_STUB (15min)
5. STOP-011: Change XP velocity check to fail closed (30min)
6. STOP-008: Fix webhook TOCTOU with atomic INSERT ON CONFLICT (2h)

**Week 2 (High priority — ~10 hours):**
7. STOP-010: Wire actual Stripe transfer in referral rewards (2h)
8. STOP-012: Fix ASAP bump / escrow amount mismatch (3-4h)
9. TODO-038: Rate-limit account creation (Sybil phase 1) (3h)
10. TODO-042: Add rejection counter to TaskService (1h)

**Week 3+ (Anti-abuse, Checkr, viral, architecture):**
- See TODOS-BY-PRIORITY.md for full P2.5 through P4 breakdown

---

## Financial Invariants Status

8 core invariants verified as of 2026-03-31:

| # | Invariant | Status |
|---|-----------|--------|
| INV-1 | Escrow amounts are positive integers in cents | PASSING |
| INV-2 | XP only granted after escrow RELEASED | PASSING |
| INV-3 | Escrow released exactly once per task | PASSING |
| INV-4 | Ledger entries are append-only immutable | PASSING |
| INV-5 | All payment amounts are positive integers in cents | PASSING |
| INV-6 | Every financial op is atomic with audit log | PASSING |
| INV-7 | Double-release protection (released_at IS NULL check) | PASSING |
| INV-8 | Stripe webhooks processed idempotently | PASSING (with TOCTOU caveat — STOP-008) |

**Known violation**: ARCH-001 in DisputeService.ts:311 — direct escrow UPDATE outside service layer. Under re-verification.

---

## Repository Map

| Repo | Purpose | Status |
|------|---------|--------|
| [hustlexp-ai-backend](https://github.com/Sebdysart/hustlexp-ai-backend) | Backend API — Hono + tRPC, 68 services, 290+ procedures | Active, Railway deployed |
| [HUSTLEXPFINAL1](https://github.com/Sebdysart/HUSTLEXPFINAL1) | iOS client — React Native + Swift dual architecture | Active, CI passing |
| [omni-link-hustlexp](https://github.com/Sebdysart/omni-link-hustlexp) | Multi-repo engineering control plane | Active, 887 tests |
| [HUSTLEXP-DOCS](https://github.com/Sebdysart/HUSTLEXP-DOCS) | Documentation authority — 316 markdown files | Active |
| [HustleXP-Vault](https://github.com/Sebdysart/HustleXP-Vault) | Obsidian knowledge vault — 16 audit pages | Active |

---

## Changelog

- **2026-04-02 (loop 7+)**: Added TODO-059 to TODO-064 (notification/outbox reliability). Confirmed 7 architectural strengths in worker layer.
- **2026-04-02 (stress test)**: Added STOP-005 to STOP-012, INFO-001, INFO-002. Added TODO-030 to TODO-058 (35 new items).
- **2026-04-02**: Added STOP-004, TODO-026 to TODO-029 (knowledge graph). Resolved TODO-023 (Supabase confirmed active).
- **2026-04-01**: Initial audit baseline. STOP-001 to STOP-003, TODO-001 to TODO-025.

---

**Generated**: 2026-04-02
**Audit Baseline**: Full source-level platform audit + 7 adversarial stress test loops
**Next Review**: 2026-07-01 (Q2 2026)
