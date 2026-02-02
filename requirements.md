# requirements.md

## Project Overview
- **Project Name:** SmartRetail Intelligence Copilot
- **Summary:** An AI-powered platform that provides demand forecasting, pricing intelligence, inventory optimization, promotion planning, customer insights, risk/anomaly detection, and compliance/document understanding—exposed via dashboards and an AI copilot for retailers and marketplace sellers.
- **Primary outcomes:** higher availability (fewer stockouts), lower working capital (less dead inventory), better margins, faster compliance checks, and faster decision-making.
- **MVP (Hackathon Demo):** single-tenant demo using CSV/API sample datasets, daily batch pipelines, web dashboard, and a copilot that can answer questions and trigger core recommendation tools.
- **Future/Production:** multi-tenant SaaS with near real-time ingestion options, automated monitoring/drift, and enterprise security controls.

## Problem Statement
- Retailers and marketplace sellers make pricing, replenishment, and promo decisions with **fragmented data** (POS/ERP/marketplace exports, competitor prices, inventory snapshots) and **manual analysis**, causing:
	- Stockouts (lost revenue, lower ratings)
	- Overstock/dead inventory (holding cost, markdowns)
	- Margin leakage (wrong price vs competitors/costs)
	- Compliance risk (invoice/certification errors)
- Existing solutions are often siloed (forecasting only, pricing only) and do not provide **grounded, explainable, action-oriented recommendations** with auditability.

## Goals
- **Unify data → insights → actions** for demand, pricing, inventory, promotions, customer insights, anomalies, and compliance.
- Provide **SKU/store/day** forecasts with actionable confidence and drivers.
- Provide **pricing suggestions** with guardrails (margin, policy, max change).
- Provide **inventory recommendations** (reorder point, safety stock, reorder quantity) aligned to service level.
- Provide a **copilot** that answers questions using RAG + tool-calling and includes sources.
- Deliver a hackathon-ready MVP that works end-to-end with real datasets and a reproducible pipeline.

## Success Metrics (KPIs)
- **Stockout reduction target:** reduce stockout events for top SKUs by **≥ 15%** vs baseline period (or simulated baseline) within 8 weeks.
- **Dead inventory reduction target:** reduce “dead inventory” (no sales in last 60 days) value by **≥ 10%** within 12 weeks.
- **Margin improvement target:** improve gross margin by **≥ 2 percentage points** on recommended SKUs (measured vs pre-change baseline).
- **Forecast accuracy target:** achieve **MAPE ≤ 20%** for top 500 SKUs at weekly horizon (7–28 days) in backtests.
- **Pricing recommendation quality:** ≥ **70%** of recommended price actions pass guardrails (min margin, max delta) and are “actionable”.
- **Latency targets:**
	- Dashboard KPI view p95 **≤ 3s**
	- `POST /copilot/chat` p95 **≤ 6s** for queries that require ≤ 1 tool call
	- OCR extraction p95 **≤ 12s** per 2-page PDF
	- `POST /forecast` p95 **≤ 4s** when serving precomputed forecasts

## User Personas / Roles
- **Retail Manager**
	- Owns category performance; approves price/promotions; monitors service levels.
- **Marketplace Seller/MSME**
	- Manages listings, inventory and pricing; needs quick, low-effort recommendations.
- **Admin**
	- Configures data connectors, RBAC, thresholds, retention, and audit.

## Scope
### In Scope
- Ingestion of sales/inventory/product/price/promo/customer (where available) datasets via CSV and/or API.
- Dashboards for KPIs and recommendation workflows.
- Forecasting, pricing, inventory optimization, promotion planning (uplift), customer insights, anomaly/risk detection.
- Document understanding (OCR + extraction + validation) for invoices/policies.
- Copilot interface to query data, explain recommendations, and retrieve policy/document evidence.

### Out of Scope
- Automatic purchase order placement and supplier EDI integration.
- End-to-end e-commerce order management (checkout, fulfillment).
- Fully automated dynamic pricing without human approval (production may add controlled automation).
- Payments risk scoring / credit underwriting.

## Functional Requirements

