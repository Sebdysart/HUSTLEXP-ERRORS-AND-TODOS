# Backend Errors & Issues (hustlexp-ai-backend)

## Dependency Vulnerabilities (npm audit)
15 total vulnerabilities: 8 low, 2 moderate, 5 high

| Package | Severity | CVE/Advisory | Fix Available |
|---------|----------|-------------|---------------|
| hono <=4.12.6 | HIGH | GHSA-5pq2-9x2x-5p6w — Cookie Attribute Injection via setCookie() | Yes — `npm audit fix` |
| hono <=4.12.6 | HIGH | GHSA-p6xx-57qc-3wxr — SSE Control Field Injection via writeSSE() | Yes — `npm audit fix` |
| hono <=4.12.6 | HIGH | GHSA-q5qw-h33p-qvwr — Arbitrary file access via serveStatic | Yes — `npm audit fix` |
| hono <=4.12.6 | HIGH | GHSA-v8w9-8mx6-g223 — Prototype Pollution via parseBody({ dot: true }) | Yes — `npm audit fix` |
| fast-xml-parser (via @aws-sdk) | HIGH | GHSA-8gc5-j5rx-235r — Numeric entity expansion bypass | Yes — `npm audit fix` |
| node-forge <=1.3.3 | HIGH | GHSA-2328-f5f3-gj25 — basicConstraints bypass in certificate chain | Yes — `npm audit fix` |
| node-forge <=1.3.3 | HIGH | GHSA-q67f-28xg-22rw — Ed25519 signature forgery | Yes — `npm audit fix` |
| node-forge <=1.3.3 | HIGH | GHSA-5m6q-g25r-mvwx — BigInteger.modInverse() DoS | Yes — `npm audit fix` |
| node-forge <=1.3.3 | HIGH | GHSA-ppp5-5v6c-4jwp — RSA-PKCS signature forgery | Yes — `npm audit fix` |
| yaml 2.0.0-2.8.2 | MODERATE | GHSA-48c2-rrv3-qjmp — Stack Overflow via deep YAML | Yes — `npm audit fix` |

**Action**: Run `npm audit fix` immediately. For hono, upgrade to >=4.12.7.

## Excluded Test Files (vitest.config.ts)
9 test files are excluded from the test suite due to broken imports or wrong mock abstractions:

| File | Reason | Priority |
|------|--------|----------|
| `src-layer-services-batch.test.ts` | Unknown import issues | MEDIUM |
| `errors-branches.test.ts` | Imports non-existent `src/utils/errors` | HIGH |
| `safety-branches.test.ts` | Imports non-existent `src/config/safety` | HIGH |
| `stripe-money-engine.test.ts` | Imports non-existent `StripeMoneyEngine.ts` | HIGH |
| `stripe-service-src.test.ts` | Imports non-existent `StripeMoneyEngine.ts` | HIGH |
| `service-tax-compliance-extra.test.ts` | Imports non-existent `TaxComplianceService` | MEDIUM |
| `service-feed-query.test.ts` | Wrong db abstraction (sql vs db.query) | MEDIUM |
| `service-capability-profile.test.ts` | Wrong db abstraction (sql vs db.query) | MEDIUM |
| `query-cache-extra.test.ts` | Wrong Redis mock (upstash vs redis.js) | MEDIUM |

**Impact**: These 9 files represent untested code paths. 3 reference non-existent service modules (StripeMoneyEngine, TaxComplianceService) — these may be planned but unbuilt services.

## Console.log Usage in Production Source
43 instances of `console.log/warn/error` in `backend/src/` instead of the structured `pino` logger:

| Service | Count | Severity |
|---------|-------|----------|
| WorkerSkillService.ts | 10 | MEDIUM — all `console.error` in catch blocks |
| TaskDiscoveryService.ts | 10 | MEDIUM — all `console.error` in catch blocks |
| server.ts | 8 | LOW — startup CORS validation messages |
| IntentParserService.ts | 3 | MEDIUM — `console.warn` in error paths |
| IncidentDiagnosisService.ts | 3 | MEDIUM — `console.warn` in error paths |
| AIRouter.ts | 2 | HIGH — error handling in AI budget tracking |
| ai/rateLimit.ts | 1 | MEDIUM — rate limit failure |
| security.ts | 1 | LOW — dev-mode rate limit warning |
| config.ts | 1 | LOW — config validation |
| AnomalyDetectionService.ts | 1 | MEDIUM |

