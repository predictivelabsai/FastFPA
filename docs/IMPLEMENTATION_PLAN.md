# FastFPA implementation plan

Status: same-day synthetic launch candidate implemented
Target: `https://fpa.fastsme.com`
Repository: `predictivelabsai/FastFPA`
Planning date: 2026-07-30

## 1. Executive recommendation

Build FastFPA as an independently deployable FastSME product that sits above
FastERP and other source systems. It must not become another accounting ledger.
Its job is to:

1. ingest actuals and operational drivers;
2. create versioned budgets and rolling forecasts;
3. calculate driver-based plans;
4. compare Actual, Budget, Forecast, and What-if scenarios;
5. coordinate submissions, review, approval, comments, and audit history; and
6. turn the resulting numbers into management dashboards and reports.

The first release should follow the FastERP product philosophy: implement one
coherent workflow deeply rather than reproduce the feature lists of Anaplan,
Adaptive Planning, or OneStream shallowly.

The recommended coherent workflow is:

```text
Import actuals
    → map accounts and dimensions
    → set revenue, workforce, and operating-expense drivers
    → calculate a monthly forecast
    → clone baseline/upside/downside scenarios
    → collect department submissions
    → approve and lock the forecast
    → explain actual-versus-plan variances
    → export a management pack
```

The first deployment will use deterministic synthetic data and SQLite like the
other thin FastSME applications. Release 1 will nevertheless implement full
integrated financial statements: P&L, balance sheet, and indirect cash flow.
The domain and persistence interfaces should be tenant-aware and
PostgreSQL-ready from the first implementation. A real hosted multi-user
finance service should move to PostgreSQL before onboarding customer data.

### Confirmed product decisions

- `fpa.fastsme.com` initially hosts synthetic data, not customer financials;
- Release 1 includes integrated P&L, balance sheet, and cash flow;
- the product serves SMEs, fractional CFOs, and mid-market finance teams;
- Release 1 integrates FastERP, FastHRM, and FastCRM;
- Release 1 uses one group currency with multiple departments/business units;
- FastOffice is the preferred suite entry point, with standalone authentication
  preserved;
- FastSheets/FastOffice import and export are sufficient; no Excel add-in is
  required for launch;
- AI is explanatory and drafting-only;
- the deployed application uses the suite's existing xAI secret-management
  convention;
- the primary synthetic model mirrors FastERP, followed by a reusable
  recurring-revenue model;
- FastInsights is the advanced visualisation and governed exploration target;
- deep teal branding, MIT licensing, and necessary sister-repository changes
  are approved.

## 2. Product boundary

### FastFPA owns

- planning calendars, periods, versions, scenarios, and forecast horizons;
- planning dimensions and hierarchies;
- mappings from source-system accounts/dimensions into planning dimensions;
- assumptions, operational drivers, calculation rules, and dependencies;
- budget and forecast values;
- scenario cloning, comparison, locking, and publication;
- variance calculations and deterministic decompositions;
- submissions, review, approval, comments, and audit events;
- planning dashboards, statements, report packs, and exports;
- FP&A-specific API contracts and grounded finance-assistant tools.

### FastFPA consumes

- FastERP actuals: accounts, GL/report balances, invoices, expenses, projects,
  customers, business units, and currencies;
- FastHRM workforce data: employees, departments, payroll, start/end dates, and
  compensation inputs;
- FastCRM commercial drivers: pipeline, expected close dates, probability,
  product/service mix, and customer segments;
- FastInsights for advanced cross-product BI and governed exploration;
- FastSheets for governed spreadsheet handoff, not as FastFPA's source of
  truth;
- CSV/XLSX files for universal onboarding and offline exchange.

### FastFPA does not own

- journals, payments, bank reconciliation, tax filing, AP/AR, or other
  transactional accounting;
- employee master-data administration or payroll processing;
- CRM pipeline administration;
- unconstrained arbitrary spreadsheet formulas;
- general-purpose BI or a general SQL workbench;
- production identity-provider brokering.

The integration boundary is a versioned HTTP/file contract. FastFPA must not
open another product's SQLite file or import another product's domain module.

## 3. Target users and permissions

| Role | Primary jobs | Initial permissions |
|---|---|---|
| Owner/Admin | Configure organisation, integrations, calendar, dimensions | Full configuration and access |
| CFO | Set targets, compare scenarios, approve and publish | Read/write all plans; approve/lock |
| Controller/FP&A | Import actuals, build models, run forecasts, analyse variances | Full planning and reporting |
| Department owner | Enter assumptions and budget for assigned scope | Read/write assigned departments |
| Executive/Board viewer | Consume approved dashboards and packs | Read published versions only |
| Auditor | Inspect lineage, changes, approvals, and exports | Read plus audit history |
| Integration service | Import source data through scoped credentials | No interactive access |

Every domain record and query should carry `org_id`. Department/entity scope
must be enforced server-side; hiding a navigation item is not authorization.

The same product supports all three target segments through progressive
capability:

- an SME gets a guided company workspace and standard model;
- a fractional CFO gets a client/organisation portfolio, reusable templates,
  controlled switching, and consolidated workflow status without mixing data;
- a mid-market team gets scoped department ownership, richer dimensions,
  approvals, lineage, and larger PostgreSQL-backed datasets.

## 4. Representative first-release story

Seed a synthetic company whose dimensions line up with the FastERP demo:

- calendar-year monthly planning;
- group currency GBP;
- business units for UK, Europe, and North America;
- departments such as Sales, Delivery, Product, G&A, and Finance;
- product/service revenue streams;
- projects and customers;
- 18 months of monthly actuals and a 12-month forecast horizon;
- opening balance-sheet balances, working-capital history, capex, debt, tax,
  and cash assumptions;