### Data ingestion (POS/ERP/Marketplace CSV/API)
- **FR-DI-1 (MVP):** System shall ingest CSV files for `sales_transactions`, `inventory_snapshots`, `product_master`, `competitor_prices`, and `promotions_calendar` into a raw storage zone and validate schema on upload.
	- **Acceptance:** upload returns success/failure with row-level error report for invalid rows.
- **FR-DI-2 (MVP):** System shall support one API ingestion adapter (mock or sample) for marketplace orders/inventory with a scheduled pull (daily).
	- **Acceptance:** a scheduled run produces a dated file in raw zone.
- **FR-DI-3 (Future):** Support incremental ingestion with idempotency keys and a dead-letter queue for failures.

### Dashboard & analytics
- **FR-DA-1 (MVP):** System shall provide KPI tiles for revenue, gross margin %, units sold, stockout rate, inventory turns, and dead inventory value.
	- **Acceptance:** KPIs match computed values from curated tables for a selected date range.
- **FR-DA-2 (MVP):** System shall allow filters by date range, store/region, category, and SKU.
	- **Acceptance:** applying a filter updates all charts consistently.
- **FR-DA-3 (Future):** Role-specific dashboards (seller vs manager) and saved views.

### Demand forecasting (SKU/store/date)
- **FR-DF-1 (MVP):** System shall generate forecasts at **SKU×store×day** for a configurable horizon (7/14/28 days).
	- **Acceptance:** forecast output includes `yhat`, `yhat_lower`, `yhat_upper` per date.
- **FR-DF-2 (MVP):** System shall provide backtest evaluation for selected SKUs (rolling window) and display MAPE.
	- **Acceptance:** MAPE is computed and stored per SKU/week.
- **FR-DF-3 (Future):** Support hierarchical forecasts (store→region→national) and reconciliation.

### Pricing intelligence (competitor + suggestions)
- **FR-PI-1 (MVP):** System shall show current price vs competitor price band and flag parity violations when deviation > configurable threshold (default 5%).
	- **Acceptance:** for a SKU with competitor prices present, parity flag toggles correctly.
- **FR-PI-2 (MVP):** System shall output a recommended price with guardrails:
	- min margin % (default 10%), max daily price change % (default 8%).
	- **Acceptance:** recommendations never violate guardrails; if impossible, return “no_action” with reason.
- **FR-PI-3 (Future):** Support promotion-aware and channel-specific pricing.

### Inventory optimization (reorder point, safety stock)
- **FR-IO-1 (MVP):** System shall compute **safety stock** and **reorder point** per SKU-store using forecast uncertainty and lead time.
	- **Acceptance:** output includes `safety_stock`, `reorder_point`, `service_level`.
- **FR-IO-2 (MVP):** System shall recommend reorder quantity bounded by min/max order constraints.
	- **Acceptance:** recommended quantity respects constraints and is non-negative.
- **FR-IO-3 (Future):** Multi-echelon optimization and supplier reliability adjustments.

### Promotion planning (uplift forecast)
- **FR-PP-1 (MVP):** System shall estimate expected uplift for a planned promotion for selected SKUs over a date range.
	- **Acceptance:** output includes baseline demand, uplift %, and uplift units.
- **FR-PP-2 (Future):** Scenario comparison across discount levels and promo types (BOGO, bundle).

### Customer insights (segments/cohorts)
- **FR-CI-1 (MVP):** System shall produce basic customer cohorts (new/returning, RFM buckets) when customer_id is available.
	- **Acceptance:** cohort counts reconcile to unique customer counts in input.
- **FR-CI-2 (MVP):** System shall display top segments by revenue and repeat rate.
- **FR-CI-3 (Future):** LTV prediction and churn risk scoring.

### Alerts & notifications
- **FR-AN-1 (MVP):** System shall generate alerts for:
	- stockout risk within 7/14 days
	- price anomaly (competitor deviation > threshold)
	- demand spike/drop anomalies
	- compliance failures
	- **Acceptance:** each alert includes severity, reason, and recommended action.
- **FR-AN-2 (MVP):** System shall provide in-app alert inbox and email notifications.
	- **Acceptance:** an alert triggers an email to configured recipients.
- **FR-AN-3 (Future):** Snooze/escalation policies and alert deduplication.

### AI Copilot (chat-based query + recommendations)
- **FR-CP-1 (MVP):** Copilot shall answer natural language queries about KPIs, forecasts, pricing, inventory, promotions, customers, and documents.
	- **Acceptance:** responses include at least one cited source for policy/document claims.
