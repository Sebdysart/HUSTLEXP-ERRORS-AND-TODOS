# HUSTLEXP-ERRORS-AND-TODOS

> **Single source of truth for every known error, violation, gap, and TODO across the entire HustleXP platform.**
> Grounded to live audit data as of **April 1, 2026**. Updated by both manual review and automated scans.

[![Status](https://img.shields.io/badge/Platform_Status-Pre--Launch-orange)]()
[![Showstoppers](https://img.shields.io/badge/Showstoppers-5-red)]()
[![Arch_Violations](https://img.shields.io/badge/Arch_Violations-1-yellow)]()
[![Invariant_Violations](https://img.shields.io/badge/Invariant_Violations-0-green)]()

---

## How to Use This Repo

This repo tracks **every known defect and task** across all HustleXP repositories and infrastructure. It does NOT contain application code. It is the error/todo counterpart to [HUSTLEXP-DOCS](https://github.com/Sebdysart/HUSTLEXP-DOCS) (which contains specs and documentation).

**Related Repositories:**

| Repo | Role | Status |
|------|------|--------|
| [hustlexp-api](https://github.com/Sebdysart/hustlexp-api) | Production Backend (Fastify, PostgreSQL, Stripe) | Production-ready, no CI/CD |
| [HUSTLEXPFINAL1](https://github.com/Sebdysart/HUSTLEXPFINAL1) | iOS/Mobile Client (React Native + Swift) | In development |
| [omni-link-hustlexp](https://github.com/Sebdysart/omni-link-hustlexp) | Engineering Control Plane | Operational (887 tests) |
| [HUSTLEXP-DOCS](https://github.com/Sebdysart/HUSTLEXP-DOCS) | Documentation Authority (242 files) | Comprehensive |
| **HUSTLEXP-ERRORS-AND-TODOS** | **This repo — error/todo tracker** | **Active** |

---

## SHOWSTOPPERS (Must Fix Before Any User Traffic)

These block launch entirely. No user should see the app until all 5 are resolved.

| ID | Finding | Repo/System | Severity | Phase |
|----|---------|-------------|----------|-------|
| **STOP-001** | Stripe account completely unconfigured — $0 balance, 0 products, 0 customers, 0 payment intents | Stripe | CRITICAL | Phase 0 |
| **STOP-002** | Backend has ZERO CI/CD — no GitHub Actions, all testing manual | hustlexp-api | CRITICAL | Phase 1 |
| **STOP-003** | Supabase project "aura-ventures" is INACTIVE with connection timeouts; unclear if this or Railway is production DB | Supabase | CRITICAL | Phase 0 |
| **STOP-004** | SSL certificate pin hashes in iOS client are placeholders — MITM vulnerability on all API calls | HUSTLEXPFINAL1 | CRITICAL | Phase 2 |
| **STOP-005** | Zero E2E tests on frontend — no regression safety net for payment flow | HUSTLEXPFINAL1 | CRITICAL | Phase 2 |

---

## ARCHITECTURE VIOLATIONS

| ID | Location | Rule Violated | Severity | Status |
|----|----------|---------------|----------|--------|
| **ARCH-001** | `DisputeService.ts:311` | Direct `UPDATE escrows` outside EscrowService for LOCKED_DISPUTE transition | MEDIUM | OPEN |

**Detail:** `DisputeService.openDispute()` executes `UPDATE escrows SET state = 'LOCKED_DISPUTE'` directly via `db.query()` instead of calling `EscrowService.lockForDispute()`. This creates a dual-maintenance path — the same state transition exists in both EscrowService:1224 and DisputeService:311. Logic drift risk increases with each sprint.

**Mitigations present:** Wrapped in `db.transaction()`, `SELECT FOR UPDATE` row lock, state guard WHERE clause, version optimistic lock. No immediate money-loss path.

**Fix:** Extract to `EscrowService.lockForDispute(escrowId, query)` with dependency injection pattern.

---

## SECURITY FINDINGS (Frontend)

| ID | Finding | Severity | Status | Phase |
|----|---------|----------|--------|-------|
| **SEC-001** | SSL pin hashes are placeholders | CRITICAL | OPEN | Phase 2 |
| **SEC-002** | Zero E2E tests | CRITICAL | OPEN | Phase 2 |
| **SEC-003** | Firebase Crashlytics commented out — no crash reporting | HIGH | OPEN | Phase 2 |
| **SEC-004** | No input validation on any user-facing form | HIGH | OPEN | Phase 2 |
| **SEC-005** | Production logging 100% disabled | HIGH | OPEN | Phase 2 |

---

## FINANCIAL INVARIANT STATUS

All 8 financial invariants verified clean as of 2026-03-31 scan.

| Invariant | Rule | Status |
|-----------|------|--------|
| INV-1 | Escrow amounts: positive integers in cents only | CLEAN |
| INV-2 | XP only after escrow fully released + ledger immutable | CLEAN |
| INV-3 | Escrow released exactly once (DB + app level) | CLEAN |
| INV-4 | Ledger entries: append-only immutable | CLEAN |
| INV-5 | All payment amounts: positive integers in cents | CLEAN |
| INV-6 | All financial ops atomic + audit log before response | CLEAN |
| INV-7 | released_at IS NULL in same transaction as release | CLEAN |
| INV-8 | Stripe webhooks idempotent (stripe_event_id unique) | CLEAN |

---

## MINOR / CLEANUP ITEMS

| ID | Finding | Repo | Priority |
|----|---------|------|----------|
| **CLEAN-001** | `Math.floor()` vs `Math.round()` inconsistency for platformFee between StripeService:165 and EscrowService | hustlexp-api | Low |
| **CLEAN-002** | 5 stale branches across repos (4 closed-PR branches + 1 abandoned) | Multiple | Low |
| **CLEAN-003** | PR #5 in HUSTLEXPFINAL1 open 10+ days with passing CI — merge or close | HUSTLEXPFINAL1 | Low |
| **CLEAN-004** | "hustlexp-ai-backend" referenced in project instructions but does NOT exist | Documentation | Medium |

---

## INFRASTRUCTURE GAPS

| ID | Gap | Current State | Required State | Phase |
|----|-----|---------------|----------------|-------|
| **INFRA-001** | Stripe not configured | $0, 0 products, 0 customers | Test products, test payments, webhook endpoint | Phase 0 |
| **INFRA-002** | Database provider undecided | Supabase INACTIVE + Railway referenced in docs | Single provider, active, backed up | Phase 0 |
| **INFRA-003** | Backend CI/CD missing | 0 GitHub Actions workflows | Test + lint + type-check on every PR | Phase 1 |
| **INFRA-004** | No production monitoring | Crashlytics disabled, logging off | Crash reporting + structured logging | Phase 2 |
| **INFRA-005** | No alerting | None | Failed payments, stuck escrows, dispute alerts | Phase 3 |

---

## DOCUMENTATION-REALITY GAP

The [HUSTLEXP-DOCS](https://github.com/Sebdysart/HUSTLEXP-DOCS) repo describes a system significantly larger than what is currently built.

| Metric | Documented | Actually Built | Gap |
|--------|-----------|----------------|-----|
| Database Tables | 103 | 10 | 93 (90%) |
| API Procedures | 294 | 23 | 271 (92%) |
| tRPC Routers | 50 | ~5 | ~45 (90%) |
| Passing Tests | 5,448 | 915 | 4,533 (83%) |
| AI Agents | 4 | 0 confirmed live | 4 (100%) |
| Live Features | 30+ | ~8 core | ~22 (73%) |
| BullMQ Workers | 23 | 0 confirmed | 23 (100%) |

**This is not a defect** — the docs represent the target architecture. But READMEs and status badges must reflect current reality, not aspirational state.

---

## LEGACY REPO CLEANUP

| Repo | Disposition | Reason |
|------|-------------|--------|
| `HustlexpAPI` | DELETE | Empty, zero commits, causes naming confusion |
| `HustleXP` (original) | ARCHIVE | CI prototype from Oct 2025, superseded by omni-link |
| `Hustlexpv1` | ARCHIVE | Investor demo from Dec 2025, superseded by HUSTLEXPFINAL1 |
| `clawbot-hustlexp-autonomous` | ARCHIVE | Single commit prototype, Feb 2026 |
| `rork-hustlexp-gig-marketplace-app-796` | EVALUATE | Rork auto-gen app, may have reusable UI patterns |

---

## ROADMAP (Risk-Ordered Phases)

| Phase | Name | Timeline | Key Deliverables |
|-------|------|----------|-----------------|
| **0** | Infrastructure Triage | Week 1 | DB decision, Stripe config, naming fix, branch cleanup |
| **1** | Financial Safety Net | Weeks 2–3 | Backend CI/CD, fix ARCH-001, Stripe E2E test |
| **2** | Frontend Security | Weeks 4–5 | Fix SEC-001→005, E2E tests, architecture decision |
| **3** | Soft Launch | Weeks 6–8 | TestFlight, 10–20 users, daily reconciliation |
| **4** | Scale & Expand | Weeks 9+ | Additional tables, AI agents, geographic expansion |

---

## PENDING DECISIONS

| ID | Decision | Options | Blocking |
|----|----------|---------|----------|
| **DEC-001** | Database provider | Supabase vs. Railway PostgreSQL | Phase 0 |
| **DEC-002** | Frontend architecture | React Native only vs. Swift only vs. Both | Phase 2 |

---

## How This Repo Is Maintained

- **Automated scans**: Financial invariant surface scan runs on schedule against the backend worktree
- **Manual updates**: After each audit or significant change, this README is updated
- **Issue tracking**: Each showstopper and security finding can be tracked as a GitHub Issue in this repo
- **Cross-repo linking**: All repos' READMEs link back here for the canonical error/todo list

---

*Last updated: 2026-04-01 | Source: Comprehensive platform audit across all repositories and infrastructure*
