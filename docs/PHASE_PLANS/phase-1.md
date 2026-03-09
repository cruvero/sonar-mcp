# Phase 1: Scaffold Config and SonarQube Manager

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now**: This phase scaffolds the foundational config and SonarQube manager (PHASE1_Scaffold_Config → PHASE1A_Sonarqube_Manager, risk: 0.1), which must precede PHASE2_Core_ReadOnly_Tools (risk: 0.2) and PHASE2A_Issues_Hotspots_Read (risk: 0.1). Without the HTTP client, health checks (/api/system/status), and config validation (SONAR_URL, SONAR_TOKEN), no read/write tools can integrate or test against http://sonar.dev.gchinfo.com/.

**Exit Criteria Checklist**:
- [ ] Repo mirrors cruvero-mcp-k8s@dev exactly except k8s/ → sonarqube/ and tools/ adaptations: `git diff $(git rev-parse cruvero-mcp-k8s:dev) -- . | grep -v '^diff --git a/sonarqube/' | grep -v '^diff --git a/tools/' | wc -l` outputs 0.
  - Evidence: Pending
- [ ] SonarQubeManager implements HealthCheck(): `go test ./sonarqube/... -run TestHealthCheck -v` passes with http://sonar.dev.gchinfo.com/.
  - Evidence: Pending
- [ ] Config validation passes: `SONAR_URL=http://sonar.dev.gchinfo.com/ SONAR_TOKEN=dummy go run ./cmd/mcp-server --help` shows no config errors.
  - Evidence: Pending
- [ ] All phase validation commands pass without errors.
  - Evidence: Pending
- [ ] No non-phase changes: `git diff HEAD~1 --name-only | grep -v 'sonarqube/\\|charts/mcp-sonarqube\\|internal/config'` empty.
  - Evidence: Pending

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P1-T01 | Repo initialized as exact mirror of github.com/cruvero/cruvero-mcp-k8s@dev: git remote add mirror https://github.com/cruvero/cruvero-mcp-k8s; git checkout -b dev; find . -name '*k8s*' | xargs sed -i 's/k8s/sonarqube/g'; go mod tidy. | Cruvero Plan Architect v2 | None | git diff mirror/dev -- . | grep -v sonarqube | wc -l == 0 |
| P1-T02 | Implement internal/sonarqube/manager.go & client.go: HTTP client with SONAR_URL/TOKEN auth, HealthCheck(/api/system/status). | Cruvero Plan Architect v2 | P1-T01 | manager.go, client.go, health_test.go | go test ./internal/sonarqube -v |
| P1-T03 | Add config/internal/config: SONAR_URL, SONAR_TOKEN, bulkThreshold=50, multiInstance JSON stub. Validate on startup. | Cruvero Plan Architect v2 | P1-T02 | config.go, validate_test.go | go run cmd/mcp-server --help | grep -q 'no errors' |
| P1-T04 | Update go.mod/Makefile for mcp-go v0.43.2, OTEL/slog unchanged. | Cruvero Plan Architect v2 | P1-T01 | go.mod | go mod tidy && go build ./... |