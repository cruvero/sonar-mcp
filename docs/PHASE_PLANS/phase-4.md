# Phase 4: Destructive Tools Polish

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now:** This phase completes core MCP tools (read/write/destructive with bulk >50 items, risk tiers) and polishes destructive ops, directly depending on PHASE3_Write_Tools → PHASE3A_Quality_Gates_Profiles (risk: 0.2) and enabling PHASE4_Destructive_Polish → PHASE5_Gateway_Health (risk: 0.2), ensuring functional server before Helm/ArgoCD (PHASE6_Helm_ArgoCD).

**Exit Criteria Checklist:**
- [ ] All tools registered in `internal/mcp/server.go` with 15+ tools (e.g., ListProjects, SearchIssues, TransitionIssue, BulkTransitionIssues, DeleteProject).
  - Evidence: Pending
- [ ] `go test ./... -race -v -coverprofile=coverage.out && go tool cover -func=coverage.out | grep tools | awk '{print $3}' | sort -nr | head -1` shows >90% coverage on `pkg/sonarqube/tools`.
  - Evidence: Pending
- [ ] Health check passes: `curl -f http://localhost:8080/healthz` returns `{"status":"healthy","sonar_status":"UP"}`.
  - Evidence: Pending
- [ ] No issues: `go vet ./... && golangci-lint run ./... && go mod tidy`.
  - Evidence: Pending
- [ ] Integration tests pass against `http://sonar.dev.gchinfo.com/` with real token.
  - Evidence: Pending
- [ ] Phase-specific audit passes with zero high-severity findings.
  - Evidence: Pending

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P4-T01 | Implement/polish all MCP destructive tools: delete_project (/api/projects/delete), bulk_delete_issues (with >50 guard, riskTier=high). | Cruvero Plan Architect v2 | phase-3 | delete_*.go | go test ./pkg/tools/destructive |
| P4-T02 | Add risk hints/confirm for destructive, project policy stub. | Cruvero Plan Architect v2 | P4-T01 | tool wrappers | curl tool | jq .riskTier == "high" |
| P4-T03 | Full tool coverage >90%, lint clean. | Cruvero Plan Architect v2 | P4-T02 | tests | go test -cover |
| P4-T04 | Docs update for tools in PLAN.md. | Cruvero Plan Architect v2 | P4-T03 | docs/PLAN.md | grep -c tool |