- Baseline, Upside, and Downside scenarios.

The demo should support this board-level question:

> Revenue is below plan in Europe. If pipeline conversion falls by five points,
> hiring slips by two months, and discretionary spend is reduced by 8%, what
> happens to revenue, EBITDA, cash, and department budgets over the next four
> quarters?

The user should be able to:

1. inspect the latest actual-data refresh and mapping quality;
2. open the Baseline forecast;
3. see revenue as volume multiplied by price;
4. see workforce expense as active headcount multiplied by compensation,
   employer costs, and start-date timing;
5. see operating expenses as direct inputs or percentages of a driver;
6. clone Baseline into Downside without mutating the source version;
7. change the three assumptions and recalculate;
8. compare both scenarios by month, quarter, department, and account;
9. review deterministic price/volume/rate/timing variance bridges;
10. inspect the linked P&L, balance sheet, and cash-flow effects;
11. submit and approve the chosen forecast; and
12. export a management pack with source/version/as-of metadata.

After that primary flow is stable, the seed/template library should demonstrate
a business-agnostic recurring-revenue model with opening customers, new
business, expansion, contraction, churn, average recurring revenue, and
recognised revenue. It should map those drivers into the same governed
financial statements rather than creating a separate SaaS-only application.

## 5. Scope by release

### Release 1: focused FP&A demonstrator

- anonymous FastSME landing and authenticated application shell;
- Google SSO, verified local accounts, and FastOffice suite-ticket callback;
- organisations, users, roles, and department/entity scope;
- monthly fiscal calendar;
- account, entity, department, product, customer, project, and currency
  dimensions;
- deterministic synthetic actuals and operating drivers;
- CSV import, preview, validation, mapping, commit, and rollback;
- FastERP, FastHRM, and FastCRM adapters with fixture-backed contract tests;
- Actual, Budget, Forecast, Baseline, Upside, and Downside versions/scenarios;
- version cloning, working/published/locked states, and immutable snapshots;
- structured driver rules:
  - direct input;
  - constant and prior-period carry-forward;
  - growth over prior period;
  - price multiplied by volume;
  - headcount multiplied by salary and employer cost;
  - percentage of another metric;
  - simple allocation by fixed weights;
  - opening balance plus movements;
  - working-capital days;
  - depreciation schedule;
  - debt and interest schedule;
- deterministic dependency graph, cycle detection, calculation runs, and
  explainable lineage;
- integrated monthly P&L, balance sheet, and indirect cash flow;
- revenue, payroll, opex, working-capital, capex, depreciation, debt, interest,
  tax, and retained-earnings schedules;
- department budget templates;
- rolling forecast;
- actual-versus-budget and actual-versus-forecast dashboards;
- revenue price/volume and cost rate/volume variance bridges;
- comments, submit/review/approve/reject/lock workflow;
- dashboards and CSV/XLSX/PDF-ready report data;
- FastSheets workbook export/import contract;
- FastOffice product registration, launch, report-artifact, and read-adapter
  contract;
- FastInsights analytical feed and `Open in FastInsights` handoff;
- primary FastERP-aligned seed model plus a recurring-revenue model template;
- public developer documentation, OpenAPI, and read-only synthetic-demo API;
- grounded assistant for explanations and scenario drafting;
- Docker, health endpoint, tests, Coolify definition, and production smoke
  checks.

### Release 1.1: operational completeness

- scheduled source imports and incremental cursors;
- data-quality dashboard and reconciliation controls;
- configurable report layouts and saved views;
- richer XLSX round trips with stable templates;
- management-pack PDF and PowerPoint generation;
- email/in-app workflow notifications;
- alerts for threshold breaches and stale source data;
- forecast accuracy tracking and bias metrics;
- reusable model templates;
- webhook/outbox events for FastOffice/FastInsights;
- performance and concurrency work needed for PostgreSQL.

### Release 2: advanced planning

- advanced treasury, covenant, tax, dividend, and financing scenarios;
- multi-entity consolidation and eliminations;
- multi-currency translation and FX scenarios;
- weekly/daily or 4-4-5 calendars;
- flexible allocations and step-down allocation methods;
- driver sensitivity tables and Monte Carlo simulation;
- statistical forecasting with backtesting and confidence intervals;
- row/column-level report designer;
- Excel or Google Sheets add-in after the API and model contracts stabilise;
- scoped, short-lived user/service tokens and customer-grade integration
  administration.

### Explicitly deferred

- a general-purpose spreadsheet engine;
- real-time multi-cursor workbook collaboration;
- arbitrary Python/SQL execution from planning models;
- autonomous AI changes to approved plans;
- Kubernetes, Airflow, dbt, ClickHouse, or a separate OLAP service before load
  proves they are needed;
- parity with every enterprise connected-planning category in the first
  release.

## 6. Recommended architecture

### 6.1 Runtime shape

Use the thin FastSME application pattern:

```text
Browser
  │
  ├── FastHTML + HTMX application
  │     landing, planning UI, workflow, reports, assistant
  │
  └── same-process FastAPI mounted at /api
        integration and developer contract
                   │
                   ▼
           FP&A domain services
       ┌───────────┼────────────┐
       ▼           ▼            ▼
  planning      calculation   ingestion
  service       engine        adapters
       └───────────┼────────────┘
                   ▼
         SQLite demo / PostgreSQL hosted
                   │
       ┌───────────┼────────────┐
       ▼           ▼            ▼
    FastERP     FastHRM       FastCRM
      API        API           API
```

Recommended initial dependencies:

