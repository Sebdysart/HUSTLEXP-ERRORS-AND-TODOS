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
- **Supabase "aura-ventures"**: INACTIVE. Connection timeouts. Likely legacy/abandoned.

**Action**: Confirm Supabase is legacy. If so, remove all Supabase references from codebase and documentation. If not, investigate why it's inactive.

## RESOLVED: Backend CI/CD (STOP-002)
4 GitHub Actions workflows confirmed active: `ci.yml`, `deploy-aws.yml`, `deploy.yml`, `security.yml`.
