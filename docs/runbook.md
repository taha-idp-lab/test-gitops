# Runbook

What to check when **test-gitops** misbehaves.

## Is it alive?

```bash
curl -s http://<host>/health      # expected: {"status": "ok"}
```

In Kubernetes:

```bash
kubectl get pods -n test-gitops
kubectl describe pod <pod> -n test-gitops   # probe failures, OOMKilled, image pull errors
kubectl logs <pod> -n test-gitops
```

## Common issues

| Symptom | Likely cause | Action |
|---|---|---|
| Pod in `CrashLoopBackOff` | The app fails at startup | Check `kubectl logs`, then the last commit |
| Pod `OOMKilled` | Memory limit (256Mi) reached | Investigate the leak, or raise `resources.limits.memory` in the GitOps values |
| `ImagePullBackOff` | Wrong image tag, or image not published | Check the CI run and GHCR |
| CI fails at *Scan image with Trivy* | Fixable CRITICAL/HIGH vulnerability | Update the affected dependency or base image |

## Rollback

Deployments are managed by GitOps: revert the image tag in the `idp-gitops`
repository (or roll back from ArgoCD) to return to the previous version.

## Metrics

Prometheus scrapes `GET /metrics`. Useful series:

- `http_requests_total` — requests by handler, method and status
- `http_request_duration_seconds` — latency histogram