- Python 3.12;
- FastHTML and HTMX for server-rendered interaction;
- FastAPI for `/api/v1`;
- Plotly for charts;
- `httpx` for source adapters;
- `pydantic` or explicit dataclasses for validated contracts;
- `openpyxl` for Release 1 XLSX exchange;
- `pytest` for tests;
- `Decimal` for rates and driver calculations;
- SQLite in WAL mode for local/demo deployment;
- PostgreSQL through a repository interface for hosted multi-user operation.

Do not introduce pandas as the core calculation engine. It can be an import or
export helper later, but domain calculations need explicit validation,
lineage, deterministic ordering, and predictable Decimal behavior.

### 6.2 Suggested repository structure

```text
FastFPA/
  AGENTS.md
  README.md
  SKILLS.md
  LICENSE
  requirements.txt
  .env.sample
  .env.coolify.sample
  Dockerfile
  docker-compose.yml
  web_app.py
  api_app.py
  db.py
  seed.py
  fpa/
    models.py
    repositories.py
    planning.py
    calculations.py
    dependencies.py
    scenarios.py
    variances.py
    workflow.py
    reports.py
    permissions.py
    audit.py
    imports/
      contracts.py
      csv_adapter.py
      fasterp_adapter.py
      fasthrm_adapter.py
      fastcrm_adapter.py
  web/
    layout.py
    landing.py
    views/
      dashboard.py
      actuals.py
      planning.py
      scenarios.py
      variance.py
      workflow.py
      reports.py
      settings.py
    charts.py
    forms.py
    ai.py
    account_auth.py
    google_auth.py
    suite_auth.py
    api.py
    api_core.py
    developer.py
  scripts/
    coolify.py
    sync_sources.py
    export_management_pack.py
    generate_openapi.py
    build_demo_gif.sh
  tests/
    unit/
    integration/
    contract/
    e2e/
  docs/
    IMPLEMENTATION_PLAN.md
    ROADMAP.md
    DATA_MODEL.md
    INTEGRATIONS.md
    SECURITY.md
```

`web_app.py` should own route wiring and startup, not business rules. `db.py`
should own schema setup and low-level connections, while domain services own
transactions and invariants. This improves on FastERP's current single large
database module without breaking the suite's recognisable structure.

## 7. Domain and data model

### 7.1 Core records

Identity and tenancy:

- `organizations`;
- `users`;
- `memberships`;
- `role_assignments`;
- `data_scopes`.

Planning structure:

- `fiscal_calendars`;
- `periods`;
- `dimensions`;
- `dimension_members`;
- `dimension_hierarchy_edges`;
- `accounts`;
- `metrics`;
- `currencies`;
- `exchange_rates`.

Planning lifecycle:

- `planning_cycles`;
- `versions`;
- `scenario_assumptions`;
- `models`;
- `model_rules`;
- `rule_dependencies`;
- `calculation_runs`;
- `calculation_results`;
- `version_snapshots`.

Financial schedules:

- `working_capital_assumptions`;
- `capex_projects`;
- `fixed_asset_schedules`;
- `debt_facilities`;
- `debt_schedule_rows`;
- `tax_assumptions`;
- `cash_flow_adjustments`.

Values and lineage:

- `fact_values`;
- `assumption_values`;
- `value_lineage`;
- `source_systems`;
- `source_mappings`;
- `import_runs`;
- `import_rows`;
- `reconciliation_results`.

Collaboration and governance:

- `submissions`;
- `approvals`;
- `comments`;
- `attachments`;
- `audit_events`;
- `saved_views`;
- `report_definitions`;
- `export_runs`.

### 7.2 Value grain

Release 1 uses monthly facts with core coordinates:

```text
org
version
period
account/metric
entity
department
product
customer
project
currency
```

The table may carry a validated `extra_dimensions_json` plus a stable
`coordinate_hash` for extension, but core reporting dimensions should remain
typed columns with indexes and foreign keys.

Financial amounts should be stored as integer minor units, accompanied by
currency. Rates, quantities, percentages, and per-unit values should use
validated decimal strings and Python `Decimal`, never binary floating-point
for authoritative calculations.

Required invariants:

- one authoritative value per version/period/coordinate/measure;
- all referenced members belong to the same organisation;
- Actual versions are imported and not manually edited after commit;
- locked/published versions are immutable;
- scenario clones retain a source-version reference;
- every calculated value identifies its rule, input set, and calculation run;
- every import records source, timestamp, checksum, mapping version, row
  counts, rejects, and reconciliation result;
- every workflow and model mutation produces an audit event.

### 7.3 Version semantics

Use distinct concepts:

- **cycle**: FY2027 Budget or 2026 Q3 Forecast;
- **version**: a concrete dataset within a cycle;
- **scenario**: Baseline, Upside, Downside, Acquisition, etc.;
- **status**: Draft, In Review, Approved, Published, Locked, Archived;
- **snapshot**: immutable values and configuration for reproducibility.

Users clone versions/scenarios instead of editing approved results in place.
Publishing creates a snapshot; reopening requires a privileged action and an
audited successor version.

## 8. Calculation engine

### 8.1 Rule types

Start with a structured rule DSL represented as validated records, not arbitrary
code:

```text
input                     user/import supplies value
constant                  fixed value over selected periods/scope
carry_forward             prior-period value
growth                    prior-period value × (1 + growth rate)
multiply                  driver A × driver B
ratio                     driver A × configured percentage
sum                       roll up child accounts/members
allocation                source amount × validated weights
headcount_cost            active FTE × salary × employer-cost factors
opening_plus_movements     opening balance + additions - reductions
working_capital_days       revenue/cost driver × days ÷ calendar basis
depreciation               capex basis ÷ useful life with start convention
debt_schedule              opening debt + draws - repayments + interest
cash_roll_forward          opening cash + calculated net cash movement
```

Rules operate over explicit member scopes and periods. All formulas are parsed
into typed operations. No `eval`, executable Python, or unrestricted SQL is
allowed.

