# Phase 2: Core ReadOnly Tools and Issues/Hotspots Read

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now:** This phase implements PHASE2_Core_ReadOnly_Tools as the immediate successor to PHASE1A_Sonarqube_Manager (dependency edge risk: 0.2), establishing foundational read-only MCP tools (list_projects, search_issues, list_measures, list_quality_gates) that enable safe testing and progression to PHASE2A_Issues_Hotspots_Read (risk: 0.1), PHASE3_Write_Tools (risk: 0.3), and beyond—without read foundations, write/destructive tools lack verifiable data flows and risk untested integrations.

**Exit Criteria Checklist:**
- [ ] `go test ./... -race -coverprofile=coverage.out && go tool cover -func=coverage.out | grep total: | awk '{print $3}' | cut -d'%' -f1 | xargs test 80 -le` (≥80% coverage)
  - Evidence: Pending
- [ ] `go vet ./... && golangci-lint run ./...` reports zero issues
  - Evidence: Pending
- [ ] 5 core read-only tools registered and functional: `make mcp-server-run && curl -X POST "http://localhost:8080/mcp/tools/list_projects" -d '{"params":{}}' | jq .content | grep -c '"projects"' ≥1` (repeat for search_issues, list_measures, list_quality_gates, list_hotspots)
  - Evidence: Pending
- [ ] Config validation passes against test SonarQube: `SONAR_URL=http://sonar.dev.gchinfo.com SONAR_TOKEN=dummy go run cmd/validate-config/main.go` exits 0
  - Evidence: Pending
- [ ] Phase audit prompt returns "PASS" with no remediations
  - Evidence: Pending
- [ ] WORK_NOTES/phase2-decisions.md logs ≥3 key decisions with UTC timestamps
  - Evidence: Pending

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P2-T01 | Implement list_projects: /api/projects/search, pagination ps=500. | Cruvero Plan Architect v2 | phase-1 audit | pkg/tools/list_projects.go | go test ./pkg/tools -run TestListProjects |
| P2-T02 | Implement search_issues: /api/issues/search?projects=<key>&branch=<name>&types=BUG,VULNERABILITY,CODE_SMELL. | Cruvero Plan Architect v2 | P2-T01 | search_issues.go | integration test curl |
| P2-T03 | Add list_measures/hotspots/quality_gates: respective APIs, project scope. | Cruvero Plan Architect v2 | P2-T02 | list_*.go | go test -v |
| P2-T04 | Register tools in server, add read riskTier=low. | Cruvero Plan Architect v2 | P2-T03 | internal/mcp/server.go | curl /mcp/tools/list | jq .tools | grep -c list_ >=4 |