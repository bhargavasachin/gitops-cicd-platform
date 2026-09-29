# Promotion checklist

The pipeline should make the release decision visible instead of hiding it in a long script.

## Build

- Source revision is identified.
- Tests and static checks pass.
- Image is built once.
- The image receives an immutable identifier.

## Promote

- The exact artifact to deploy is recorded.
- Environment configuration is reviewed independently of the build.
- Production changes go through the normal review/approval boundary.
- Argo CD is the deployment authority for the target cluster.

## Verify

Check both Kubernetes state and application behavior:

```bash
kubectl -n platform-demo rollout status deployment/platform-app
kubectl -n platform-demo get pods
kubectl -n platform-demo get endpoints
```

A green pipeline is not enough if the application is not ready or Service endpoints are missing.

## Rollback

If the new revision cannot reach the expected healthy state, revert the desired image version in Git and allow Argo CD to reconcile it. The rollback should use the same observable deployment path rather than a separate emergency mechanism that is difficult to audit.

## Useful distinction

**Build failure:** the artifact should not be promoted.

**Deployment failure:** the artifact exists, but the desired state did not become healthy.

**Application failure after deployment:** Kubernetes may report a healthy workload while user-facing behavior is degraded; application-level signals are required for this case.