### 8.2 Execution

For each calculation:

1. validate the version is editable;
2. resolve applicable rules and dimension scope;
3. build a directed dependency graph;
4. reject missing references and cycles;
5. topologically sort rules;
6. calculate using Decimal and a stable period/member ordering;
7. validate roll-ups and allocation weights;
8. write results in one transaction;
9. record input/configuration hashes and timings;
10. expose row-level lineage and an explanation trace.

The same input snapshot and rule set must always produce the same output.
Calculation runs should be idempotent and safe to retry.

### 8.3 Variance engine

Deterministic math comes before AI narrative:

- actual minus budget/forecast, amount and percentage;
- favourable/unfavourable semantics by account type;
- revenue price/volume decomposition;
- payroll rate/headcount/start-date decomposition;
- operating-cost rate/volume decomposition;
- mix variance when product/customer mix exists;
- monthly, quarterly, year-to-date, full-year, and run-rate views;
- materiality thresholds.

The assistant may explain these outputs, but it must cite the underlying report
rows and must not invent a decomposition.

## 9. Ingestion and integration

### 9.1 Common adapter contract

Every adapter should implement:

```text
discover_schema()
test_connection()
extract(cursor, date_range)
normalise()
validate()
preview()
commit()
reconcile()
```

Import is a staged transaction:

```text
uploaded/fetched
    → parsed
    → validated
    → mapped
    → previewed
    → committed
    → reconciled
```

Failed rows remain visible with actionable messages. A failed run must not
partially alter an Actual version.

### 9.2 FastERP

The current FastERP API already exposes accounts, customers, invoices,
expenses, projects, profit-and-loss, and trial-balance data. Monthly planning
will need a small, separately reviewed FastERP contract extension:

- GL entries or period balances;
- business units and currencies;
- `from`/`to` filters on financial reports;
- stable source IDs and update timestamps;
- pagination/cursors for incremental sync;
- authenticated non-demo access.

FastFPA should ship a deterministic internal fixture first, then test the
adapter against FastERP's committed OpenAPI snapshot. FastERP changes should
be a separate repository PR and deployment.

### 9.3 FastHRM

The workforce adapter should import departments, employee status, start/end
dates, FTE, base salary, employer-cost assumptions, and payroll actuals.
Sensitive fields need an explicit contract and scope; FastFPA does not need
personal attributes unrelated to workforce cost.

The current FastHRM API exposes employees and departments, while its payroll
table is not exposed and its employee model has joining date/base salary but no
termination date or FTE. Add a scoped payroll read contract and the minimum
planning fields needed for workforce cost. Employer-cost rates can remain
FastFPA assumptions instead of expanding the HR master unnecessarily.

### 9.4 FastCRM

The commercial adapter should aggregate opportunities by expected close month,
probability, owner/team, segment, product, and currency. Release 1 converts
weighted pipeline into a revenue-driver proposal without silently replacing an
approved forecast.

The current FastCRM opportunity resource already carries value, probability,
stage, industry, owner, and expected close. Product/service and currency are
not first-class deal fields, so add them through a separately reviewed schema/
API extension or a controlled FastFPA mapping before using them as forecast
dimensions.

### 9.5 FastSheets

FastSheets is the governed workbook exchange target. Release 1 should support:

- export of selected plan/report grids into a labelled workbook;
- source organisation, version, scenario, as-of date, and coordinate metadata;
- import of permitted input cells into a new draft/change set;
- formulas treated as presentation or proposals, never as a bypass around the
  FastFPA calculation engine;
- explicit review before imported values affect a working version.

The current FastSheets API can create workbooks and read cells, but it cannot
atomically populate an exported workbook's cells. Add a separately reviewed,
token-gated bulk workbook import endpoint with idempotency, size limits,
formula safety, ownership, and audit metadata. Do not write the FastSheets
SQLite database directly.

### 9.6 FastInsights

FastFPA owns its essential planning dashboards and statement views. FastInsights
provides optional advanced visualisation, governed exploration, saved queries,
and cross-functional dashboards.

Release 1 should expose a stable analytical feed of statement, variance,
driver, scenario, workflow, and forecast-accuracy facts. `Open in
FastInsights` should create or open a governed dataset/dashboard that retains a
deep link back to the authoritative FastFPA report.

The current FastInsights API reads queries, charts, dashboards, and warehouse
facts, but only dashboards have a generic write surface. Add a separate
tenant-safe contract for external datasets and idempotent query/chart/dashboard
bundles. FastInsights should pull through the FastFPA API or an exported
snapshot; it must not share the FastFPA database.

### 9.7 FastOffice

Register FastFPA in the FastOffice product catalogue and ticket issuer with
audience `fpa`. Add a typed read adapter for plans, scenarios, statements,
variances, workflow status, and canonical links. Management reports can be
published as typed FastOffice artifacts and handed to FastSheets, FastDocs, or
FastSlides without making those products the planning source of truth.

FastPilot may explain results and draft a scenario change set. It may not apply
the draft, approve workflow, or modify a locked version.

### 9.8 Files

CSV and XLSX are required in Release 1 with:

- downloadable templates;
- stable column identifiers;
- a mapping wizard;
- locale-aware date and number parsing;
- duplicate detection;
- row-level validation;
- dry-run preview;
- import checksum and audit log;
- export metadata sheet.

## 10. User interface and routes

Anonymous `/` should render a white, restrained FastSME landing page with a
top-right Sign In. Authenticated `/` should render the FP&A dashboard.

Recommended navigation:

