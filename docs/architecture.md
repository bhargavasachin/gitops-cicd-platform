# Delivery architecture

The pipeline separates artifact creation from deployment state.

```text
Developer
   |
   v
Git commit
   |
   v
Jenkins CI
   |  test / lint / build
   v
Immutable image
   |
   v
Environment configuration in Git
   |
   v
Argo CD
   |
   v
Kubernetes
   |
   +--> rollout health
   +--> application health
```

## Responsibilities

**Jenkins** builds and verifies the artifact. It should not become the long-lived source of deployment state.

**Git** records the desired environment configuration and provides the review/audit boundary for promotion.

**Argo CD** reconciles the desired state into the cluster and reports synchronization/health.

**Kubernetes** provides workload orchestration and runtime health signals.

## Why keep deployment state out of CI?

A CI job can finish successfully while a deployment later becomes unhealthy. Keeping desired state in Git makes the target version visible, reviewable and reproducible, while the GitOps controller continuously reconciles that state.

## Failure boundaries

- Build/test failure: no artifact promotion.
- Registry failure: deployment cannot obtain the artifact.
- Git configuration failure: desired state is not updated.
- Argo CD synchronization failure: desired and live state diverge.
- Kubernetes rollout failure: workload does not become ready.
- Application failure: workload may be running while user-facing behavior is degraded.

Each boundary has a different diagnostic path; treating all failures as "the deployment failed" makes recovery slower.
