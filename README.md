# TraceNest

A focused distributed-trace explorer for understanding request paths.

> Planning repository — documentation and Git initialization only. No application, package, solution, database, tests, CI workflow, or runnable demo has been implemented yet. Everything below describes the proposed implementation, unless explicitly marked as verified.

## Goals

Help developers follow a request across services, identify the critical path, and compare slow and normal executions. Start with trace ingestion and investigation instead of attempting a complete observability platform.

## Target users

Backend developers learning distributed tracing and small service teams investigating latency or broken parent-child relationships.

## MVP scope

- Ingest OTLP/HTTP JSON trace payloads through a bounded adapter; normalize identifiers and timestamps.
- Search by service, duration, status, time range, and trace ID; show a trace waterfall.
- Display span attributes, parent/child relationships, events, and explicit gaps from missing spans.
- Compare two traces by service/operation and highlight latency differences.
- Configurable retention, payload limits, and attribute redaction before persistence.

## Stretch goals

- OTLP protobuf and gRPC ingestion with conformance fixtures.
- Service dependency view and trace-linked logs.
- Tail sampling and storage indexing experiments with published benchmark datasets.

## Proposed tech stack

C# / .NET 10, ASP.NET Core, Blazor UI, SQLite for local indexed trace storage, OpenTelemetry SDK in sample services, xUnit. Protocol support is deliberately incremental.

Select compatible stable dependency versions during milestone 1, record them in project files, and pin the SDK/toolchain after the compatibility spike. No packages or development workloads are installed by this documentation task.

## Architecture

An ingestion adapter maps supported OTLP input into a canonical trace model. Application services validate batches and persist spans; queries assemble traces without assuming ingestion order. The analysis layer calculates a documented critical-path approximation, and the UI renders normalized span trees. Transport DTOs remain separate from storage and view models.

## Proposed folder structure

Only README.md and .gitignore exist initially, alongside Git metadata. The following tree is a plan; create directories when their implementation begins.

```text
TraceNest/
  README.md
  .gitignore
  TraceNest.slnx
  src/TraceNest.Domain/
  src/TraceNest.Ingestion/
  src/TraceNest.Storage/
  src/TraceNest.Api/
  src/TraceNest.Web/
  samples/TraceNest.DemoServices/
  tests/TraceNest.UnitTests/
  tests/TraceNest.IntegrationTests/
  docs/protocols/
```

## Data models and invariants

Trace(TraceId, FirstSeenUtc, LastSeenUtc); Span(TraceId, SpanId, ParentSpanId, Service, Operation, StartUnixNs, DurationNs, Status, Attributes); SpanEvent(TimestampUnixNs, Name, Attributes). Store nanosecond values as integers; convert to display units at the UI boundary. Missing parents remain visible rather than being silently discarded.

## Proposed API and contracts

POST /v1/traces handles the documented OTLP/HTTP JSON subset; GET /api/traces?service=...&from=...&to=... performs paginated search; GET /api/traces/{traceId} returns spans and completeness warnings; GET /api/traces/compare?left=...&right=... returns normalized comparison data. Do not advertise full collector compatibility before protocol tests pass.

Version persisted/exported formats from their first implementation. Validate at boundaries and return structured errors rather than leaking internal exceptions. The contracts above remain proposals until backed by code and contract tests.

## Implementation milestones

1. Define supported ingestion schema and representative traces, including missing and out-of-order spans.
2. Implement validation, batch writes, duplicate handling, and retention.
3. Build searchable waterfall and gap indicators.
4. Add trace comparison, instrumented demo services, and an ingestion/query benchmark report.

Each milestone should end with a small demonstrable result, a written acceptance check, and tests for its important failure modes. Work through a vertical slice before expanding the stretch scope.

## Testing and CI plan

Cover malformed IDs, duplicate span delivery, out-of-order ingestion, clock skew, missing parents, attribute limits, and retention boundaries. Run contract fixtures against the declared OTLP subset. CI builds/tests on Windows and Linux, and smoke-tests ingestion from the demo services with bounded generated data.

CI is planned, not configured. Once a solution exists, add .github/workflows/ci.yml with read-only default permissions, explicit toolchain setup, dependency restore, build, and the checks above. Keep ordinary CI offline from real services and hardware. Store test reports as artifacts; do not publish packages, create releases, or deploy from the first workflow. Record hardware/manual results separately from automated passes.

## Setup and development

### Current checkout

Open this README in a Markdown viewer. Git is initialized locally; no build or run step exists yet. Check the working tree with `git status --short --branch`. Documentation files start untracked; no initial commit or remote is created.

### Prerequisites and first implementation

After scaffolding, create the solution and projects below, initialize storage through migrations, and run the API plus UI. Start sample services with their trace exporter explicitly pointed at the local collector endpoint and the supported JSON transport. Seed a small trace set before instrumenting a real service.

Install only the dependencies needed for the selected milestone. Add a checked-in sample configuration with harmless defaults; keep local databases, recordings, generated datasets, credentials, and device identifiers outside version control. Pick a license before public distribution; no license is implied by this plan.

### Future commands

These commands are examples for after the referenced projects/files have been created; they will not build this documentation-only repository.

```powershell
dotnet restore TraceNest.slnx
dotnet test TraceNest.slnx
dotnet run --project src/TraceNest.Api
```

## Supporting libraries and boundaries

DataSynth for deterministic trace fixtures and ConfigLens for ingestion limits. TimeSeriesKit may support duration trend summaries later; do not treat spans as ordinary numeric samples.

Consume stable library versions through local package artifacts initially. Avoid absolute ProjectReference paths into sibling repositories and avoid Git submodules for the MVP. Keep a consumer's first vertical slice runnable using a small local interface or fixture while a supporting package is under development.

## Realistic limitations

This is a trace investigation tool, not a replacement for a production collector, metrics backend, or log platform. Sampling and dropped spans limit conclusions; clocks across hosts can disagree. SQLite retention and indexing must be measured before claiming high-volume support.

Scope and performance targets are hypotheses until measured. Document reproducible dataset sizes, hardware, dependency versions, and failure cases alongside any benchmark claim. Do not present planned capabilities as shipped features.

## Documentation provenance

Prepared 2026-10-23. The previous chat supplied the project names and the AutoDiag platform change, but its detailed assistant responses were unavailable content references. The user confirmed the seven proposed application themes in this session; individual designs here are newly proposed plans, not a recovered transcript.