| Area | Routes | Purpose |
|---|---|---|
| Dashboard | `/` | KPIs, forecast outlook, risks, workflow, refresh status |
| Portfolio | `/portfolio` | Client/organisation status for fractional CFOs |
| Actuals | `/actuals`, `/actuals/imports/{id}` | Refresh, map, validate, reconcile |
| Plans | `/plans`, `/plans/{id}` | Budget/forecast grid and drivers |
| Models | `/models`, `/models/{id}` | Structured rules and lineage |
| Scenarios | `/scenarios`, `/scenarios/compare` | Clone, change assumptions, compare |
| Variance | `/variance` | Actual vs plan and decomposition bridges |
| Reports | `/reports`, `/reports/{id}` | P&L, department, KPI, management pack |
| Workflow | `/workflow`, `/submissions/{id}` | Assign, submit, review, approve |
| Integrations | `/integrations`, `/imports` | Sources, mappings, run history |
| Settings | `/settings/*` | Calendar, dimensions, access, thresholds |
| Developers | `/developers`, `/api/docs` | API documentation |

The planning grid should be purpose-built:

- dimensions and periods are explicit;
- actual periods are visually locked;
- input, imported, and calculated cells are distinguishable;
- every value has a lineage/details action;
- bulk paste and keyboard navigation are supported incrementally;
- filters and saved views are server-authoritative;
- calculations happen server-side;
- totals and variance semantics remain accessible without relying on colour.

Recommended landing identity:

- eyebrow: `Financial planning & analysis`;
- headline: `Plan forward with numbers everyone can trust.`;
- description: `Connect actuals, drivers, budgets, forecasts, scenarios, and
  performance reviews in one governed planning workspace.`;
- features:
  - `Driver-based budgets and forecasts`;
  - `Scenarios and variance analysis`;
  - `Approvals, lineage, and management reporting`;
- proposed accent: deep teal `#0f766e`;
- proposed tint: `#f0fdfa`.

The final wording and palette must be registered in the FastSME landing
portfolio source of truth, not copied ad hoc.

## 11. Reporting

Release 1 reports:

- monthly/quarterly/annual P&L;
- monthly/quarterly/annual balance sheet;
- monthly/quarterly/annual indirect cash-flow statement;
- working-capital, capex/depreciation, debt/interest, tax, and cash schedules;
- three-statement integrity and cash roll-forward checks;
- Actual vs Budget vs Forecast;
- Baseline vs selected scenario;
- department budget and forecast;
- revenue by business unit/product/customer;
- workforce/headcount and payroll cost;
- operating expense by department/account;
- EBITDA bridge;
- top favourable/unfavourable variances;
- forecast accuracy and stale-data indicators;
- workflow completion status.

Every report must display:

- organisation;
- version/scenario;
- actual-data as-of timestamp;
- currency and units;
- filters;
- generated timestamp;
- draft/published status.

Drill-down should move from statement line to dimensions, source values, and
calculation lineage without exposing another tenant's data.

## 12. API

Match the FastSME API surface:

- `/developers`;
- `/api/`;
- `/api/v1/health`;
- `/api/docs`;
- `/api/redoc`;
- `/api/openapi.json`;
- `/swagger.json`.

Initial resources:

- `/api/v1/calendars`;
- `/api/v1/dimensions`;
- `/api/v1/accounts`;
- `/api/v1/cycles`;
- `/api/v1/versions`;
- `/api/v1/scenarios`;
- `/api/v1/assumptions`;
- `/api/v1/values`;
- `/api/v1/calculation-runs`;
- `/api/v1/import-runs`;
- `/api/v1/submissions`;
- `/api/v1/reports/profit-and-loss`;
- `/api/v1/reports/balance-sheet`;
- `/api/v1/reports/cash-flow`;
- `/api/v1/reports/financial-schedules`;
- `/api/v1/reports/variance`;
- `/api/v1/reports/scenario-comparison`.

Synthetic demo reads can follow the current public-read fleet convention.
Customer/tenant data must never be anonymously readable. Production writes
need scoped user/service tokens, idempotency keys, audit events, and optimistic
concurrency. A single deployment-wide bearer token is acceptable only for the
suite's current synthetic integration preview, not as the final tenant API.

Commit `swagger.json` and fail CI when the runtime schema drifts.

## 13. Authentication, tenancy, and security

Implement the current suite-compatible paths:

- standalone Google OIDC at `/auth/google` and
  `/auth/google/callback`;
- verified local accounts and Postmark email;
- FastOffice handoff at `/auth/suite/callback`, audience `fpa`;
- host-only, Secure, HTTP-only sessions in production;
- preserved standalone login for self-hosted use.

The FastOffice ticket pattern is useful for suite consistency, but the long-term
identity target should be standards-based OIDC with key rotation rather than a
per-service shared HMAC secret.

Finance-specific controls:

- CSRF protection on mutations;
- tenant and scope checks inside domain services and queries;
- immutable published snapshots;
- dual-control option for approval and reopening;
- no secrets or financial values in routine logs;
- request IDs and security/audit events;
- upload size/type limits and spreadsheet formula-injection protection;
- outbound request allowlist for connectors;
- timeouts and retry limits;
- encrypted connector credentials;
- rate limits on authentication, imports, calculation, export, and AI;
- backups plus tested restore before real customer data;
- retention and deletion policy;
- no real data in deterministic seed fixtures or screenshots.

## 14. AI assistant

Release 1 AI is useful but non-authoritative:

- answer questions from a bounded planning snapshot;
- explain deterministic variances with report citations;
- summarise scenario differences;
- identify missing assumptions and stale actuals;
- draft management commentary;
- propose, but not apply, a scenario change set.

Slash commands should work without an API key:

```text
/outlook
/variance
/scenario baseline downside
/cash
/workflow
```

Initial tools are read-only. If write tools are later added, they create a
reviewable draft and require explicit confirmation. AI may never alter a
published/locked version, approve a submission, run arbitrary SQL, or bypass
scope checks.

