# All TODOs — Prioritized

## P0 — Launch Blockers (Do First)

| ID | Task | Repo | Category |
|----|------|------|----------|
| TODO-001 | Configure Stripe dashboard (products, prices, webhooks, test keys) | Stripe / hustlexp-ai-backend | Infrastructure |
| TODO-002 | Replace placeholder SSL pin hashes with real SPKI hashes | HUSTLEXPFINAL1 | Security |
| TODO-003 | Remove C7 rehearsal failure injection code from client.ts | HUSTLEXPFINAL1 | Security |
| TODO-004 | Run `npm audit fix` on backend (5 HIGH vulnerabilities in hono, node-forge, fast-xml-parser) | hustlexp-ai-backend | Security |
| TODO-005 | Add frontend CI/CD pipeline (GitHub Actions: lint, typecheck, test, build) | HUSTLEXPFINAL1 | Infrastructure |

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

---

## Changelog

- **2026-04-02**: Added TODO-026 through TODO-029 (knowledge graph indexer pipeline). Resolved TODO-023 (Supabase confirmed active, not legacy). Added STOP-004 to INFRASTRUCTURE-ERRORS.md — Kimi Moonshot API does not support embeddings, blocking indexer activation.
