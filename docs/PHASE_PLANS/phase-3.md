# Phase 3: Write Tools and Quality Gates/Profiles

## Overview
Execute the second scoped segment of this workstream with deterministic outputs and validation.

### Why Now
This phase implements core write tools (PHASE3_Write_Tools) and quality gates/profiles tools (PHASE3A_Quality_Gates_Profiles), which are direct prerequisites for downstream phases like PHASE4_Destructive_Polish (dependency edge PHASE3A_Quality_Gates_Profiles → PHASE4_Destructive_Polish, risk: 0.2). Write capabilities enable issue transitions and quality gate associations before bulk handling (>50 items) and gateway heartbeat/registration in later phases; delaying risks blocking SonarQube API integration testing against http://sonar.dev.gchinfo.com/.

**Exit Criteria Checklist:**
- [ ] `go test ./pkg/tools/write/... -v -race` passes with workflow mocks
  - Evidence: Pending
- [ ] Tools registered: curl /mcp/tools | jq 'map(select(.name | contains("transition")))' non-empty
  - Evidence: Pending

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P3-T01 | Implement issue/hotspot transition tool: `tools/transition_issue.go` with `TransitionIssue(ctx context.Context, params mcpgo.Params) (*mcpgo.CallResponse, error)` supporting statuses RESOLVE/WONT_FIX/FALSE_POSITIVE via SonarQube /api/issues/set_type?issue=KEY&set_type=... using `sonarqube/client.go::UpdateIssue(key, transition)`. Add riskTier:"write", projectKey scoping (stub RBAC allow-all). | Cruvero Plan Architect v2 | PHASE2A_Issues_Hotspots_Read (edge risk: 0.3), phase-2 audit pass | `tools/transition_issue.go`, `sonarqube/client.go::UpdateIssue`, tool registration in `cmd/server/main.go`. | go test ./pkg/tools/transition_issue -v |
| P3-T02 | Implement quality_gates/profiles list/update tools. | Cruvero Plan Architect v2 | P3-T01 | list_quality_gates.go | go test -v |
| P3-T03 | Add bulk_transition_issues stub (>50 guard). | Cruvero Plan Architect v2 | P3-T02 | bulk_transition.go | go test ./pkg/tools/bulk |
| P3-T04 | Integration tests vs real sonar.dev.gchinfo.com. | Cruvero Plan Architect v2 | P3-T03 | *_test.go | make integration |