Provider configuration follows the fleet pattern (`MODEL_PROVIDER`,
`MODEL_NAME`, optional provider keys). Core planning remains fully functional
without an LLM key. The hosted deployment will declare `XAI_API_KEY` as a
required Coolify secret because the owner confirmed the suite key is available.
Its value must be configured through the existing secret store; it must never
be copied into source, samples, logs, command arguments, or this repository.

## 15. FastDevOps and `fpa.fastsme.com`

Add a `fastfpa` service to `FastDevOps/config/services.yaml` only after the
application has a working container and health check.

Proposed catalog entry:

```yaml
fastfpa:
  description: FastFPA financial planning and analysis
  repo: predictivelabsai/FastFPA
  local_dir: FastFPA
  port: 5018
  domain: https://fpa.fastsme.com
  health: {path: /healthz}
  volume: /data
  env:
    required:
      - XAI_API_KEY
      - GOOGLE_CLIENT_ID
      - GOOGLE_CLIENT_SECRET
      - POSTMARK_API_TOKEN
      - FASTOFFICE_SSO_SECRET
    runtime:
      FASTFPA_ENV: production
      FASTFPA_PORT: "5018"
      FASTFPA_DB: /data/fastfpa.sqlite
      FASTFPA_PUBLIC_URL: https://fpa.fastsme.com
      MODEL_PROVIDER: xai
      GOOGLE_REDIRECT_URI: https://fpa.fastsme.com/auth/google/callback
      FASTSME_AUTH_DB: /data/fastsme-accounts.sqlite
      FROM_EMAIL: info@fastsme.com
```

The required-secret list should include only secrets consumed by the
implemented release. `XAI_API_KEY` is confirmed for the hosted FastFPA
deployment, but its value remains independently configured in Coolify. A
self-hosted deployment may omit it and retain all deterministic planning
features.

Deployment sequence:

1. add `.env.sample`, `.env.coolify.sample`, Dockerfile, volume, and `/healthz`;
2. make the app listen on `0.0.0.0:5018`;
3. include `curl` or `wget` if the Coolify health wrapper requires it;
4. add the FastFPA portfolio record and FastSME product catalogue card;
5. add the FastDevOps service and run `python cli.py validate`;
6. run `python cli.py doctor fastfpa`;
7. create the Coolify application from the repository Dockerfile;
8. attach persistent storage at `/data`;
9. add secret values in Coolify without exposing or committing them;
10. create/verify DNS for `fpa.fastsme.com`;
11. configure exactly one automatic deploy path:
    `main push → GitHub webhook → Coolify`;
12. register the exact Google callback URI;
13. deploy the exact pushed commit;
14. verify `/`, `/healthz`, auth redirects, `/developers`, API schema, static
    assets, TLS, console/network errors, desktop/mobile layouts, and one
    authenticated planning flow;
15. record rollback target and smoke procedure.

No Coolify, DNS, GitHub, OAuth, or production mutation is part of this planning
document. Those steps require separate implementation/deployment authority.

## 16. Testing strategy

### Unit tests

- Decimal and minor-unit conversions;
- fiscal-period generation;
- dimension hierarchy validation;
- rule parsing and validation;
- dependency sorting and cycle rejection;
- each calculation rule;
- allocation weights and rounding;
- working-capital, fixed-asset, debt, tax, retained-earnings, and cash schedules;
- balance-sheet and cash-flow statement assembly;
- scenario cloning and immutability;
- favourable/unfavourable variance semantics;
- price/volume and rate/volume decompositions;
- role/scope decisions;
- import parsing, mappings, and reconciliation.

### Property/invariant tests

- roll-ups equal their children;
- allocation outputs equal the source after controlled rounding;
- assets equal liabilities plus equity for every calculated period;
- indirect cash flow equals the change in cash for every calculated period;
- fixed assets, debt, and retained earnings roll forward without unexplained
  movements;
- calculation order does not change results;
- identical snapshots produce identical results;
- locked versions cannot change;
- no cross-organisation IDs can be joined or queried;
- failed imports make no committed changes;
- published reports reconcile to stored values;
- all audit-sensitive mutations generate events.

### Integration and contract tests

- temporary SQLite database and migration from empty;
- PostgreSQL repository contract before hosted migration;
- CSV preview/commit/rollback;
- FastERP/FastHRM/FastCRM adapter fixtures;
- FastSheets bulk-workbook, FastInsights dataset, and FastOffice artifact/
  read-adapter fixtures;
- OpenAPI snapshot;
- auth, suite-ticket replay, and role boundaries;
- API idempotency and concurrency behavior;
- export escaping and formula-injection protection.

### Browser tests

- anonymous landing and keyboard-accessible Sign In;
- local/Google/suite auth boundaries without storing credentials in fixtures;
- plan creation, driver change, calculation, and lineage;
- scenario clone and compare;
- department submit and CFO approve;
- variance drill-down;
- import validation errors;
- three-statement integrity view;
- report export to FastSheets/FastOffice and `Open in FastInsights`;
- desktop and mobile navigation;
- no unexpected console errors or failed static requests.

### Performance targets for the demonstrator

- dashboard/report response under 500 ms after warm-up on seeded data;
- recalculation under two seconds for the standard seed model;
- at least 250,000 fact values and 100 concurrent read requests in a repeatable
  benchmark before calling the data layer production-ready;
- import and calculation progress visible when work exceeds one second.

Targets must be re-baselined against the production database and hardware.

## 17. Delivery phases and exit criteria

### Same-day public demonstrator — 2026-07-30

The five-engineer launch cut is deliberately narrower than the hardened
Release 1 programme below. It is implemented as five parallel ownership lanes:

