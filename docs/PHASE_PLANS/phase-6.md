# Phase 6: Core Services & Interfaces Segment B Segment B Segment B

## Overview
Execute the second scoped segment of this workstream with deterministic outputs and validation.

## Why Now
This phase must precede production deployment and integration phases because PHASE8_CI_Docs establishes automated quality gates (CI/CD pipelines, 100% test coverage) required for all downstream changes, and PHASE9_MultiTenant_Telemetry future-proofs observability (multi-instance config, instanceId-tagged OTEL/slog) for scalable operations. Delaying risks quality regression in later phases. References dependency edges: PHASE5_Core_Write → PHASE8_CI_Docs (risk: 0.1), PHASE8_CI_Docs → PHASE9_MultiTenant_Telemetry (risk: 0.05), PHASE9_MultiTenant_Telemetry → PHASE10_Deployment (risk: 0.2).

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P6-T01 | Mirror CI/CD workflows from github.com/cruvero/cruvero-mcp-k8s@dev/.github/workflows/ (test.yml, build.yml, release.yml); adapt for sonarqube (e.g., test pkg/sonarqube/api_client.go, internal/tools/list_projects_test.go); add jobs for staticcheck, race detector, coverage report. | Cruvero Plan Architect v2 | phase-5 audit pass, PHASE5_Core_Write → PHASE8_CI_Docs | .github/workflows/*.yml, go.mod unchanged | `go test ./... -race && staticcheck ./... && go vet ./...` |
| P6-T02 | Achieve 100% test coverage across cmd/mcp-server, pkg/sonarqube, internal/config, internal/tools; generate coverage report; update phased docs (docs/PLAN.md#phase-6, docs/AUDIT/phase6.md, docs/WORK_NOTES/phase6-decisions.md with UTC timestamps). | Cruvero Plan Architect v2 | P6-T01 | coverage.out, docs/*.md updated | `go test ./... -race -coverprofile=coverage.out && go tool cover -func=coverage.out \| grep total \| awk '{print \$3}' \| grep -q '100.0%' && gocovhtml coverage.out > coverage.html` |
| P6-T03 | Implement PHASE9_MultiTenant_Telemetry: extend internal/config/config.go (add Instances []struct{InstanceId string; SONAR_URL string; SONAR_TOKEN string}), pkg/sonarqube/manager.go (NewMultiClient(ctx, instanceId) selects client), tools funcs (add optional instanceId param routed via mcp-go v0.43.2), OTEL/slog with "sonarqube.instance_id" attr; healthz /api/system/status per instance. | Cruvero Plan Architect v2 | P6-T02, PHASE8_CI_Docs → PHASE9_MultiTenant_Telemetry | internal/config/config.go, pkg/sonarqube/manager.go, pkg/otel/instrumentation.go | `go test ./internal/config/... ./pkg/sonarqube/... -race -cover && go run cmd/mcp-server/main.go --config config.yaml & PID=$!; sleep 5; curl -f http://localhost:8080/healthz?instanceId=default; kill $PID` |

## Exit Criteria Checklist
- [ ] All Detailed Task List validation commands pass without errors or warnings
- [ ] Test coverage exactly 100.0% (`go tool cover -func=coverage.out | grep total`)
- [ ] No lint/staticcheck issues (`staticcheck ./...`)
- [ ] Docs updated with phase changes (git diff docs/ shows only phase-6 commits)
- [ ] Dependency edges validated: integration tests from PHASE5 outputs pass (e.g., list_projects with multi-instance)
- [ ] Risk matrix in docs/AUDIT/risk-matrix.md unchanged or reduced
- [ ] Phase-audit-6 gate passes (deterministic behavior confirmed)

## Phase-Specific Prompt
```text
You are executing Phase 6: Core Services & Interfaces Segment B Segment B Segment B.
Overview: Execute the second scoped segment of this workstream with deterministic outputs and validation.
Scope nodes: PHASE8_CI_Docs, PHASE9_MultiTenant_Telemetry
Required reading: docs/PLAN.md, docs/AUDIT/risk-matrix.md, docs/WORK_NOTES/memory-palace.md.
Specific steps: Mirror .github/workflows/*.yml; ensure 100% coverage on pkg/sonarqube/*_test.go; add multi-instance to internal/config/config.go and route in tools/list_projects etc.; tag OTEL spans with instanceId in pkg/otel.
Output only phase-scoped changes with git diff --stat, full validation command outputs, and explicit assumptions.
Record every key decision in UTC under WORK_NOTES/phase6-decisions.md.
```

## Phase-Specific Audit Prompt
```text
Audit Phase 6: Core Services & Interfaces Segment B Segment B Segment B against acceptance criteria, risk controls, and deterministic behavior.
Verify: CI workflows match cruvero-mcp-k8s (diff .github/workflows/), coverage 100%, multi-instance config parses/validates (go test internal/config), tools route by instanceId, OTEL traces tagged.
Return findings ordered by severity (CRITICAL/HIGH/MED/LOW) with explicit pass/fail gate decision, remediations (e.g., "fix manager.go routing"), and evidence commands (e.g., "go test ./pkg/sonarqube -v").
```

## Risks / Gates
| Risk | Trigger | Gate | Mitigation | Success Criteria |
|---|---|---|---|---|
| Dependency mismatch | go.mod/go.sum differs from cruvero-mcp-k8s@dev or PHASE5 tools fail integration (e.g., api_client.go) | phase-audit-6 | `go mod tidy && go mod vendor` to reconcile contracts; retest tools | all phase validation commands pass; `git diff --no-index cruvero-mcp-k8s/go.mod go.mod` empty |
| Scope drift | commits touch non-scope (e.g., changes to pkg/tools beyond instanceId param) | phase-audit-6 | git revert non-phase; update PLAN.md + re-scope before proceeding | git log --oneline --grep="Phase 6" shows only PHASE8/9; only phase-scoped deliverables remain |
| Quality regression | coverage <100%, staticcheck fails, or OTEL spans missing instanceId | release quality gate | patch tests (e.g., add *_test.go), rerun full checks incl. `go test -race ./...` | clean quality signal: 100% coverage, zero staticcheck/vet issues, curl /healthz?instanceId=default succeeds |

## Estimated Swarm Duration
- 9h swarm time