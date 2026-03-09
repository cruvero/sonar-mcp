# Phase 1: Core Services & Interfaces Segment A Segment A

## Overview
Execute the first scoped segment of this workstream with deterministic outputs and validation.

**Why Now**: This phase scaffolds the foundational config and SonarQube manager (PHASE1_Scaffold_Config → PHASE1A_Sonarqube_Manager, risk: 0.1), which must precede PHASE2_Core_ReadOnly_Tools (risk: 0.2) and PHASE2A_Issues_Hotspots_Read (risk: 0.1). Without the HTTP client, health checks (/api/system/status), and config validation (SONAR_URL, SONAR_TOKEN), no read/write tools can integrate or test against http://sonar.dev.gchinfo.com/.

**Exit Criteria Checklist**:
- [ ] Repo mirrors cruvero-mcp-k8s@dev exactly except k8s/ → sonarqube/ and tools/ adaptations: `git diff $(git rev-parse cruvero-mcp-k8s:dev) -- . | grep -v '^diff --git a/sonarqube/' | grep -v '^diff --git a/tools/' | wc -l` outputs 0.
- [ ] SonarQubeManager implements HealthCheck(): `go test ./sonarqube/... -run TestHealthCheck -v` passes with http://sonar.dev.gchinfo.com/.
- [ ] Config validation passes: `SONAR_URL=http://sonar.dev.gchinfo.com/ SONAR_TOKEN=dummy go run ./cmd/mcp-server --help` shows no config errors.
- [ ] All phase validation commands pass without errors.
- [ ] No non-phase changes: `git diff HEAD~1 --name-only | grep -v 'sonarqube/\|charts/mcp-sonarqube\|internal/config'` empty.

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P1-T01 | Repo initialized as exact mirror of github.com/cruvero/cruvero-mcp-k8s@dev: <br>- `git remote add mirror https://github.com/cruvero/cruvero-mcp-k8s.git && git fetch mirror dev:mcp-k8s-dev`. <br>- Retain structure: cmd/mcp-server/main.go, internal/config/config.go (add SonarQube{URL, Token string, BulkThreshold int=50}), pkg/mcp/*.go, gateway/*.go. <br>- Delete k8s/, create sonarqube/manager.go: `type SonarQubeManager struct { client *http.Client config config.SonarQube }; func NewSonarQubeManager(c *http.Client, cfg config.SonarQube) *SonarQubeManager; func (m *SonarQubeManager) HealthCheck(ctx context.Context) error { req, _ := http.NewRequestWithContext(ctx, "GET", m.config.URL+"/api/system/status", nil); ... }`. <br>- Adapt tools/list_projects.go etc. to stubs calling manager.HealthCheck(). <br>- Update go.mod: `replace github.com/cruvero/mcp-go => github.com/cruvero/mcp-go v0.43.2`. | Cruvero Plan Architect v2 | none | Repo at github.com/cruvero/cruvero-mcp-sonarqube with mirror commit + changes; docs/CHANGELOG.md updated. | `go mod tidy && go vet ./... && go test ./... -race -v && SONAR_URL=http://sonar.dev.gchinfo.com/ SONAR_TOKEN=$(cat .env.token) go run ./cmd/mcp-server --health` returns "healthy" |
| P1-T02 | Helm chart charts/mcp-sonarqube deploys successfully with ArgoCD values: <br>- Copy charts/mcp-k8s/ → charts/mcp-sonarqube/, update Chart.yaml name/version. <br>- values-argocd.yaml: add sonar.url, sonar.token from secrets, image tags matching cruvero-mcp-k8s. <br>- templates/deployment.yaml: envFrom secretRef for SONAR_TOKEN, livenessProbe exec: /mcp-server --health. <br>- Retain OTEL/slog configs. | Cruvero Plan Architect v2 | P1-T01 | charts/mcp-sonarqube/ with deployable manifests. | `helm lint charts/mcp-sonarqube && helm template charts/mcp-sonarqube --values charts/mcp-sonarqube/values-argocd.yaml --debug \| kubectl apply --dry-run=client -f - && go test ./charts/mcp-sonarqube/... -race` passes without errors |

## Phase-Specific Prompt
```text
You are executing Phase 1: Core Services & Interfaces Segment A Segment A.
Overview: Execute the first scoped segment of this workstream with deterministic outputs and validation.
Scope nodes: PHASE1A_Sonarqube_Manager, PHASE1_Scaffold_Config, PHASE2A_Issues_Hotspots_Read
Required reading: docs/PLAN.md, docs/AUDIT/risk-matrix.md, docs/WORK_NOTES/memory-palace.md.
Output only phase-scoped changes with validation evidence and explicit assumptions.
Record every key decision in UTC under WORK_NOTES.
```

## Phase-Specific Audit Prompt
```text
Audit Phase 1: Core Services & Interfaces Segment A Segment A against acceptance criteria, risk controls, and deterministic behavior.
Return findings ordered by severity with explicit pass/fail gate decision and required remediations.
```

## Risks / Gates
| Risk | Trigger | Gate | Mitigation | Success Criteria |
|---|---|---|---|---|
| Dependency mismatch | integration check fails | phase-audit-1 | reconcile contracts and retest | all phase validation commands pass |
| Scope drift | non-phase changes appear | phase-audit-1 | update PLAN + re-scope before proceeding | only phase-scoped deliverables remain |
| Quality regression | validation gate fails | release quality gate | patch and rerun full checks | clean quality signal |

## Estimated Swarm Duration
- 12h swarm time