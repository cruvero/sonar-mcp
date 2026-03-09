# Phase 5: Core Services & Interfaces Segment B Segment B Segment A

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now:** This phase establishes the core MCP server runtime, gateway registration/heartbeat, and SonarQube health checks (/api/system/status) as foundational infrastructure, which must precede tool integrations, bulk operations, and RBAC stubs. It directly enables dependency edge PHASE4_Quality_Gates_Profiles → PHASE5_Core_Services_Interfaces (risk: 0.2) and PHASE5_Core_Services_Interfaces → PHASE7_Bulk_RBAC_Stubs (risk: 0.1), ensuring server stability before read/write tools and multi-instance scaling.

**Exit Criteria Checklist:**
- `go test ./... -race` passes with ≥95% coverage on pkg/mcp/* and internal/sonarqube/* (`go tool cover -func=coverage.out | grep total | awk '{print $3}' | grep -q '95.0\|100.0'`)
- Health endpoint responds OK: `curl -f http://localhost:8080/health | jq '.sonar.status' | grep -q '"UP"'`
- Gateway registration succeeds: logs contain "Gateway registered" and heartbeat ticker active (grep server.log "heartbeat sent")
- `go vet ./... && go build ./cmd/server` succeeds without errors/warnings
- No non-phase changes: `git diff HEAD~1 --name-only | grep -vE 'pkg/mcp/server.go|internal/sonarqube/(client|health).go|go.mod' | wc -l | grep -q 0`
- Phase audit passes with zero high-severity findings

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P5-T01 | Implement gateway registration/heartbeat in `pkg/mcp/server.go` (RegisterGateway(ctx context.Context, gwURL string) using mcp-go v0.43.2 stdio.Serve), SonarQube health checks in `internal/sonarqube/client.go` (SystemStatus(ctx) (*api.SystemStatus, error) calling /api/system/status with global SONAR_TOKEN), integrate into `pkg/mcp/server.go` HealthzHandler(); add unit tests `internal/sonarqube/client_test.go`, `pkg/mcp/server_test.go`; keep `otel/otel.go` (OTEL setup), `logger/slog.go` unchanged. | Cruvero Plan Architect v2 | PHASE4_Quality_Gates_Profiles audit pass (dependency edge risk: 0.2) | `pkg/mcp/server.go`, `internal/sonarqube/client.go`, `*_test.go` files + `go.mod` pinned mcp-go v0.43.2 + logs/evidence | `go test ./pkg/mcp/... ./internal/sonarqube/... -race -v -coverprofile=coverage.out && go tool cover -func=coverage.out \| grep total \| awk '{print \$3}' \| grep -q '95.0'; go vet ./...; go build ./cmd/server/; curl -f -H "Authorization: Bearer \$SONAR_TOKEN" http://localhost:8080/health \| jq '.status' \| grep -q '"UP"'; echo "Gateway heartbeat: tail -f server.log \| grep 'heartbeat sent'"` |

## Phase-Specific Prompt
```text
You are executing Phase 5: Core Services & Interfaces Segment B Segment B Segment A.
Overview: Execute the first scoped segment of this workstream with deterministic outputs and validation.
Scope nodes: PHASE7_Bulk_RBAC_Stubs (implement server stubs for future bulk/RBAC integration).
Required reading: docs/PLAN.md, docs/AUDIT/risk-matrix.md, docs/WORK_NOTES/memory-palace.md.
Output only phase-scoped changes to pkg/mcp/server.go, internal/sonarqube/client.go/health.go, corresponding _test.go with validation evidence (test output, curl responses, logs) and explicit assumptions (e.g., SONAR_URL=http://sonar.dev.gchinfo.com, single token).
Record every key decision in UTC under WORK_NOTES/phase5-decisions.md.
```

## Phase-Specific Audit Prompt
```text
Audit Phase 5: Core Services & Interfaces Segment B Segment B Segment A against acceptance criteria (gateway/heartbeat/OTEL unchanged, health checks via /api/system/status), risk controls (dependency edge PHASE4→PHASE5 risk 0.2), and deterministic behavior (tests pass, health=UP).
Return findings ordered by severity (critical/high/medium/low) with explicit pass/fail gate decision, required remediations, and evidence checks (coverage≥95%, curl health UP, no scope drift).
```

## Risks / Gates
| Risk | Trigger | Gate | Mitigation | Success Criteria |
|---|---|---|---|---|
| Dependency mismatch | integration check fails (e.g., mcp-go v0.43.2 incompatibility post-PHASE4) | phase-audit-5 | reconcile contracts (update go.mod), retest health/gateway | all phase validation commands pass; `go mod tidy && go test ./...` clean |
| Scope drift | non-phase changes appear (e.g., tool impl in server.go) | phase-audit-5 | update PLAN.md + re-scope before proceeding; `git reset --hard` | only phase-scoped deliverables remain (`git diff --name-only \| grep -vE 'pkg/mcp/server.go\|internal/sonarqube/client.go' \| wc -l == 0`) |
| Quality regression | validation gate fails (e.g., race in heartbeat ticker) | release quality gate | patch new code, rerun full checks (`go test -race ./...`) | clean quality signal: `go test ./... -race -cover && go vet ./...` passes |

## Estimated Swarm Duration
- 6h swarm time