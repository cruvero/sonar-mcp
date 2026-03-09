# Phase 6: Helm ArgoCD

## Overview
Execute the second scoped segment of this workstream with deterministic outputs and validation.

## Why Now
This phase deploys via Helm/ArgoCD (PHASE6_Helm_ArgoCD), depending on PHASE5_Gateway_Health → PHASE6_Helm_ArgoCD (risk: 0.1), preceding PHASE7_Bulk_RBAC_Stubs (risk: 0.3). Ensures runtime patterns match k8s mirror before bulk/CI.

**Exit Criteria Checklist:**
- [ ] Helm chart mirrors k8s, deploys dry-run.
  - Evidence: Pending
- [ ] ArgoCD values ready.
  - Evidence: Pending

## Detailed Task List
| Task ID | Task | Owner/Agent | Depends On | Deliverable | Validation Command |
|---|---|---|---|---|---|
| P6-T01 | Mirror/update charts/mcp-sonarqube from k8s (values: image, SONAR_URL/TOKEN secrets, replicas=1, resources). templates/deployment,service. | Cruvero Plan Architect v2 | phase-5 audit pass | charts/ | helm lint charts/mcp-sonarqube && helm template . |
| P6-T02 | ArgoCD app yaml stub: github.com/cruvero/cruvero-mcp-sonarqube//charts/mcp-sonarqube, values from secrets. | Cruvero Plan Architect v2 | P6-T01 | argocd-app.yaml | kubectl apply --dry-run=client -f argocd-app.yaml |
| P6-T03 | Test deploy local kind/minikube: port-forward curl /healthz. | Cruvero Executor v1 | P6-T02 | deploy logs | helm install mcp-sonarqube charts/ --kube-context kind; kubectl port-forward svc/mcp-sonarqube 8080:80; curl /healthz |
| P6-T04 | Audit phase-5/6. | Cruvero Plan Architect v2 | P6-T03 | phase-audit-5 |