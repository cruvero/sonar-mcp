# Phase 5: Gateway Health

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now:** This phase establishes the core MCP server runtime, gateway registration/heartbeat, and SonarQube health checks (/api/system/status) as foundational infrastructure, depending on PHASE4_Destructive_Polish → PHASE5_Gateway_Health (risk: 0.2), enabling PHASE5_Gateway_Health → PHASE6_Helm_ArgoCD (risk: 0.1), ensuring server stability before bulk/RBAC stubs (PHASE7).

**Exit Criteria Checklist:**
- [ ] `go test ./... -race` passes with ≥95% coverage on pkg/mcp/* and internal/sonarqube/* (`go tool cover -func=coverage.out | grep total | awk '{print $3}' | grep -q '95.0\\|100.0'`)
  - Evidence: Pending
- [ ] Health endpoint responds OK: `curl -f http://localhost:8080/health | jq '.sonar.status' | grep -q '"UP"'`
  - Evidence: Pending
- [ ] Gateway registration succeeds: logs contain "Gateway registered" and heartbeat ticker active (grep server.log "heartbeat sent")
  - Evidence: Pending
- [ ] `go vet ./... && go build ./cmd/server` succeeds without errors/warnings
  - Evidence: Pending
- [ ] No non-phase changes: `git diff HEAD~1 --name-only | grep -vE 'pkg/mcp/server.go|internal/sonarqube/(client|health).go|go.mod' | wc -l | grep -q 0`
  - Evidence: Pending
- [ ] Phase audit passes with zero high-severity findings
  - Evidence: Pending

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P5-T01 | Implement gateway registration/heartbeat in pkg/gateway (mirror k8s: register /mcp/gateway/register, ticker 30s). TLS/SPIFFE. | Cruvero Executor v1 | PHASE4 | gateway_test.go | go test |
| P5-T02 | /healthz: aggregate mcp healthy + manager.HealthCheck() cached. | Cruvero Plan Architect v2 | P5-T01 | curl /healthz | jq |
| P5-T03 | cmd/mcp-server/main.go: flags, server start, graceful shutdown. | Cruvero Executor v1 | P5-T02 | go run . & curl /healthz |
| P5-T04 | Audit phase-5. | Cruvero Plan Architect v2 | P5-T03 | phase-audit-5 |