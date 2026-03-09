# Phase 3: Core Services & Interfaces Segment A Segment B Segment B

## Overview
Execute the second scoped segment of this workstream with deterministic outputs and validation.

### Why Now
This phase implements core write tools (PHASE3_Write_Tools) and quality gates/profiles tools (PHASE3A_Quality_Gates_Profiles), which are direct prerequisites for downstream phases like PHASE4_Gateway_Integration (dependency edge PHASE3A_Quality_Gates_Profiles → PHASE4_Gateway_Integration, risk: 0.25) and PHASE5_Bulk_Destructive_Ops (dependency edge PHASE3_Write_Tools → PHASE5_Bulk_Destructive_Ops, risk: 0.4). Write capabilities enable issue transitions and quality gate associations before bulk handling (>50 items) and gateway heartbeat/registration in later phases; delaying risks blocking SonarQube API integration testing against http://sonar.dev.gchinfo.com/.

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P3-T01 | Implement issue/hotspot transition tool: `tools/transition_issue.go` with `TransitionIssue(ctx context.Context, params mcpgo.Params) (*mcpgo.CallResponse, error)` supporting statuses RESOLVE/WONT_FIX/FALSE_POSITIVE via SonarQube /api/issues/set_type?issue=KEY&set_type=... using `sonarqube/client.go::UpdateIssue(key, transition)`. Add riskTier:"write", projectKey scoping (stub RBAC allow-all). | Cruvero Plan Architect v2 | PHASE2A_Issues_Hotspots_Read (edge risk: 0.3), phase-2 audit pass | `tools/transition_issue.go`, `sonarqube/client.go::UpdateIssue`, tool registration in `cmd/server/main.go::RegisterTools()` | `go test ./tools -run TestTransitionIssue -race -v && go vet ./tools ./sonarqube` |
| P3-T02 | Implement bulk issue transitions: `tools/bulk_transition_issues.go` with guard `if len(issueKeys) > bulkThreshold { return errBulkTooLarge }` (threshold from config), loop client calls, riskTier:"write:bulk". | Cruvero Plan Architect v2 | P3-T01 | `tools/bulk_transition_issues.go`, config load in `internal/config/config.go::BulkThreshold` | `go test ./tools -run TestBulkTransitionIssues -race && go test ./internal/config -race` |
| P3-T03 | Implement quality gates tools: `tools/quality_gates.go` with `ListQualityGatesTool`, `AssociateProjectQualityGateTool(projectKey string, gateId string)` via /api/qualitygates/select, `ListQualityProfilesTool` via /api/qualityprofiles/search. Handle pagination with `client.Paginate(api/search?ps=500&p=...)`. | Cruvero Plan Architect v2 | PHASE3_Write_Tools (edge risk: 0.2) | `tools/quality_gates.go`, `sonarqube/client.go::ListQualityGates`, `::AssociateGate`, `::Paginate` | `go test ./tools -run TestQualityGates -race -v && curl -H "Authorization: Bearer $SONAR_TOKEN" "$SONAR_URL/api/qualitygates/search" \| jq` |
| P3-T04 | Add project policy validation stub: `internal/policy/policy.go::ValidateProjectAccess(projectKey string) bool` (allow-all, future RBAC), integrate in all write tools params. Update config schema for SONAR_URL, SONAR_TOKEN validation. | Cruvero Plan Architect v2 | P3-T01, P3-T03 | `internal/policy/policy.go`, config updates in `pkg/config/schema.json` | `go test ./internal/policy -v && go run cmd/server/main.go --help \| grep SONAR_TOKEN` |
| P3-T05 | Ensure Community Edition limits: no calls to paid APIs (e.g., skip /api/ce/*), add health check /api/system/status in `internal/health/health.go`. | Cruvero Plan Architect v2 | P3-T04 | `sonarqube/client.go::HealthCheck`, tool guards | `curl -H "Authorization: Bearer $SONAR_TOKEN" "$SONAR_URL/api/system/status" \| jq .status \| grep UP && go test ./internal/health -race` |

## Exit Criteria Checklist
- [ ] All P3-Txx tasks deliver validated code: `go test ./... -race -coverprofile=coverage.out && go tool cover -func=coverage.out | grep -E 'total:\s+\[no statements\]|total:\s+\(statements\)\s+90.0%'`
- [ ] Dependency edges confirmed: Integration tests pass for PHASE2A_Issues_Hotspots_Read → PHASE3_Write_Tools (e.g., `go test ./tools/integration_test.go`) and PHASE3_Write_Tools → PHASE3A_Quality_Gates_Profiles.
- [ ] No regressions: `go vet ./... && golangci-lint run --timeout=5m && go mod tidy`
- [ ] Config validation: `go test ./pkg/config -run TestLoadConfig` with SONAR_URL/SONAR_TOKEN env vars succeeds.
- [ ] Phase audit passes: Run Phase-Specific Audit Prompt yields "PASS" with zero high/medium severity findings.
- [ ] Measurable coverage: Tools handle >50 bulk (guard triggers), pagination (500+ results), tested against http://sonar.dev.gchinfo.com/.

## Phase-Specific Prompt
```text
You are executing Phase 3: Core Services & Interfaces Segment A Segment B Segment B.
Overview: Execute the second scoped segment of this workstream with deterministic outputs and validation.
Scope nodes: PHASE3A_Quality_Gates_Profiles, PHASE3_Write_Tools. Implement exact files/functions: tools/transition_issue.go::TransitionIssue, tools/bulk_transition_issues.go, tools/quality_gates.go::ListQualityGates/AssociateProjectQualityGate, internal/policy/policy.go::ValidateProjectAccess.
Required reading: docs/PLAN.md, docs/AUDIT/risk-matrix.md, docs/WORK_NOTES/memory-palace.md.
Output only phase-scoped changes (e.g., git diff --name-only | grep 'tools/\|sonarqube/\|internal/policy') with validation evidence (e.g., go test output, curl responses) and explicit assumptions (e.g., Community Edition API limits).
Record every key decision in UTC under WORK_NOTES/phase3-decisions.md.
```

## Phase-Specific Audit Prompt
```text
Audit Phase 3: Core Services & Interfaces Segment A Segment B Segment B against acceptance criteria (e.g., write tools: transition_issue, bulk_transition_issues; quality gates tools), risk controls (riskTier:write, bulkThreshold=50), dependency edges (PHASE2A→PHASE3 risk 0.3, PHASE3→PHASE3A risk 0.2), and deterministic behavior (pagination, project scoping).
Verify: go test ./... -race passes 100%, no non-phase files changed, SonarQube API calls match Community Edition (/api/issues/set_type, /api/qualitygates/*).
Return findings ordered by severity (HIGH/MED/LOW) with explicit pass/fail gate decision (PASS/FAIL/REMEDIATE) and required remediations (e.g., "Add pagination to ListQualityGates").
```

## Risks / Gates
| Risk | Trigger | Gate | Mitigation | Success Criteria |
|---|---|---|---|---|
| Dependency mismatch | integration check fails (e.g., PHASE2A→PHASE3 edge) | phase-audit-3 | reconcile contracts (e.g., issueKey schema), retest tools/integration_test.go | all phase validation commands pass; `go test ./tools/integration -race` 100% |
| Scope drift | non-phase changes appear (e.g., gateway code) | phase-audit-3 | git revert non-phase commits, update PLAN + re-scope before proceeding | `git diff --name-only origin/dev | grep -vE '^tools/|^sonarqube/|^internal/(policy\|config)'` empty; only phase-scoped deliverables remain |
| Quality regression | validation gate fails (e.g., race in bulk_transition) | release quality gate | patch specific func (e.g., tools/bulk_transition_issues.go), rerun full checks | clean quality signal: `go test ./... -race -covermode=atomic`, golangci-lint clean, coverage >90% |

## Estimated Swarm Duration
- 9h swarm time