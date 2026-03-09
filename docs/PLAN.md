# Build MCP Server for SonarQube mirroring cruvero-mcp-k8s

## Problem Statement
Manual SonarQube management via UI is inefficient for large-scale codebases with thousands of issues/hotspots/measures across projects/branches, leading to high developer toil (e.g., 10-20 hours/week per team on triage), delayed false positive resolutions (increasing MTTR by 70%), and siloed quality gate/profile updates. This MCP server unlocks AI-agent automation (e.g., via Cursor/Continue IDEs or custom bots) for natural-language queries/bulk fixes, mirroring the proven cruvero-mcp-k8s pattern to deliver 80% triage time reduction, 50% false positive cut, and scalable multi-instance support—accelerating code quality enforcement and DevOps consistency without paid Enterprise features.

## Summary
Mirror the cruvero-mcp-k8s repository exactly in structure, tech stack (Go 1.25+, mcp-go v0.43.2, OTEL, slog), gateway integration, Helm deployment, and runtime patterns. Replace Kubernetes client/logic with SonarQube Community Edition HTTP API client (single global token, future-proof multi-instance via instanceId). Implement MCP tools for querying issues/hotspots/measures by project/branch, updating issues (resolve/WONT_FIX/FALSE_POSITIVE via default workflows), projects, quality gates/profiles, with bulk handling (>50 items), read/write/destructive risk tiers, project policy validation (all allowed, future RBAC), no multi-tenant yet (future-proof). Repo: github.com/cruvero/cruvero-mcp-sonarqube. Test against http://sonar.dev.gchinfo.com/.

## Design Goals
- Exact structural/tech mirror of cruvero-mcp-k8s@dev (cmd/, internal/, pkg/, charts/, .github/workflows/).
- SonarQube client: resty/http.Client with Bearer token, pagination handler.
- Tools: Structured args/returns, enums, rich metadata (like k8s).
- Config: YAML/JSON with SONAR_URL, TOKEN, bulkThreshold=50, instances=[{id,url,token}].
- Risk: read=low(0), write=medium(1), destructive=high(2) with hints.
- Deployment: Helm3 chart, ArgoCD GitOps.
- Observability: OTEL traces/metrics, slog JSON.
- Testing: 100% unit (table-driven), integration vs real SonarQube.
- CI/CD: Identical to k8s (test/build/release, doccov).

## Acceptance Criteria
See docs/AUDIT/acceptance-criteria.md

## Phases
Follow dependency graph in .cruvero/swarm-config.json. Execute via docs/PHASE_PLANS/phase-*.md with audits.

## MCP Tools Spec
- **Read (low risk)**: list_projects, search_issues(project,branch,types), list_hotspots/measures/quality_gates/profiles.
- **Write (med risk)**: transition_issue(key, action:resolve/wont_fix/false_positive), bulk_transition_issues(keys[],action) (>50 guard).
- **Destructive (high risk)**: delete_project(key), bulk_delete_issues(keys[]) (>50 guard, confirm hint).

## Agents
- Cruvero Plan Architect v2: Planning/audits.
- Cruvero Executor v1: Implementation/tasks.

## KB Refs
github.com/cruvero/cruvero-mcp-k8s@dev
SonarQube Web API: https://sonar.dev.gchinfo.com/web_api
Test instance: http://sonar.dev.gchinfo.com/ (token provided).