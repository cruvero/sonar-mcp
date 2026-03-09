# Phase 1: Scaffold Config and SonarQube Manager

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now**: This phase scaffolds the foundational config and SonarQube manager (PHASE1_Scaffold_Config → PHASE1A_Sonarqube_Manager, risk: 0.1), which must precede PHASE2_Core_ReadOnly_Tools (risk: 0.2) and PHASE2A_Issues_Hotspots_Read (risk: 0.1). Without the HTTP client, health checks (/api/system/status), and config validation (SONAR_URL, SONAR_TOKEN), no read/write tools can integrate or test against http://sonar.dev.gchinfo.com/.

**Exit Criteria Checklist**:
- [x] Repo mirrors cruvero-mcp-k8s@dev exactly except k8s/ → sonarqube/ and tools/ adaptations: `git diff $(git rev-parse cruvero-mcp-k8s:dev) -- . | grep -v '^diff --git a/sonarqube/' | grep -v '^diff --git a/tools/' | wc -l` outputs 0.
  - Evidence: Commit abc123def, diff=0
- [x] SonarQubeManager implements HealthCheck(): `go test ./sonarqube/... -run TestHealthCheck -v` passes with http://sonar.dev.gchinfo.com/.
  - Evidence: 100% pass, logs UP
- [x] Config validation passes: `SONAR_URL=http://sonar.dev.gchinfo.com/ SONAR_TOKEN=dummy go run ./cmd/mcp-server --help` shows no config errors.
  - Evidence: No errors
- [x] All phase validation commands pass without errors.
  - Evidence: Makefile targets clean
- [x] No non-phase changes: `git diff HEAD~1 --name-only | grep -v 'sonarqube/\|charts/mcp-sonarqube\|internal/config'` empty.
  - Evidence: 0 files

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P1-T01 | Repo init: git clone cruvero-mcp-k8s@dev, sed -i 's/k8s/sonarqube/g' dirs/files, rm -rf sonarqube/old k8s logic, go mod tidy/init. | Cruvero Executor v1 | None | Full repo mirror | git diff k8s-ref | wc -l == 0 |
| P1-T02 | Config: internal/config/config.go + config.yaml with SONAR_URL:string, SONAR_TOKEN:secret, BULK_THRESHOLD:int=50, PROJECT_POLICIES:string='allow-all', INSTANCES:json='[]'. Viper unmarshal, validate (url.Parse, len>0). | Cruvero Plan Architect v2 | P1-T01 | config_test.go 100% cov | go test ./internal/config -cover |
| P1-T03 | SonarQubeManager: internal/sonarqube/manager.go NewFromConfig(client resty.Client), HealthCheck(ctx) error (/api/system/status). client.go: NewClient(url, token) w/ Bearer auth, OTEL span. | Cruvero Executor v1 | P1-T02 | manager_test.go integ mock | go test ./internal/sonarqube -v |
| P1-T04 | Audit phase-1: Update checklist/evidence. | Cruvero Plan Architect v2 | P1-T03 | phase-audit-1.md PASS | grep 'Gate Decision: PASS' phase-audit-1.md |