- **FR-CP-2 (MVP):** Copilot shall support tool-calling for numeric answers, using:
	- Forecast tool, Recommendations tool, Pricing tool, Document validation tool.
	- **Acceptance:** for a query like “what will stock out next week?”, copilot calls at least one tool and returns structured results.
- **FR-CP-3 (Future):** Multi-turn memory across sessions (tenant-scoped) and action approvals.

### Document understanding (OCR + invoice/policy extraction)
- **FR-DU-1 (MVP):** System shall accept PDF/JPG/PNG documents and extract invoice fields: GSTIN, invoice number/date, seller/buyer name, taxable value, CGST/SGST/IGST, total.
	- **Acceptance:** extracted JSON contains all mandatory fields or explicit missing-field markers.
- **FR-DU-2 (MVP):** System shall validate compliance rules:
	- GSTIN format, invoice date not in future, tax totals consistency (tolerance ±0.5), certificate expiry.
	- **Acceptance:** validation returns pass/fail and failed-rule list.
- **FR-DU-3 (Future):** Layout-aware extraction and human review workflow for low-confidence pages.

### Admin controls (user mgmt, thresholds, configs)
- **FR-AD-1 (MVP):** Admin shall manage users and assign roles (Retail Manager, Seller/MSME, Admin).
	- **Acceptance:** role change takes effect on next login.
- **FR-AD-2 (MVP):** Admin shall configure thresholds for alerts and guardrails (min margin %, max price delta, stockout horizon).
	- **Acceptance:** updated threshold is applied to the next alert/recommendation run.
- **FR-AD-3 (Future):** Tenant management, SSO, SCIM provisioning.

## AI/ML Requirements

### Model types
- **Demand forecasting:** Prophet/SARIMAX baseline + LightGBM/XGBoost regression with lag/rolling features; optional LSTM/TFT in production.
- **Pricing:** elasticity estimation (regression/GBM), competitor banding, constrained optimization with rule guardrails.
- **Inventory:** service-level reorder point + safety stock using forecast uncertainty; EOQ optional.
- **Promotion uplift:** uplift regression with promo features; causal uplift modeling (future).
- **Customer insights:** rule-based RFM cohorts (MVP); clustering + LTV/churn models (future).
- **Risk/anomaly detection:** residual-based anomaly detection (actual vs forecast), z-score/ESD; Isolation Forest (future).

### Input features required
- Time-series: `date`, `sku_id`, `store_id`, `units_sold`, `revenue`.
- Pricing: `price`, `discount_pct`, `competitor_price`, `cost` (if available).
- Promo: `promo_flag`, `promo_type`, `promo_start/end`, `ad_spend` (optional).
- Inventory: `on_hand`, `in_transit`, `lead_time_days`, `min_order_qty`, `case_pack`.
- Calendar: day-of-week, month, holiday/festival flags.
- Customer (if available): `customer_id`, order frequency, recency, monetary value.

### Output formats
- Forecast output: JSON/CSV with `sku_id`, `store_id`, `date`, `yhat`, `yhat_lower`, `yhat_upper`, `model_version`.
- Pricing output: JSON with `recommended_price`, `expected_margin_uplift_pct`, `guardrail_status`, `reason_codes`.
- Inventory output: JSON with `safety_stock`, `reorder_point`, `reorder_qty`, `service_level`, `risk_bucket`.
- Promotion output: JSON with `baseline_units`, `uplift_pct`, `uplift_units`, `confidence`.
- Anomaly output: JSON with `anomaly_score`, `type`, `explanation`.

### Explainability requirements (why recommendation was given)
- Every recommendation shall include:
	- **Top drivers** (top 3–5 features) OR rule reasons (guardrail triggers)
	- **Confidence** (interval or score)
	- **Evidence** (competitor band, forecast residuals, inventory position)
- For ML-based forecasts, provide SHAP-based feature attribution for top SKUs (MVP: top 100 SKUs; production: configurable).