1. financial kernel and reconciliation invariants;
2. planner UI, scenario controls, recurring-revenue view, and workflow;
3. FastERP/FastHRM/FastCRM adapters, API, FastOffice SSO, and Google OIDC;
4. Docker, Coolify, secrets, persistence, health, and deployment controls; and
5. tests, browser QA, management reporting, catalogue, and launch verification.

The launch cut includes deterministic synthetic data, four editable planning
scenarios plus locked actuals, full linked statements, variance analysis,
department workflow, audit history, CSV exchange, suite links, explanatory
xAI, committed OpenAPI, and a persistent SQLite deployment. It intentionally
uses fixture fallback when a sister API is unavailable and does not claim
tenant-ready customer-data hosting.

Exit: tests and container checks pass, both financial controls reconcile to
zero, exact commits are pushed, Coolify is healthy, the public catalogue and
FastOffice launcher are current, and DNS/TLS production smoke checks pass.

### Phase 0: decisions and contracts — 2 to 4 days

- record the approved decisions in section 21;
- write product brief, terminology, and Release 1 acceptance story;
- finalise account hierarchy, dimensions, calendar, currency, and seed company;
- write API/adapter contracts and three-statement calculation invariants;
- define the synthetic SQLite deployment and PostgreSQL repository contract.

Exit: a signed-off scope and data model with no unresolved decision that would
invalidate the persistence or calculation design.

### Phase 1: suite foundation — 4 to 6 days

- scaffold the FastHTML/FastAPI repository;
- add configuration, migrations, seed, shell, landing, auth, health, API docs,
  tests, Docker, and local Compose;
- make every seeded domain record organisation-aware;
- add FastFPA to the landing portfolio in its own reviewed change.

Exit: anonymous and authenticated shells, health, auth, seed, tests, and
container all work locally.

### Phase 2: planning and financial kernel — 12 to 18 days

- implement calendar, dimensions, accounts, cycles, versions, scenarios,
  values, assumptions, rule DSL, dependency graph, Decimal calculations,
  lineage, snapshots, and audit events;
- implement linked working-capital, capex/depreciation, debt/interest, tax,
  retained-earnings, and cash schedules;
- enforce balance-sheet balance and cash-flow roll-forward controls;
- build unit/property tests before the full UI.

Exit: the full seed model calculates deterministically; cycles fail closed;
published versions are immutable; P&L, balance sheet, and cash flow reconcile.

### Phase 3: planning experience — 8 to 12 days

- build dashboard, planning grid, driver editor, version/scenario management,
  clone/recalculate/compare flows, saved filters, and lineage views;
- implement bulk edit/paste in bounded scope.

Exit: a planner can complete the representative scenario story without direct
database access.

### Phase 4: actuals, full financials, variance, and reporting — 12 to 16 days

- implement staged CSV imports, mapping, reconciliation, actual locking;
- implement three-statement reports, financial schedules, actual-versus-plan,
  scenario comparison, deterministic variance bridges, charts, drill-down,
  FastSheets exchange, and initial management-pack exports;
- implement and validate the recurring-revenue template after the
  FastERP-aligned model.

Exit: imported and seeded actuals reconcile, and every displayed variance
drills to its calculation/source lineage; both model templates produce
balanced statements and cash roll-forwards.

### Phase 5: workflow and assistant — 5 to 8 days

- implement assignments, comments, submit/review/reject/approve/lock;
- enforce role and scope rules;
- add notifications hooks;
- add read-only grounded assistant and keyless slash commands.

Exit: department-to-CFO workflow is auditable and the assistant cannot exceed
the user's data scope.

### Phase 6: integration and fleet contract — 8 to 12 days

- implement FastERP adapter against fixtures and the agreed API extension;
- implement FastHRM and FastCRM adapters against fixtures and agreed API
  extensions;
- implement the FastSheets bulk-workbook, FastInsights analytical-dataset, and
  FastOffice read/artifact integration contracts in separate repositories;
- commit OpenAPI, developer page, coolify launcher, docs, and demo capture;
- add service/portfolio catalogue entries in separate repositories.

Exit: contract tests pass and source refresh failure is safe, visible, and
retryable.

### Phase 7: hardening and deployment — 4 to 7 days

- security/accessibility/performance review;
- test backup/restore and migration;
- verify Docker health behavior;
- provision `fpa.fastsme.com`, webhook, OAuth callback, volume, and secrets;
- deploy and perform production smoke tests against the exact commit.

Exit: all release gates pass, the deployed commit is known, TLS/auth/health
work, and rollback is documented.

Estimated Release 1 effort:

- approved synthetic three-statement demonstrator with all named integrations:
  roughly 12 to 16 engineer-weeks;
- tenant-ready hosted MVP with PostgreSQL and hardened connectors: roughly 18
  to 26 engineer-weeks;
- advanced multi-entity, multi-currency consolidation remains a separate
  programme.

These are sequencing estimates for the hardened product after the public
demonstrator, not commitments for the same-day launch cut.

## 18. Suggested implementation PR sequence

1. Product brief, terminology, plan, AGENTS, licence, and roadmap.
2. Application skeleton, settings, health, migrations, seed harness, tests.
3. Anonymous landing, authenticated shell, Google/local/suite auth.
4. Organisation, role, and data-scope enforcement.
5. Calendar, dimensions, accounts, metrics.
6. Cycles, versions, scenarios, clone/lock/snapshot.
7. Fact and assumption value store with lineage.
8. Rule DSL and dependency validation.
9. Calculation execution and reconciliation tests.
10. Seeded integrated P&L, balance-sheet, and cash-flow model.
11. Planning grid and driver editing.
12. Scenario comparison.
13. CSV import, mapping, preview, commit, reconciliation.
14. Working-capital, capex, debt, tax, retained-earnings, and cash schedules.
15. Variance engine and bridges.
16. Three-statement dashboards, controls, and management reporting.
17. Recurring-revenue template mapped into the financial model.
18. Workflow, comments, approvals, and audit UI.
19. Grounded assistant and keyless commands.
20. API, developer page, OpenAPI snapshot, and contract tests.
21. FastERP, FastHRM, and FastCRM source contracts and adapters.
22. FastSheets, FastInsights, and FastOffice integration contracts.
23. Docker/Coolify/FastDevOps/portfolio updates and demo assets.
24. Production deployment and exact-commit verification.

