# CI/CD and GitOps Delivery

This document describes the sanitized delivery pattern used to move banking application changes from source control into RKE2 environments without making the cluster itself the source of truth.

The operating model separated **continuous integration**, **artifact storage**, **deployment configuration**, and **continuous delivery**:

```text
Developer / Merge Request
          |
          v
 Application Git repository
          |
          v
      CI pipeline
   build / test / validate
          |
          +------------------------------+
          | deployment-approved branch   |
          v                              |
  build + tag container image            |
          |                              |
          v                              |
   private Harbor registry               |
          |                              |
          v                              |
     GitOps repository <-----------------+
 chart + service values + env values
          |
          v
       Argo CD
          |
    reconcile desired state
          |
          v
      RKE2 cluster
          |
   Deployment / Service
   ConfigMap / Secret refs
          |
          v
 rollout + runtime validation
```

The exact repository names, registry paths, internal domains, credentials, and customer identifiers are intentionally omitted.

## 1. CI responsibilities

The application pipeline was responsible for proving that a source revision was buildable before it could become a deployable artifact.

Depending on the service, the CI stages included:

- application build
- automated tests and validation
- Docker image build from the repository Dockerfile
- version/tag calculation
- authentication to the private container registry
- image push to Harbor
- update of the GitOps repository when a deployment was intended

A key principle was that **not every source-control event should deploy an application**.

## 2. Branch and merge gates

A representative banking-service pipeline used merge-based controls rather than deploying on every direct push.

| Event | Pipeline behavior |
| --- | --- |
| Merge request | Build and test only; no deployment |
| Merge into the deployment branch | Build, test, update version, build/push image, update GitOps deployment state |
| Merge into the protected main branch | Build and test only |
| Direct push to protected/deployment branches | Not used as the normal deployment trigger |

This separated **code validation** from **environment promotion** and reduced accidental deployments from ordinary development activity.

The specific branch names varied by project; the important design was the promotion rule, not the literal branch name.

## 3. Container artifact flow

After a deployment-approved merge passed CI, the pipeline produced a versioned container image and pushed it to a private Harbor registry.

```text
source revision
     |
     v
application build
     |
     v
Docker image
     |
     +--> service:<version>
     |
     v
Harbor
```

The RKE2 cluster did not build application images. Kubernetes consumed artifacts that had already passed the CI path.

Image-pull credentials were supplied to workloads through Kubernetes image-pull Secrets rather than embedded into application manifests or container images.

## 4. Shared Helm chart and per-service GitOps values

The GitOps repository used a shared service chart with service-specific and environment-specific values rather than copying complete Kubernetes manifests for every application.

A representative sanitized structure was:

```text
charts/
  service-chart/

services/
  <namespace>/
    <service>/
      values.yaml
      values-uat.yaml
      values-prod.yaml

applicationsets/
  <namespace>/
    ...
```

The shared chart supplied the reusable Kubernetes resource structure while the service files supplied items such as:

- image repository and tag
- container/service ports
- resource requests and limits
- ConfigMap references
- Secret references
- health probes where applicable
- Service type and exposure settings
- environment-specific overrides

This reduced duplication and made differences between environments easier to review.

## 5. Argo CD ApplicationSet delivery

Argo CD watched the GitOps repository and generated environment-specific Applications through ApplicationSets.

```text
GitOps desired state
        |
        v
ApplicationSet
        |
        +--> service-uat
        +--> service-prod
        |
        v
Argo CD Application
        |
        v
Helm render
        |
        v
RKE2 namespace
```

The Git repository represented the intended deployment state. Argo CD then reconciled that state into Kubernetes.

Operationally, this meant a Git commit alone was not considered proof of a successful deployment. The delivery path still had to be verified in Argo CD and Kubernetes.

## 6. Deployment verification

Validation was performed layer by layer:

```bash
kubectl -n <namespace> get deploy,svc,configmap,pods
kubectl -n <namespace> rollout status deployment/<service>
kubectl -n <namespace> describe pod <pod>
kubectl -n <namespace> logs <pod> --tail=200
```

Argo CD checks included:

- Application exists
- desired revision is visible
- sync state is expected
- health state is expected
- hard refresh when repository/cache state needs to be re-evaluated

Kubernetes checks included:

- Deployment replica availability
- image tag actually running
- `ImagePullBackOff` / registry authentication failures
- `ContainerCreating` causes
- ConfigMap and Secret references
- Service type, ports, endpoints, and external exposure where applicable
- readiness/startup behavior
- application logs and downstream dependency connectivity

Finally, the consuming business/application path was tested. A green Git commit or a reachable Service was not treated as sufficient evidence on its own.

## 7. Troubleshooting model

The delivery chain was treated as independent layers:

```text
Source merge succeeded
        !=
CI pipeline succeeded
        !=
Image exists in Harbor
        !=
GitOps repository updated correctly
        !=
Argo CD synchronized
        !=
Pod pulled and started
        !=
Service has working endpoints
        !=
Application is healthy end to end
```

This distinction matters during incidents. For example:

- a healthy Argo CD Application can still point to an image tag that cannot be pulled
- a successfully pulled image can still crash because of runtime configuration
- a Running pod can still fail readiness or downstream dependency checks
- a Service can exist while selecting no healthy endpoints
- a GitOps change can be correct while Argo CD still needs its repository state refreshed

The investigation starts by identifying **which boundary first stops being true**.

## 8. Rollback and recovery

Rollback favored restoring known desired state rather than leaving undocumented manual changes inside the cluster.

Depending on the failure, recovery could include:

- reverting the GitOps commit
- restoring the previously known-good image tag
- correcting an environment values file
- fixing image-pull or runtime Secret references
- reverting configuration in Git and allowing Argo CD to reconcile

Emergency cluster-side changes may sometimes be necessary during incident response, but they should be reconciled back into Git afterward so Git and runtime state do not permanently diverge.

## 9. Why this model was useful

The pipeline created a clear separation of responsibility:

- **CI** proved the code could be built and tested.
- **Harbor** stored the deployable container artifact.
- **GitOps** defined what version and configuration should run.
- **Argo CD** reconciled that desired state.
- **RKE2** ran the workload.
- **runtime verification** proved the application actually worked.

In short:

> Build once, store the artifact, declare the desired deployment in Git, let Argo CD reconcile it, and verify the running system independently.

## Evidence boundaries

This public version deliberately excludes:

- private Git server and registry hostnames
- repository and customer names that identify production systems
- credentials, tokens, certificates, and Kubernetes Secret values
- proprietary application source code
- exact internal namespace/IP/domain mappings where disclosure is unnecessary

The purpose is to demonstrate the delivery architecture and operational reasoning without reproducing a customer environment.