### Accuracy benchmarks (MAPE for forecast etc.)
- Forecasting: **MAPE ≤ 20%** for top 500 SKUs at weekly horizon; **MAPE ≤ 30%** for long-tail SKUs (if modeled).
- Promotion uplift (MVP): directional accuracy where uplift sign matches actual ≥ **70%** in backtests (or holdout simulation).
- OCR extraction: mandatory invoice fields extracted with **≥ 90% field-level accuracy** on a labeled sample set (minimum 50 docs for demo).

### Drift monitoring requirements
- Compute weekly drift for key features (price, units, promo share) using PSI.
- Trigger alert when PSI > **0.2** for any key feature or when MAPE worsens by **> 10%** over 2 consecutive evaluation windows.
- Store model versions + training data snapshot identifiers for reproducibility.

## Data Requirements

### Required datasets & formats
- **Sales transactions:** CSV/JSON; required columns: date, sku_id, store_id, units_sold, revenue (and optional price).
- **Inventory snapshots:** CSV/JSON; required: date, sku_id, store_id, on_hand, in_transit, lead_time_days.
- **Product master:** CSV/JSON; required: sku_id, category, brand, cost (optional), min_margin_pct (optional).
- **Competitor prices:** CSV/JSON; required: date, sku_id, competitor, competitor_price.
- **Promotions calendar:** CSV/JSON; required: sku_id/category, promo_type, start_date, end_date, discount_pct.
- **Customer/orders (optional for MVP):** CSV/JSON; required: order_id, customer_id, date, sku_id, revenue.
- **Documents:** PDF/JPG/PNG invoices/certificates and marketplace policy PDFs.

### Schema outline (tables/columns)
- `sales_transactions(date, sku_id, store_id, units_sold, revenue, price, promo_flag, channel)`
- `inventory_snapshots(date, sku_id, store_id, on_hand, in_transit, backorder, lead_time_days)`
- `product_master(sku_id, category, brand, pack_size, cost, mrp, min_margin_pct)`
- `competitor_prices(date, sku_id, competitor, competitor_price, source, confidence)`
- `promotions_calendar(entity_type, entity_id, promo_type, start_date, end_date, discount_pct)`
- `customer_orders(date, order_id, customer_id, sku_id, store_id, revenue, units)`
- `recommendations(run_date, sku_id, store_id, rec_type, payload_json, score, status)`
- `documents_metadata(doc_id, doc_type, upload_ts, s3_uri, extracted_json, validation_status, confidence)`
- `chat_logs(conversation_id, user_id, ts, role, message, citations_json)`

### Cleaning rules
- Deduplicate facts by `(date, sku_id, store_id)` for aggregated sources and by `(source_event_id)` for event sources.
- Normalize IDs (`sku_id`, `store_id`) to uppercase and trim whitespace.
- Enforce non-negative units and revenue; quarantine rows violating constraints.
- Forward-fill `price` per SKU-store up to 7 days; mark `imputed_price=true`.
- Cap extreme units using per-SKU percentile caps unless `promo_flag=true`.
- Validate GSTIN with regex; flag invalid.

### Refresh frequency (hourly/daily)
- **MVP:** daily refresh (T+1) for sales/inventory; competitor prices daily; documents near real-time.
- **Future:** hourly or near real-time ingestion for inventory and competitor prices where available.

### Data quality validation checks
- Completeness: ≥ **98%** of rows have non-null `date`, `sku_id`, `store_id`.
- Referential integrity: ≥ **99%** of fact rows join to `product_master` on `sku_id` (else quarantine).
- Freshness: daily pipeline completes by **06:00 local time**.
- Consistency: revenue ≈ units × price within ±2% for sources that provide price.
- Duplicates: duplicate rate < **0.5%** after dedupe.

## Non-Functional Requirements

### Performance (response time)
- Dashboard KPI view p95 **≤ 3s** for a 90-day range and top 500 SKUs.
- Forecast API p95 **≤ 4s** for precomputed forecasts.
- Recommendations API p95 **≤ 5s**.
- Copilot p95 **≤ 6s** (≤ 1 tool call) and **≤ 10s** (≤ 3 tool calls).

### Scalability (number of SKUs/stores)
- **MVP:** 10k SKUs, 100 stores, 20 concurrent users.
- **Future:** 50k SKUs, 500 stores, 200 concurrent users.

### Availability
- **MVP:** best-effort; target ≥ **99.0%** during demo window.
- **Future:** ≥ **99.5%** monthly for core APIs and dashboard.

