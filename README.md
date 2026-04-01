# HUSTLEXP-ERRORS-AND-TODOS

> **Single source of truth for every known error, violation, gap, and TODO across the entire HustleXP platform.**
> Grounded to live audit data as of **April 1, 2026**. Updated by both manual review and automated scans.
> **Backend identity corrected:** Real production backend is **hustlexp-ai-backend** (PRIVATE).

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
| **hustlexp-ai-backend** (PRIVATE) | Production Backend (Hono + tRPC, Neon PostgreSQL, 85 services, 4 AI agents) | LIVE on Railway, 5,448 tests passing, 89.6% coverage |
| [HUSTLEXPFINAL1](https://github.com/Sebdysart/HUSTLEXPFINAL1) | iOS/Mobile Client (React Native + Swift) | In development |
| [omni-link-hustlexp](https://github.com/Sebdysart/omni-link-hustlexp) | Engineering Control Plane | Operational (887 tests) |
| [HUSTLEXP-DOCS](https://github.com/Sebdysart/HUSTLEXP-DOCS) | Documentation Authority (242 files) | Comprehensive |
| **HUSTLEXP-ERRORS-AND-TODOS** | **This repo — error/todo tracker** | **Active** |

### Backend Tech Stack (hustlexp-ai-backend)
- **Framework:** Hono + tRPC (NOT Fastify)
- **Database:** PostgreSQL 16 on Neon (PostGIS enabled), 103 tables
- **Services:** 85 microservices, 50 routers, 290+ API procedures
- **Testing:** 239 test files, 5,448 passing tests, 89.6% statement coverage, 77.6% branch coverage
- **Workers:** 23 BullMQ job queues
- **AI Agents:** Judge, Matchmaker, Dispute, Reputation (all active)
- **Deployment:** Railway (LIVE), Terraform infrastructure, Docker containerized
- **Security:** v2.8.7 through v2.11.0 hardening, active rotation
- **Money Math:** decimal.js for all financial calculations
- **CI/CD:** 4 GitHub Actions workflows (ci.yml, deploy-aws.yml, deploy.yml, security.yml)

---

## SHOWSTOPPERS (Must Fix Before Any User Traffic)

These block launch entirely. No user should see the app until all 5 are resolved.

| ID | Finding | Repo/System | Severity | Phase | Status |
|----|---------|-----------|-----------|----|--------|
| **STOP-001** | Stripe account completely unconfigured — $0 balance, 0 products, 0 customers, 0 payment intents. Backend services and webhooks are wired but dashboard is empty. | Stripe | CRITICAL | Phase 0 | OPEN |
| **STOP-002** | Backend has ZERO CI/CD — no GitHub Actions, all testing manual | hustlexp-ai-backend | CRITICAL | Phase 1 | **RESOLVED** — 4 workflows active (ci.yml, deploy-aws.yml, deploy.yml, security.yml) |
| **STOP-003** | Database provider unclear — Supabase "aura-ventures" inactive, but backend is LIVE on Neon PostgreSQL + Railway | Database | CRITICAL | Phase 0 | **REVISED** — Neon is the active production database. Supabase may be legacy/irrelevant. |
| **STOP-004** | SSL certificate pin hashes in iOS client are placeholders — MITM vulnerability on all API calls | HUSTLEXPFINAL1 | CRITICAL | Phase 2 | OPEN |
| **STOP-005** | Zero E2E tests on frontend — no regression safety net for payment flow | HUSTLEXPFINAL1 | CRITICAL | Phase 2 | OPEN |

---

## ARCHITECTURE VIOLATIONS

| ID | Location | Rule Violated | Severity | Status |
|----|----------|---------------|----------|--------|
| **ARCH-001** | `DisputeService.ts:311` | Direct `UPDATE escrows` outside EscrowService for LOCKED_DISPUTE transition | MEDIUM | NEEDS RE-VERIFICATION |

**Detail:** `DisputeService.openDispute()` executes `UPDATE escrows SET state = 'LOCKED_DISPUTE'` directly via `db.query()` instead of calling `EscrowService.lockForDispute()`. This creates a dual-maintenance path — the same state transition exists in both EscrowService:1224 and DisputeService:311. Logic drift risk increases with each sprint.

**Mitigations present:** Wrapped in `db.transaction()`, `SELECT FOR UPDATE` row lock, state guard WHERE clause, version optimistic lock. No immediate money-loss path.

**Fix:** Extract to `EscrowService.lockForDispute(escrowId, query)` with dependency injection pattern.

**NOTE:** This finding was from analysis of the old hustlexp-api codebase. Must be re-verified against the actual hustlexp-ai-backend to confirm pattern still exists and severity level in current 85-service architecture.

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

All 8 financial invariants verified **CLEAN** as of 2026-03-31 scan against hustlexp-ai-backend. All enforced by PostgreSQL triggers, decimal.js precision, and transactional guarantees.

| Invariant | Rule | Enforced By | Status |
|-----------|------|-------------|--------|
| INV-1 | Escrow amounts: positive integers in cents only | PostgreSQL CHECK, decimal.js | CLEAN |
| INV-2 | XP only after escrow fully released + ledger immutable | Trigger, app-level state machine | CLEAN |
| INV-3 | Escrow released exactly once (DB + app level) | Unique constraint, LOCKED state guard | CLEAN |
| INV-4 | Ledger entries: append-only immutable | INSERT-only table, no UPDATE/DELETE | CLEAN |
| INV-5 | All payment amounts: positive integers in cents | decimal.js, CHECK constraints | CLEAN |
| INV-6 | All financial ops atomic + audit log before response | db.transaction(), audit_log table | CLEAN |
| INV-7 | released_at IS NULL in same transaction as release | Trigger ensures atomicity | CLEAN |
| INV-8 | Stripe webhooks idempotent (stripe_event_id unique) | UNIQUE(stripe_event_id) constraint | CLEAN |

---

## MINOR / CLEANUP ITEMS

| ID | Finding | Repo | Priority |
|----|---------|------|----------|
| **CLEAN-001** | Some features are coded but not fully wired to frontend (insurance contributions, dispute UI connection, Stripe fee enforcement) | hustlexp-ai-backend | Medium |
| **CLEAN-002** | 5 stale branches across repos (4 closed-PR branches + 1 abandoned) | Multiple | Low |
| **CLEAN-003** | PR #5 in HUSTLEXPFINAL1 open 10+ days with passing CI — merge or close | HUSTLEXPFINAL1 | Low |
| **CLEAN-004** | Platform documentation previously referenced "hustlexp-api" as backend but REAL backend is hustlexp-ai-backend (PRIVATE) | Documentation | **RESOLVED** |

---

## INFRASTRUCTURE GAPS

| ID | Gap | Current State | Required State | Phase |
|----|-----|---------------|----------------|-------|
| **INFRA-001** | Stripe dashboard unconfigured | $0 balance, 0 products, 0 payment methods | Test products, test prices, webhook endpoint configured | Phase 0 |
| **INFRA-002** | Database provider clarity | Neon (LIVE) confirmed; Supabase "aura-ventures" INACTIVE — unclear if legacy | Archive or document Supabase status; confirm Neon as single source of truth | Phase 0 |
| **INFRA-003** | Backend CI/CD gaps | 4 workflows exist (ci.yml, deploy-aws.yml, deploy.yml, security.yml) but may need trigger expansion | Expand test coverage, add pre-merge gates, security scanning on all PRs | Phase 1 |
| **INFRA-004** | No production monitoring | Crashlytics disabled, logging off | Crash reporting + structured logging + error budgets | Phase 2 |
| **INFRA-005** | No alerting | None | Failed payments, stuck escrows, dispute alerts, DB connection pool alerts | Phase 3 |

---

## DOCUMENTATION-REALITY GAP

The [HUSTLEXP-DOCS](https://github.com/Sebdysart/HUSTLEXP-DOCS) repo describes a system that is **substantially built** in the real backend (hustlexp-ai-backend). Documentation-reality gap is much smaller than previously thought.

| Metric | Documented | Actually Built | Gap | Gap % |
|--------|-----------|----------------|-----|-------|
| Database Tables | 103 | 103 | 0 | **0%** |
| API Procedures | 294 | 290+ | 4 | **1%** |
| tRPC Routers | 50 | 50 | 0 | **0%** |
| Passing Tests | 5,448 | 5,448 | 0 | **0%** |
| AI Agents | 4 | 4 (Judge, Matchmaker, Dispute, Reputation) | 0 | **0%** |
| BullMQ Workers | 23 | 23 | 0 | **0%** |
| Live Features | 30+ | 25+ core | ~5 | **~17%** |

**Analysis:** The documentation closely mirrors the backend implementation. Most gaps are features that are **coded but not fully wired to the frontend** (e.g., insurance contributions, dispute UI connection, Stripe fee enforcement). The backend is 89.6% statement coverage with 239 test files and 5,448 passing tests.

**This is NOT a defect** — the backend is mature and well-tested. The gaps are primarily in frontend integration and optional features.

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

| Phase | Name | Timeline | Key Deliverables | Blocking Issues |
|-------|------|----------|-----------------|-----------------|
| **0** | Infrastructure Triage | Week 1 | Confirm Neon as production DB; archive Supabase docs; configure Stripe dashboard (products, prices, webhooks); branch cleanup | STOP-001, STOP-003 |
| **1** | Financial Safety Net | Weeks 2–3 | Expand backend CI/CD gates; re-verify ARCH-001 against real codebase; Stripe E2E test; ensure decimal.js precision on all fee calculations | STOP-002 (partially), ARCH-001 |
| **2** | Frontend Security | Weeks 4–5 | Fix SEC-001–005 (SSL pins, E2E tests, logging, input validation); wire unconnected features (insurance, disputes, Stripe fees); architecture decision (React Native vs. Swift) | STOP-004, STOP-005, SEC-001–005 |
| **3** | Soft Launch | Weeks 6–8 | TestFlight beta with 10–20 users; daily escrow reconciliation; 24/7 monitoring active; dispute resolution runbook validated | INFRA-004, INFRA-005 |
| **4** | Scale & Expand | Weeks 9+ | Geographic expansion; additional gig categories; reputation system tuning; AI agent fine-tuning based on beta feedback | Feature backlog |

---

## PENDING DECISIONS

| ID | Decision | Options | Status | Blocking |
|----|----------|---------|--------|----------|
| **DEC-001** | Database provider confirmation | Officially declare Neon as production DB; archive Supabase "aura-ventures" or document its role | **RECOMMENDED:** Declare Neon primary, archive Supabase | Phase 0 |
| **DEC-002** | Frontend architecture | React Native only vs. Swift only vs. Dual (Expo + native modules) | OPEN | Phase 2 |

---

## How This Repo Is Maintained

- **Automated scans**: Financial invariant surface scan runs on schedule against hustlexp-ai-backend worktree
- **Manual audits**: Platform-wide audits conducted quarterly; backend audit completed 2026-03-31
- **Manual updates**: After each audit, findings, or significant change, this README is updated by human review
- **Issue tracking**: Each showstopper and security finding can be tracked as a GitHub Issue in this repo
- **Cross-repo linking**: All repos' READMEs link back here for the canonical error/todo list
- **Backend documentation**: See [hustlexp-ai-backend](https://github.com/Sebdysart/hustlexp-ai-backend) (PRIVATE) for:
  - Service architecture (85 services, 50 routers)
  - Database schema (103 tables on Neon PostgreSQL)
  - API procedures (290+ tRPC endpoints)
  - AI agents (Judge, Matchmaker, Dispute, Reputation)
  - Test coverage (5,448 tests, 89.6% statement coverage)
  - Deployment & infrastructure (Railway, Terraform, Docker)
  - Security hardening details (v2.8.7 through v2.11.0)

---

**Revision History:**
- **2026-04-01**: MAJOR REVISION — Backend identity corrected from "hustlexp-api" to "hustlexp-ai-backend" (PRIVATE). STOP-002 marked RESOLVED (4 CI/CD workflows confirmed). STOP-003 revised (Neon confirmed as active production DB). Documentation-reality gap re-analyzed: 0% for tables, 1% for procedures, 0% for tests. All financial invariants CLEAN.

---

*Last updated: 2026-04-01 | Source: Comprehensive platform audit across all repositories and infrastructure*
