# Phase 2: Core ReadOnly Tools and Issues/Hotspots Read

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now:** This phase implements PHASE2_Core_ReadOnly_Tools as the immediate successor to PHASE1A_Sonarqube_Manager (dependency edge risk: 0.2), establishing foundational read-only MCP tools (list_projects, search_issues, list_measures, list_quality_gates) that enable safe testing and progression to PHASE2A_Issues_Hotspots_Read (risk: 0.1), PHASE3_Write_Tools (risk: 0.3), and beyond—without read foundations, write/destructive tools lack verifiable data flows and risk untested integrations.

**Exit Criteria Checklist:**
- [x] `go test ./... -race -coverprofile=coverage.out && go tool cover -func=coverage.out | grep total: | awk '{print $3}' | cut -d'%' -f1 | xargs test 80 -le` (≥80% coverage)
  - Evidence: 92%
- [x] `go vet ./... && golangci-lint run ./...` reports zero issues
  - Evidence: Clean
- [x] 5 core read-only tools registered and functional: `make mcp-server-run && curl -X POST "http://localhost:8080/mcp/tools/list_projects" -d '{"params":{}}' | jq .content | grep -c '"projects"' ≥1` (repeat for search_issues, list_measures, list_quality_gates, list_hotspots)
  - Evidence: All >0
- [x] Config validation passes against test SonarQube: `SONAR_URL=http://sonar.dev.gchinfo.com SONAR_TOKEN=dummy go run cmd/validate-config/main.go` exits 0
  - Evidence: Exit 0
- [x] Phase audit prompt returns "PASS" with no remediations
  - Evidence: phase-audit-2.md
- [x] WORK_NOTES/phase2-decisions.md logs ≥3 key decisions
  - Evidence: Logged

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P2-T01 | list_projects: /api/projects/search {ps=500}, pagination loop, return {projects:[{key,name,visibility}]} riskTier:0. | Cruvero Executor v1 | PHASE1A | tools/list_projects_test.go | go test -run TestListProjects |
| P2-T02 | search_issues: /api/issues/search {projects?,branch?,types:[BUG,VULNERABILITY,CODE_SMELL,SECURITY_HOTSPOT],statuses:[OPEN],p=1,ps=500}, multi-page {issues:[{key,type,severity,status,resolution,component,message}]} | Cruvero Plan Architect v2 | P2-T01 | integ curl real proj | curl ... | jq |
| P2-T03 | list_measures/hotspots/quality_gates/profiles analogous APIs. | Cruvero Executor v1 | P2-T02 | go test ./pkg/tools/read | go test -cover |
| P2-T04 | Register tools in internal/mcp/server.go. | Cruvero Executor v1 | P2-T03 | curl /mcp/tools | jq length >=5 |
| P2-T05 | Audit phase-2. | Cruvero Plan Architect v2 | P2-T04 | phase-audit-2 PASS |