# Phase 4: Core Services & Interfaces Segment B Segment A

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now:** This phase completes core MCP tools (read/write/destructive with bulk >50 items, risk tiers) and integrates them via gateway/heartbeat/health checks, directly preceding deployment phases. It depends on PHASE3_Write_Tools → PHASE3A_Quality_Gates_Profiles (risk: 0.2) and enables PHASE4_Destructive_Polish → PHASE5_Gateway_Health (risk: 0.2), ensuring functional server before Helm/ArgoCD (PHASE6_Helm_ArgoCD).

**Exit Criteria Checklist:**
- [ ] All tools registered in `internal/mcp/server.go` with 15+ tools (e.g., ListProjects, SearchIssues, TransitionIssue, BulkTransitionIssues, DeleteProject).
- [ ] `go test ./... -race -v -coverprofile=coverage.out && go tool cover -func=coverage.out | grep tools | awk '{print $3}' | sort -nr | head -1` shows >90% coverage on `pkg/sonarqube/tools`.
- [ ] Health check passes: `curl -f http://localhost:8080/healthz` returns `{"status":"healthy","sonar_status":"UP"}`.
- [ ] Gateway registration: `curl -f http://localhost:8080/mcp/register` lists all tools with risk tiers/hints.
- [ ] No issues: `go vet ./... && golangci-lint run ./... && go mod tidy`.
- [ ] Integration tests pass against `http://sonar.dev.gchinfo.com/` with real token.
- [ ] Phase-specific audit passes with zero high-severity findings.

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P4-T01 | Implement/polish all MCP tools in `pkg/sonarqube/tools/`: read (`list_projects.go`: ListProjects(projectsOpts), `search_issues.go`: SearchIssues(issuesOpts), `list_measures.go`: ListMeasures, `list_hotspots.go`, `list_quality_gates.go`, `list_quality_profiles.go`); write (`transition_issue.go`: TransitionIssue(status: "RESOLVED\|WONT_FIX\|FALSE_POSITIVE"), `bulk_transition_issues.go`: BulkTransitionIssues (>50 via config.bulkThreshold)); destructive (`delete_project.go`: DeleteProject(key) with `RiskTier{Level: "DESTRUCTIVE", Hint: "Permanently deletes project data"}`, `bulk_delete_issues.go`). Register in `internal/mcp/server.go` via `mcpgo.RegisterTool`. Add project/branch filters, policy validation. | Cruvero Plan Architect v2 | PHASE3_Write_Tools → PHASE3A_Quality_Gates_Profiles (risk: 0.2), phase-3 audit pass | Changed files: `pkg/sonarqube/tools/*.go` (12+ files), `internal/mcp/server.go`; func signatures with godoc; evidence: `curl http://localhost:8080/mcp/register \| jq '.tools[] \| select(.riskTier)'` | `go test ./pkg/sonarqube/tools/... ./internal/mcp/... -race -v && go vet ./pkg/sonarqube/... && golangci-lint run ./pkg/sonarqube/tools` |
| P4-T02 | Unit tests (`pkg/sonarqube/tools/*_test.go`, `pkg/sonarqube/manager_test.go`) with `mcpgo.TestTool` mocks; integration tests (`tests/integration/tools_test.go`) vs real `http://sonar.dev.gchinfo.com/` (env: SONAR_URL, SONAR_TOKEN); gateway health (`internal/health/health.go`: CheckSonarStatus via `/api/system/status`); register/heartbeat (`cmd/mcp-sonarqube/main.go`, `mcpgo v0.43.2`). | Cruvero Plan Architect v2 | phase-3 audit pass | Test files: 20+ `_test.go`; coverage report; evidence: logs with OTEL/slog traces | `go test ./... -race -v -cover && go tool cover -func=coverage.out \| grep -E '(pkg/sonarqube/tools|internal/mcp)' \| awk '{sum+=$3} END {print sum/NR}' \| awk '{print $1*100"%"}'` (>90%); `curl -H "Authorization: Bearer $SONAR_TOKEN" http://sonar.dev.gchinfo.com/api/system/status \| jq .status` == "UP" |

## Phase-Specific Prompt
```text
You are executing Phase 4: Core Services & Interfaces Segment B Segment A.
Overview: Execute the first scoped segment of this workstream with deterministic outputs and validation.
Scope nodes: PHASE4_Destructive_Polish (destructive tools + bulk/risks in pkg/sonarqube/tools/), PHASE5_Gateway_Health (register/heartbeat in internal/mcp/server.go, healthz with /api/system/status), PHASE6_Helm_ArgoCD (charts/mcp-sonarqube/values.yaml with SONAR_URL/TOKEN).
Required reading: docs/PLAN.md, docs/AUDIT/risk-matrix.md, docs/WORK_NOTES/memory-palace.md.
Output only phase-scoped changes with validation evidence (go test outputs, curl responses) and explicit assumptions.
Record every key decision in UTC under WORK_NOTES.
Implement exact func signatures: e.g., func ListProjects(ctx context.Context, opts *ProjectsOpts) (*[]Project, error); add RiskTier structs.
```

## Phase-Specific Audit Prompt
```text
Audit Phase 4: Core Services & Interfaces Segment B Segment A against acceptance criteria, risk controls, and deterministic behavior.
Verify: all tools (15+) registered/typed correctly; bulkThreshold=50 config in sonarqube/manager.go; health/gateway functional; no k8s remnants.
Return findings ordered by severity (High/Med/Low) with explicit pass/fail gate decision and required remediations.
Check dependency edges: PHASE4_Destructive_Polish complete before PHASE5_Gateway_Health.
```

## Risks / Gates
| Risk | Trigger | Gate | Mitigation | Success Criteria |
|---|---|---|---|---|
| Dependency mismatch | integration check fails | phase-audit-4 | reconcile contracts and retest | all phase validation commands pass |
| Scope drift | non-phase changes appear | phase-audit-4 | update PLAN + re-scope before proceeding | only phase-scoped deliverables remain |
| Quality regression | validation gate fails | release quality gate | patch and rerun full checks | clean quality signal |
| Destructive tool exposure | RiskTier missing on delete funcs | phase-audit-4 | add hints/confirmations in tool metadata | `curl /mcp/register \| jq '[.tools[] \| select(.name \| contains("delete")) .riskTier.level]' \| uniq` == '"DESTRUCTIVE"' |

## Estimated Swarm Duration
- 12h swarm time