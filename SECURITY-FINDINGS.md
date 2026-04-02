# Security Findings — All Repos

## Backend (hustlexp-ai-backend)

| ID | Finding | Severity | Status |
|----|---------|----------|--------|
| SEC-B01 | hono <=4.12.6: 4 HIGH vulnerabilities (cookie injection, SSE injection, file access, prototype pollution) | HIGH | OPEN — fix: `npm audit fix` |
| SEC-B02 | node-forge <=1.3.3: 4 HIGH vulnerabilities (cert bypass, sig forgery x2, DoS) | HIGH | OPEN — fix: `npm audit fix` |
| SEC-B03 | fast-xml-parser: entity expansion bypass (via @aws-sdk) | HIGH | OPEN — fix: `npm audit fix` |
| SEC-B04 | yaml: stack overflow via deep collections | MODERATE | OPEN — fix: `npm audit fix` |
| SEC-B05 | 4 eslint-disable directives (all justified, documented) | LOW | ACCEPTED |

## Frontend (HUSTLEXPFINAL1)

| ID | Finding | Severity | Status |
|----|---------|----------|--------|
| SEC-F01 | SSL pin hashes are placeholders — no MITM protection beyond CA | CRITICAL | OPEN |
| SEC-F02 | C7 rehearsal failure injection left in production client.ts | HIGH | OPEN |
| SEC-F03 | Zero input validation on all user-facing forms | HIGH | OPEN |
| SEC-F04 | Firebase Crashlytics commented out — no crash reporting | HIGH | OPEN |
| SEC-F05 | Production logging 100% disabled | HIGH | OPEN |
| SEC-F06 | GoogleService-Info.plist potentially exposing API_KEY | MEDIUM | NEEDS VERIFICATION |
| SEC-F07 | Zero E2E tests — no regression safety net for payment flow | CRITICAL | OPEN |
| SEC-F08 | Zero CI/CD — no automated security checks | CRITICAL | OPEN |

## omni-link-hustlexp
| ID | Finding | Severity | Status |
|----|---------|----------|--------|
| SEC-O01 | npm audit: 0 vulnerabilities | — | CLEAN |
