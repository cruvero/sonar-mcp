# Phase 2: Core Services & Interfaces Segment A Segment B Segment A

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now:** This phase implements PHASE2_Core_ReadOnly_Tools as the immediate successor to PHASE1A_Sonarqube_Manager (dependency edge risk: 0.2), establishing foundational read-only MCP tools (list_projects, search_issues, list_measures, list_quality_gates) that enable safe testing and progression to PHASE2A_Issues_Hotspots_Read (risk: 0.1), PHASE3_Write_Tools (risk: 0.3), and beyond—without read foundations, write/destructive tools lack verifiable data flows and risk untested integrations.

**Exit Criteria Checklist:**
- [ ] `go test ./... -race -coverprofile=coverage.out && go tool cover -func=coverage.out | grep total: | awk '{print $3}' | cut -d'%' -f1 | xargs test 80 -le` (≥80% coverage)
- [ ] `go vet ./... && golangci-lint run ./...` reports zero issues
- [ ] 5 core read-only tools registered and functional: `make mcp-server-run && curl -X POST "http://localhost:8080/mcp/tools/list_projects" -d '{"params":{}}' | jq .content | grep -c '"projects"' ≥1` (repeat for search_issues, list_measures, list_quality_gates, list_hotspots)
- [ ] Config validation passes against test SonarQube: `SONAR_URL=http://sonar.dev.gchinfo.com SONAR_TOKEN=dummy go run cmd/validate-config/main.go` exits 0
- [ ] Phase audit prompt returns "PASS" with no remediations
- [ ] WORK_NOTES/phase2-decisions.md logs ≥3 key decisions with UTC timestamps

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P2-T01 | Enhance pkg/config/config.go: Add `SONAR_URL string \`env:"SONAR_URL" validate:"required,url"\``, `SONAR_TOKEN mcpgo.Secret`, `BulkThreshold int \`env:"BULK_THRESHOLD" default:"50"\``, `MultiInstanceJSON string \`json:"multiInstance"\``. Implement `(*Config).Validate(ctx)` calling manager.AuthValidate() via /api/authentication/validate; add unit tests in config_test.go covering invalid URL/token. | Cruvero Plan Architect v2 | PHASE1A_Sonarqube_Manager | pkg/config/config.go + config_test.go (≥90% coverage) + README.md config example | `go test ./pkg/config/... -race -coverprofile=coverage.out -v && go tool cover -func=coverage.out \| grep config.go \| awk '{print $3}' \| cut -d'%' -f1 \| xargs test 90 -le` |
| P2-T02 | Implement tools/projects/list_projects.go: mcp.Tool with Name()="list_projects", riskTier=mcp.ReadTier; Call() parses params (query string, ps=500 max), calls s.mgr.(*sonarqube.Manager).SearchProjects(ctx, sonarqube.SearchProjectsParams{Query:query, Ps:ps}); return ListProjectsResult{Projects:[]Project{...}}. Add tests/projects/list_projects_test.go with mocks. Register in server/register_tools.go. | Cruvero Plan Architect v2 | P2-T01 | tools/projects/list_projects.go + _test.go + server/register_tools.go update | `go test ./tools/projects/list_projects/... -race -v && go test ./server -race` |
| P2-T03 | Implement tools/issues/search_issues.go: mcp.Tool "search_issues", riskTier=mcp.ReadTier; params (project string, branch string, types[]string, resolved=bool); calls mgr.SearchIssues(ctx, params) mapping /api/issues/search response; support pagination (p=1, ps=500). Tests in _test.go. Register tool. | Cruvero Plan Architect v2 | P2-T01 | tools/issues/search_issues.go + _test.go + server/register_tools.go update | `go test ./tools/issues/search_issues/... -race -v -timeout=5m && curl -X POST "http://localhost:8080/mcp/tools/search_issues" -H "Content-Type:application/json" -d '{"params":{"project":"test"}}' \| jq .content \| grep -c "issues"` |
| P2-T04 | Implement tools/measures/list_measures.go: mcp.Tool "list_measures", riskTier=mcp.ReadTier; params (project string, branch string, metrics[]string); calls /api/measures/component with bulk>50 handling via loop/ps=500. Tests + register. | Cruvero Plan Architect v2 | P2-T01 | tools/measures/list_measures.go + _test.go + server/register_tools.go update | `go test ./tools/measures/... -race -v && go test ./... -race \| grep -c "ok"` |
| P2-T05 | Implement tools/quality_gates/list_quality_gates.go: mcp.Tool "list_quality_gates", riskTier=mcp.ReadTier; fetches /api/qualitygates/list + /api/qualitygates/project_status; tests + register. Update docs/TOOLS.md with params/hints. | Cruvero Plan Architect v2 | P2-T01 | tools/quality_gates/list_quality_gates.go + _test.go + docs/TOOLS.md | `go test ./tools/quality_gates/... -race -v && make docs-validate` |

## Phase-Specific Prompt
```text
You are executing Phase 2: Core Services & Interfaces Segment A Segment B Segment A.
Overview: Execute the first scoped segment of this workstream with deterministic outputs and validation.
Scope nodes: PHASE2_Core_ReadOnly_Tools
Required reading: docs/PLAN.md, docs/AUDIT/risk-matrix.md, docs/WORK_NOTES/memory-palace.md.
Output only phase-scoped changes with validation evidence and explicit assumptions.
Record every key decision in UTC under WORK_NOTES.
Implement exactly the 5 Detailed Task List items above, using SonarQube Community Edition APIs (/api/projects/search, /api/issues/search, /api/measures/component, /api/qualitygates/list), sonarqube.Manager methods, and mcp-go v0.43.2 Tool interfaces. Test against http://sonar.dev.gchinfo.com/ where possible.
```

## Phase-Specific Audit Prompt
```text
Audit Phase 2: Core Services & Interfaces Segment A Segment B Segment A against acceptance criteria, risk controls, and deterministic behavior.
Verify: 5 read-only tools implemented/registered with risk tiers/hints; config enhancements pass validation; all deps from PHASE1A_Sonaruqbe_Manager resolved; exit criteria checklist fully checked.
Return findings ordered by severity with explicit pass/fail gate decision and required remediations.
```

## Risks / Gates
| Risk | Trigger | Gate | Mitigation | Success Criteria |
|---|---|---|---|---|
| Dependency mismatch | PHASE1A_Sonarqube_Manager absent (no internal/sonarqube/manager.go or NewSonarQubeManager fails) | phase-audit-2 | Pull latest repo; implement missing mgr.SearchProjects etc. via http.Client + OTEL; retest | all phase validation commands pass; `go test ./internal/sonarqube -race` ok |
| Scope drift | non-phase changes appear (e.g., write tools or k8s remnants) | phase-audit-2 | git revert non-phase commits; update PLAN + re-scope before proceeding | only phase-scoped deliverables remain; `git diff HEAD~5 --name-only \| grep -E '^(pkg/config|tools/(projects|issues|measures|quality_gates)|server/register_tools)' \| wc -l` = expected files |
| Quality regression | validation gate fails (e.g., race in bulk>50, API auth fail) | release quality gate | Patch tests for real SonarQube http://sonar.dev.gchinfo.com/; rerun full checks + coverage | clean quality signal; `go test ./... -race -parallel=4` <1s avg, zero failures |

## Estimated Swarm Duration
- 6h swarm time