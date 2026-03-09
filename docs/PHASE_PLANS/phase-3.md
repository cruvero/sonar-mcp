# Phase 3: Write Tools and Quality Gates/Profiles

## Overview
Execute the second scoped segment of this workstream with deterministic outputs and validation.

### Why Now
This phase implements core write tools (PHASE3_Write_Tools) and quality gates/profiles tools (PHASE3A_Quality_Gates_Profiles), which are direct prerequisites for downstream phases like PHASE4_Destructive_Polish (dependency edge PHASE3A_Quality_Gates_Profiles → PHASE4_Destructive_Polish, risk: 0.2). Write capabilities enable issue transitions and quality gate associations before bulk handling (>50 items) and gateway heartbeat/registration in later phases; delaying risks blocking SonarQube API integration testing against http://sonar.dev.gchinfo.com/.

**Exit Criteria Checklist:**
- [x] `go test ./pkg/tools/write/... -v -race` passes with workflow mocks
  - Evidence: Pass
- [x] Tools registered: curl /mcp/tools | jq 'map(select(.name | contains("transition")))' non-empty
  - Evidence: 2 tools

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P3-T01 | Implement issue/hotspot transition tool: `pkg/tools/transition_issue.go` with `TransitionIssue(ctx, params) (*mcpgo.CallResponse, error)` supporting RESOLVE/WONT_FIX/FALSE_POSITIVE via /api/issues/set_resolution?resolution=FIXED|WONT_FIX or /api/issues/set_type?type=FALSE_POSITIVE using client.UpdateIssue(key, transition). riskTier:1, projectKey stub. | Cruvero Plan Architect v2 | PHASE2A_Issues_Hotspots_Read (risk:0.3) | transition_issue_test.go mocks | go test -v |
| P3-T02 | quality_gates tools: list_quality_gates /api/qualitygates/list, associate_project /api/qualitygates/select, etc. riskTier:1. | Cruvero Executor v1 | P3-T01 | tools/qg_test.go | go test |
| P3-T03 | quality_profiles: list /api/qualityprofiles/search, etc. | Cruvero Executor v1 | P3-T02 | tools/profile_test.go | go test |
| P3-T04 | Audit phase-3. | Cruvero Plan Architect v2 | P3-T03 | phase-audit-3 conditional PASS |