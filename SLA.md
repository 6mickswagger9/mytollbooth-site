# Service Level Agreement (SLA)

**Provider:** MyTollBooth
**Contact:** support@mytollbooth.com
**Website:** https://6mickswagger9.github.io/mytollbooth-site
**Effective Date:** November 3, 2026
**Version:** 1.0

---

## 1. Scope

This SLA covers all 24 production services (100 endpoints: 99 paid, 1 free) hosted on Railway and accessible via the x402 v2 payment protocol on Base (USDC).

| Service | Paid Endpoints |
|---------|---------------|
| compute-api | 17 |
| x402-price-api | 6 |
| risk-api | 7 |
| scan-api | 6 |
| url-intel-api | 7 |
| webhook-api | 9 |
| code-toll | 4 |
| image-process-api | 5 |
| valuation-api | 5 |
| margin-api | 4 |
| escrow-api | 4 |
| calendar-api | 4 |
| screenshot-api | 3 |
| tax-api | 3 + 1 free |
| compliance-api | 1 |
| collateral-eligibility | 1 |
| reconciliation-api | 2 |
| llm-toll | 2 |
| translate-toll | 2 |
| gecko-toll | 2 |
| rdap-api | 1 |
| scrape-toll | 2 |
| search-toll | 1 |
| summarize-toll | 1 |

---

## 2. Availability

| Metric | Commitment |
|--------|-----------|
| Monthly uptime | >= 99.9% |
| Measured via | UptimeRobot (24 independent monitors) |
| Excluded | Scheduled maintenance (announced >= 24h prior) |

**Trailing performance (October 2026):** 99.992% uptime, 7 incidents, 1 hr total downtime, MTBF 2,550 hrs.

---

## 3. Performance

| Metric | Target | Measured (Oct 2026) |
|--------|--------|---------------------|
| p50 response time | < 300 ms | 252 ms |
| p99 response time | < 1,000 ms | - |
| Cold start (rare) | < 5,000 ms | 10,208 ms (single event) |

Cold starts are possible on Railway Hobby plan after inactivity. A 503 with retry_after header is returned in this case.

---

## 4. Error Contract

All errors are returned as structured JSON with machine-actionable codes. Every response includes an X-Request-ID header for traceability.

| HTTP Status | Meaning | Retry Guidance |
|-------------|---------|---------------|
| 402 | Payment required / insufficient | Include valid PAYMENT-SIGNATURE header (x402 v2). Check x402.json for pricing. |
| 429 | Rate limit exceeded | Respect X-RateLimit-Remaining and X-RateLimit-Reset headers. Wait and retry. |
| 503 | Service unavailable / cold start | Retry after retry_after seconds. |
| 400 | Malformed request | Fix request body/headers. Do not retry. |
| 404 | Endpoint not found | Check x402.json for valid routes. |
| 500 | Internal error | Retry with exponential backoff (max 3 attempts). |

Structured error body example:
  { "error": "rate_limited", "message": "Rate limit exceeded for this wallet.", "retry_after": 30, "request_id": "a3f2b8c1-..." }

---

## 5. Rate Limits

| Scope | Limit | Enforcement |
|-------|-------|-------------|
| Per IP (top 5 services) | 30 req/min | 429 + X-RateLimit-* headers |
| Per wallet (all 24) | 100 req/min | 429 + X-RateLimit-* headers |
| 503 + retry_after | Top 5 services | Returned on overload |

Headers returned on every response:
- X-RateLimit-Limit
- X-RateLimit-Remaining
- X-RateLimit-Reset

---

## 6. Payment Protocol

| Parameter | Value |
|-----------|-------|
| Protocol | x402 v2 |
| Header | PAYMENT-SIGNATURE |
| Network | Base (Chain ID 8453) |
| Currency | USDC |
| Facilitator | @coinbase/x402 (CDP JWT) |
| Quote TTL | 120 seconds |
| Replay protection | SHA-256 dedup, 1-hour window |
| Batch settlement | Supported (declared in x402.json) |
| Spend ceiling | Rejected with amount_below_minimum if below service minimum |

Discovery: Each service publishes an x402.json at its root with endpoint pricing, descriptions, and related-service chaining metadata.

---

## 7. Security

| Control | Status |
|---------|--------|
| Transport | HTTPS (TLS 1.2+) |
| Payment verification | EIP-3009 transferWithAuthorization (signed) |
| Idempotency | SHA-256 dedup cache (4 endpoints) |
| Request tracing | X-Request-ID on all responses |
| Structured logging | Error logs with request context |

---

## 8. Maintenance & Incidents

- Scheduled maintenance: Announced >= 24 hours prior via support@mytollbooth.com.
- Unplanned incidents: Acknowledged within 1 hour. Status visible via UptimeRobot public monitors.
- Post-mortem: Provided for any incident exceeding 15 minutes of downtime.

---

## 9. Support

| Channel | Response Time |
|---------|--------------|
| support@mytollbooth.com | < 24 hours |
| GitHub Issues (per-repo) | < 48 hours |

---

## 10. Service Credits

If monthly uptime falls below 99.9%:

| Downtime | Credit |
|----------|--------|
| 99.0% - 99.9% | 10% of monthly volume |
| 95.0% - 99.0% | 25% of monthly volume |
| < 95.0% | 50% of monthly volume |

Credits are issued as USDC to the requesting wallet within 5 business days of the affected month's close.

---

## 11. Revision History

| Version | Date | Change |
|---------|------|--------|
| 1.0 | Nov 3, 2026 | Initial publication for AP4M onboarding |

---

This SLA applies to all endpoints listed in Section 1. Pricing and endpoint availability are governed by each service's x402.json manifest and may change with 48 hours' notice.
