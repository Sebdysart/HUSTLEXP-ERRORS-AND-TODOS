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