Each PR should preserve a runnable application and add tests for its new
invariants. Cross-repository changes must be independently committed,
deployed, and verified.

## 19. Release gates

- all financial calculations use Decimal/minor-unit semantics and reconcile;
- all roll-ups and allocations satisfy invariants;
- balance sheet balances and indirect cash flow reconciles to cash movement;
- retained earnings, working capital, fixed assets, and debt roll forward;
- published/locked versions are immutable;
- model cycles and invalid references fail closed;
- every value has input/import/calculation lineage;
- failed imports cannot partially update Actual;
- actual, forecast, scenario, and report as-of/version labels are visible;
- permissions are checked at query/service level with tenant-isolation tests;
- AI output is never the authoritative calculation;
- no real financial or personal data exists in seed/demo artifacts;
- no secret, `.env`, database, credential, or absolute developer path is
  committed;
- anonymous `/` and authenticated `/` behave correctly;
- Sign In and core planning workflow are keyboard-accessible;
- API schema is versioned and matches committed `swagger.json`;
- tests, compile checks, `git diff --check`, and FastDevOps validation pass;
- Docker health works with the actual Coolify health wrapper;
- DNS, TLS, callback, static assets, console, health, and key flow smoke tests
  pass in production;
- the exact deployed Git commit and rollback target are recorded.

## 20. Principal risks and mitigations

| Risk | Mitigation |
|---|---|
| Scope expands into ERP/accounting | Enforce the product boundary and one Release 1 workflow |
| Spreadsheet flexibility undermines control | Structured rules, typed inputs, lineage, versions |
| Calculation results cannot be reproduced | Immutable snapshots, hashes, deterministic order, Decimal |
| SQLite becomes a production bottleneck | Repository contracts, migrations, PostgreSQL gate for real data |
| Generic dimensions make queries unmaintainable | Typed core dimensions plus bounded extension mechanism |
| Integration data does not reconcile | Staged imports, mapping versions, control totals, atomic commit |
| Cross-tenant finance data leaks | `org_id` everywhere, scoped queries, adversarial isolation tests |
| AI hallucinates financial explanations | Deterministic calculations and cited tool outputs only |
| FastERP API lacks monthly detail | Separate contract extension with date filters and stable source IDs |
| Sister APIs lack required bulk/dataset writes | Small token-gated, idempotent contracts; never share databases |
| Cross-product auth remains fragmented | Support FastOffice handoff now; migrate to OIDC authority later |
| Management reporting becomes a document platform | Fixed first reports; integrate FastSlides/FastDocs for richer packs |
| Multi-currency/consolidation consumes Release 1 | Single group currency first unless explicitly prioritised |

## 21. Decision record and launch dependencies

Approved on 2026-07-30:

1. Synthetic public demonstrator first; no real customer finance data.
2. Full integrated monthly P&L, balance sheet, and indirect cash flow in
   Release 1.
3. Serve SMEs, fractional CFOs, and mid-market finance teams.
4. Integrate FastERP, FastHRM, and FastCRM in Release 1.
5. One group currency, multiple departments/business units, and no Release 1
   FX translation or formal multi-entity consolidation.
6. FastOffice is the preferred entry point; Google/local standalone login
   remains available.
7. FastSheets/FastOffice import/export is sufficient; no launch Excel add-in.
8. AI explains and drafts only. The existing suite xAI secret is available
   through protected deployment configuration.
9. Mirror FastERP's synthetic model first, then include a reusable
   recurring-revenue model.
10. Use FastInsights for advanced visualisation and governed exploration while
    keeping core FP&A reports in FastFPA.
11. Use the proposed deep-teal palette and landing wording.
12. Release under MIT like FastERP.
13. Sister-repository changes required for the approved integrations,
    catalogue, FastOffice, and deployment are authorised.
14. Use calendar-year monthly planning initially, with future-compatible
    calendar tables.
15. Launch the synthetic demonstrator on 2026-07-30 with five engineers.

The only unresolved launch dependency is external DNS control: create an A
record for `fpa.fastsme.com` pointing to `191.218.164.166`. Once it resolves,
Coolify can issue the trusted TLS certificate and the final public-domain smoke
test can run. The Google OAuth client must also contain the exact redirect URI
`https://fpa.fastsme.com/auth/google/callback`; FastOffice remains the preferred
entry point if that console update is scheduled after launch.

## 22. Definition of Release 1 done

Release 1 is done when a new user can sign in, open a fully synthetic and
internally reconciled company, import or refresh actuals, inspect mappings,
create a forecast from governed revenue/workforce/expense drivers, clone and
change a scenario, calculate it deterministically, compare it with baseline and
actuals, inspect balanced P&L/balance-sheet/cash-flow results, submit/approve/
lock it, drill every material variance to its lineage, exchange governed
workbooks with FastSheets/FastOffice, open an analytical view in FastInsights,
export a labelled management report, and ask grounded questions without the AI
being able to alter authoritative values. Both the FastERP-aligned and
recurring-revenue templates must pass the financial integrity checks.

The same exact tested commit must be healthy at `fpa.fastsme.com`, with valid
TLS, correct OAuth callback, persistent storage, one deployment trigger,
working developer docs, and a recorded rollback path.
