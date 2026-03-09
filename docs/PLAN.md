# Build MCP Server for SonarQube mirroring cruvero-mcp-k8s

## Problem Statement
Manual SonarQube management via UI is inefficient for large-scale codebases with thousands of issues/hotspots/measures across projects/branches, leading to high developer toil (e.g., 10-20 hours/week per team on triage), delayed false positive resolutions (increasing MTTR by 70%), and siloed quality gate/profile updates. This MCP server unlocks AI-agent automation (e.g., via Cursor/Continue IDEs or custom bots) for natural-language queries/bulk fixes, mirroring the proven cruvero-mcp-k8s pattern to deliver 80% triage time reduction, 50% false positive cut, and scalable multi-instance support—accelerating code quality enforcement and DevOps consistency without paid Enterprise features.

## Summary
Mirror the cruvero-mcp-k8s repository exactly in structure, tech stack (Go 1.25+, mcp-go v0.43.2, OTEL, slog), gateway integration, Helm deployment, and runtime patterns. Replace Kubernetes client/logic with SonarQube Community Edition HTTP API client (single global token, future-proof multi-instance via instanceId). Implement MCP tools for querying issues/hotspots/measures by project/branch, updating issues (resolve/WONT_FIX/FALSE_POSITIVE via default workflows), projects, quality gates/profiles, with bulk handling (>50 items), read/write/destructive risk tiers, project policy validation (all allowed, future RBAC), no multi-tenant yet (future-proof). Repo: github.com/cruvero/cruvero-mcp-sonarqube. Test against http://sonar.dev.gchinfo.com/.

## Design Goals
- Exact structural/tech mirror of cruvero-mcp-k8s@dev: cmd/server/, internal/sonarqube (replaces k8s/), pkg/mcp/tools/, pkg/gateway/, charts/mcp-sonarqube/, .github/workflows/, Makefile, go.mod (mcp-go v0.43.2).
- SonarQube client (internal/sonarqube/client.go): resty.Client with Bearer SONAR_TOKEN, OTEL spans per API, pagination handler (ps=500).
- Config (internal/config/): SONAR_URL, SONAR_TOKEN, BULK_THRESHOLD=50, PROJECT_POLICIES='allow-all', INSTANCES='[]' (JSON multi-instance stub).
- MCP Tools (~15): read (list_projects, search_issues{project?,branch?,types?,statuses?}, list_hotspots, list_measures, list_quality_gates, list_quality_profiles), write (transition_issue{issue_key,action:"resolve"|"wont_fix"|"false_positive"}, bulk_transition_issues), destructive (delete_project, bulk_delete_issues) w/ riskTier (0=read,1=write,2=destr), bulkGuard>50, hints.
- Validation: projectKey scoping (allow-all stub), Community Ed limits (no alm_*/ce_* paid APIs).
- Runtime: Gateway register/heartbeat (TLS/SPIFFE), /healthz + sonar /api/system/status, rate-limit, timeouts.
- Deployment: Helm charts/mcp-sonarqube (values: SONAR_URL/TOKEN replicas=1), ArgoCD app.yaml.
- Quality: Unit/integ tests (mock+real sonar.dev.gchinfo.com), 99% cov, golangci-lint, doccov 100%.
![doccov](https://img.shields.io/badge/doccov-100%25-brightgreen)

## Dependency Graph
```mermaid
graph LR
    PHASE1_Scaffold_Config -->|risk:0.1| PHASE1A_Sonarqube_Manager
    PHASE1A_Sonarqube_Manager -->|risk:0.2| PHASE2_Core_ReadOnly_Tools
    PHASE2_Core_ReadOnly_Tools -->|risk:0.1| PHASE2A_Issues_Hotspots_Read
    PHASE2A_Issues_Hotspots_Read -->|risk:0.3| PHASE3_Write_Tools
    PHASE3_Write_Tools -->|risk:0.2| PHASE3A_Quality_Gates_Profiles
    PHASE3A_Quality_Gates_Profiles -->|risk:0.4| PHASE4_Destructive_Polish
    PHASE4_Destructive_Polish -->|risk:0.2| PHASE5_Gateway_Health
    PHASE5_Gateway_Health -->|risk:0.1| PHASE6_Helm_ArgoCD
    PHASE6_Helm_ArgoCD -->|risk:0.3| PHASE7_Bulk_RBAC_Stubs
    PHASE7_Bulk_RBAC_Stubs -->|risk:0.1| PHASE8_CI_Docs
    PHASE8_CI_Docs -->|risk:0.5| PHASE9_MultiTenant_Telemetry
```
