# Promotion and Rollback

The delivery model in this repository separates **build**, **promotion**, and **reconciliation**. That separation is useful when several environments share the same artifact but have different release policies.

## Build once

Jenkins validates the source and builds the container. The resulting artifact should be identified by an immutable tag or digest tied to the source revision.

A deployment should not rebuild the same source separately for each environment. Rebuilding makes it harder to prove that production is running the artifact that was tested earlier.

## Promote by changing desired state

Environment configuration lives in Git. A promotion changes the image reference in the target environment rather than embedding a hidden deployment command in the CI job.

```text
source revision
      |
      v
   Jenkins
      |
      v
immutable image
      |
      +------> dev desired state
      |
      +------> production desired state
                         |
                         v
                      Argo CD
                         |
                         v
                     Kubernetes
```

The exact promotion mechanism can be a pull request, an automated change with policy controls, or another reviewed Git workflow. The important property is that the desired state is visible and auditable.

## Rollback

Rollback should use the same path as deployment:

1. Identify the last known-good image.
2. Change the environment's desired image reference.
3. Review the change.
4. Let Argo CD reconcile it.
5. Wait for rollout and readiness checks.
6. Confirm application and platform health.

Avoid treating `kubectl rollout undo` as the only rollback strategy when GitOps is the source of truth. A manual cluster-only change can be overwritten by the next reconciliation.

## Release safety checks

For higher-risk releases, add checks around:

- workload readiness
- error rate
- latency
- replica availability
- recent Kubernetes events
- dependency health
- deployment progress

A failed check should stop promotion and leave enough information to explain why the release was blocked.