### Reliability (fallback if AI fails)
- Provide deterministic fallbacks:
	- Forecast fallback: seasonal naive / moving average.
	- Pricing fallback: competitor parity + min margin.
	- Inventory fallback: reorder using moving average demand and fixed safety factor.
- Retries for pipeline tasks with exponential backoff; failed runs produce an operator-visible incident record.

### Maintainability
- Services must expose health endpoints and structured logs.
- Configuration (thresholds, guardrails) stored centrally and versioned.
- Models and features versioned; training artifacts reproducible.

## Security & Privacy Requirements

### RBAC
- Roles: Admin, Retail Manager, Seller/MSME.
- Authorization enforced on all APIs; least-privilege access.

### Encryption at rest/in transit
- TLS 1.2+ for all network traffic.
- AES-256 encryption at rest for object and database storage.

### Audit logs
- Capture: authentication events, data uploads, model overrides, recommendation accept/reject, admin configuration changes.
- Audit logs must be immutable (append-only) in production.

### PII rules
- If customer PII exists, it must be masked/redacted before being used as LLM context.
- Logs must not store raw PII fields.

### Retention policy
- **MVP defaults:** chat logs 30 days; documents 90 days.
- **Future:** configurable per tenant; support deletion requests and legal holds.

## System Requirements / Architecture Needs

### AWS service mapping (S3, Glue, Athena/Redshift, SageMaker, Bedrock, Lambda, API Gateway, CloudWatch)
- **S3:** raw/cleaned/curated zones + document storage.
- **Glue:** ETL + catalog + scheduled jobs.
- **Athena (optional Redshift):** query curated tables powering dashboards.
- **SageMaker:** training jobs + hosted endpoints (or batch inference) for forecasts/recommendations.
- **Bedrock:** LLM for copilot and (optionally) embeddings.
- **Lambda + API Gateway:** ingestion APIs, recommendation APIs, copilot orchestration.
- **CloudWatch:** logs/metrics/alarms; dashboards for operational monitoring.

### API requirements
- REST endpoints with JWT auth:
	- `POST /ingest/*`, `POST /forecast`, `POST /recommendations`, `POST /pricing-suggest`, `POST /promo/impact`, `GET /alerts`, `POST /copilot/chat`, `POST /upload-document`.
- Rate limiting per user/tenant; request validation and input schema enforcement.

### Logging/monitoring requirements
- Correlation ID per request across services.
- Metrics: latency p50/p95, error rate, job duration, model MAPE, drift PSI.
- Alerts:
	- API error rate > 2% (10 min)
	- pipeline failure
	- drift PSI > 0.2

## Assumptions & Constraints
- Hackathon scope uses sample datasets and simplified connectors.
- Competitor price data availability may be limited; MVP supports CSV ingestion.
- Compute budget is constrained; prefer batch pipelines and precomputed outputs.
- Some sellers may not provide cost data; pricing guardrails must work without cost (fallback to MRP/min margin assumptions).

## Risks & Mitigations
- **Low data quality / missing columns:** strict schema validation + quarantine + clear error reports; provide sample templates.
- **Cold start / sparse history for SKUs:** use category-level models and fallback heuristics.
- **LLM hallucinations:** require tool-calling for numeric outputs; require citations for policy claims.
- **OCR inaccuracies:** provide confidence scores and “needs review” status; support manual correction.
- **Over-reliance on recommendations:** keep approvals/manual override workflows; log decisions.

## Milestones (Hackathon plan)
- **Day 1:** Data schemas + ingestion (CSV) + curated tables + basic KPI dashboard.
- **Day 2:** Demand forecasting pipeline + forecast API + evaluation metrics.
- **Day 3:** Pricing + inventory recommendation services + alerts.
- **Day 4:** Document OCR/extraction + compliance validation + copilot tool-calling.
- **Day 5:** Polish UI, run demo scenarios, harden with tests, prepare pitch deck.

## Future Enhancements
- Multi-lingual copilot (Hindi + regional languages) and WhatsApp-first UX.
- Automated procurement workflows and supplier collaboration.
- Promotion optimization (budget allocation) and continuous experimentation.
- Advanced anomaly detection (returns abuse, fraud signals) with investigation workflows.
- Dynamic pricing automation with approvals and A/B testing.