**Action**: Migrate all `console.*` calls to `pino` logger for structured logging, log levels, and production observability.

## eslint-disable Directives
4 eslint-disable comments in production source (all justified):

| File | Line | Rule Disabled | Justification |
|------|------|--------------|---------------|
| ai-guard.ts:141 | `no-control-regex` | Intentional control char removal from AI output |
| security.ts:286 | `no-control-regex` | Intentional control char removal for input sanitization |
| config.ts:244 | `no-console` | Config validation at startup |
| monitoring/metrics.ts:70 | `@typescript-eslint/no-explicit-any` | Prometheus metric type flexibility |

**Status**: All 4 justified. No action needed unless stricter lint policy adopted.

## Coverage Threshold Gap
Current vitest coverage thresholds are set to 10% (lines, functions, branches, statements). Actual coverage is 89.6% statements / 77.6% branches. The thresholds should be raised to prevent regression.

**Coverage ramp-up plan (from vitest.config.ts comments):**
- Phase 4b (current): 10% threshold
- Phase 5 (GA target): 40%
- Phase 6 (Mature target): 70%

**Action**: Immediately raise thresholds to at least 70% to match current reality and prevent regression.

## Missing TODO.md
`dispute.ts:16` references `TODO.md §iOS Feature Sync` but **no TODO.md file exists** in the repo. This is a broken reference.

## Architecture: Direct db.query in 20+ Services
The following services use `db.query` directly for database operations. While this is the standard pattern (not all services are restricted to service-layer abstractions), any service touching escrow/task/ledger tables should go through the dedicated service layer:

Services with `db.query`: TrustTierService, DisputeService, MatchmakerAIService, ComplianceGuardianService, WorkerSkillService, StripeSubscriptionProcessor, JudgeAIService, JuryPoolService, PushNotificationService, ContentModerationService, AIProposalService, EligibilityGuard, StripeService, DisputeAIService, FeedQueryService, ScoperAIService, OnboardingAIService, AnomalyDetectionService, GeofenceService, HeatMapService, LogisticsAIService, BetaService, NotificationService, MovementTrackingService, MessagingService, EscrowService, ChargebackService, TaxReportingService, ReputationAIService, InstantObservability, and many more (68 total).

**Known Violation**: ARCH-001 — DisputeService direct UPDATE on escrows table (needs re-verification against current codebase).

## Scattered process.env Usage
67 `process.env` references in `config.ts` (centralized — good). But 20+ additional `process.env` references scattered across other files: `messaging.ts`, `upload.ts`, `ai-guard.ts`, `encrypted-session.ts`, `sentry.ts`, `logger.ts`, `escrow-action-worker.ts`, `db.ts`, `datadog.ts`.

**Action**: Consolidate all env var access through `config.ts` for single-source-of-truth configuration.

## Sentry Release Tag Incorrect
`sentry.ts:27` sets release as `hustlexp-api@${version}` — should be `hustlexp-ai-backend@${version}`.

## God Services (Complexity Risk)
Services over 1,000 lines (high complexity, refactor candidates):

| Service | Lines | Risk |
|---------|-------|------|
| ExpertiseSupplyService.ts | 1,621 | HIGH — largest service |
| TaskService.ts | 1,487 | CRITICAL — core financial path |
| NotificationService.ts | 1,430 | MEDIUM |
| GDPRService.ts | 1,286 | HIGH — compliance-critical |
| TaskDiscoveryService.ts | 1,111 | MEDIUM |
| EscrowService.ts | 1,040 | CRITICAL — financial core |
| ContentModerationService.ts | 1,036 | MEDIUM |
