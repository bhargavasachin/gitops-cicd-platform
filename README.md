# GitOps CI/CD Platform

A reference delivery pipeline showing how source changes move through CI, artifact creation, Helm packaging and GitOps deployment. The examples are intentionally provider-neutral so the same workflow can be adapted to a Kubernetes platform running on OCI OKE or another cluster.

> **Portfolio note:** The design is based on delivery-platform patterns from professional work. Internal repositories, credentials, service names and proprietary release logic are not included.

## Flow

```text
Developer
   |
   v
Git repository
   |
   v
Jenkins CI  ---> tests / validation
   |
   v
Container image + immutable tag
   |
   v
Versioned GitOps configuration
   |
   v
Argo CD
   |
   v
Kubernetes
```

The important separation is between **building an artifact** and **promoting that artifact**. CI should produce a traceable artifact; the deployment system should make the desired state explicit and auditable.

## Repository layout

```text
.
├── app/
│   └── Dockerfile
├── helm/
│   └── platform-app/
├── jenkins/
│   └── Jenkinsfile
├── gitops/
│   └── environments/
│       └── dev/
├── argocd/
│   └── application.yaml
└── docs/
    └── release-strategy.md
```

## Design principles

### Immutable artifacts

Use an immutable image tag or digest for promotion. Avoid rebuilding the same source revision separately for each environment.

### Configuration is versioned

Helm values and GitOps configuration are reviewed as code. Environment differences are explicit instead of hidden inside pipeline scripts.

### CI does not need cluster-admin access

The Jenkins stage that builds and tests an image should not require broad production cluster credentials. Argo CD owns the deployment path.

### Rollback is a state change

A rollback should be a deliberate change to the desired version, followed by the same reconciliation and health checks used for a normal release.

### Promotion is explicit

The sample repository contains a development environment. A production environment would use the same chart with a separately reviewed values change and an explicit promotion boundary rather than silently changing the artifact during deployment.
