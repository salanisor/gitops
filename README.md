<h2 align="center">GitOps Directory Structure</h2>

<p align="center">
  <img src="https://github.com/salanisor/gitops/blob/master/yaml/gitops.png" height="1000"/>
</p

* How does the tenants directory structure look like in actuality?

~~~
tenants/ (A)
├── dev-nadc (B)
│   ├── apps (C)
│   │   └── kustomization.yaml
│   ├── dev-gateway-service (D)
│   │   ├── app (E)
│   │   │   ├── dev-gateway-service.yaml
│   │   │   └── kustomization.yaml
│   │   └── namespace (F)
│   │       ├── kustomization.yaml
│   │       └── namespace.yaml
│   ├── dev-web-api-service
│   │   ├── app
│   │   │   ├── dev-web-api-service.yaml
│   │   │   └── kustomization.yaml
│   │   └── namespace
│   │       ├── kustomization.yaml
│   │       └── namespace.yaml
│   ├── namespace (G)
│   │   └── kustomization.yaml
~~~

* (A) - Top level directory contains all cluster(s) ordered by their respective name.
* (B) - Target cluster(s) directory containing all namespace(s) by their respective name.
* (C) - `apps` directory holding the `kustomization.yaml` file pointing to individual ArgoCD `application` definitions by their respective `namespace`.
* (D) - Directory ordered by the actual `namespace` name.
* (E) - Subdirectory `app` containing the actual `application` definitions provided by the application team(s).
* (F) - Subdirectory `namespace` containing the actual `namespace` object definitions required to manage the tenant namespace.
* (G) - `namespace` directory holding the `kustomization.yaml` file pointing to individual `namespace(s)` object definitions by their respective `namespace`.

***


## **Onboarding a new namespace & application**

This example walks you through as if you've receive a request to create a `namespace` called `test-namespace` in the `commops nonprd` cluster.

1. For the given environment, match the branch name (e.g. `nonprd`).
2. Create a directory under > **tenants** > **cluster name** (e.g. `dev-nadc`) with the name of the requested `namespace` (must be 52 characters max long)

    `mkdir -p tenants/nonprd-nad/test-namespace/{app,namespace}`

3. Generate the `yaml` definitions with the appropriate values under the directory created during step #2 under > **tenants** > **cluster-name** > **namespace name** > **namespace**

**Example template yaml definitions**

**`kustumization.yaml`**
~~~
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- namespace.yaml
~~~~

**`namespace.yaml`**
~~~
apiVersion: v1
kind: Namespace
metadata:
  name: test-namespace
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: test-ns-quota
  namespace: test-namespace
spec:
  hard:
    pods: "10" 
    requests.cpu: "500m" 
    requests.memory: 1Gi 
    limits.cpu: "1" 
    limits.memory: 2Gi 
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: edit
  namespace: test-namespace
subjects:
  - kind: Group
    apiGroup: rbac.authorization.k8s.io
    name: k8s-nonprod-developer
  - kind: Group
    apiGroup: rbac.authorization.k8s.io
    name: k8s-nonprod-cd
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: edit
~~~

4. If an application definition is included as part of the request. Generate the app `yaml` definition with the name of the service using the provided values under the directory created during step #2 under > **tenants** > **cluster-name** > **namespace name** > **app**

**`test-namespace-service.yaml`**
~~~
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "200"
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  labels:
    gitops.ownedBy: k8s-nonprod-group
  name: test-namespace-service
  namespace: openshift-gitops
spec:
  destination:
    namespace: test-namespace
    server: https://kubernetes.default.svc
  project: k8s-nonprod-group
  source:
    path: nonprd
    repoURL: git@github.example.com:ACME/test-namespace-service-gitops-non-prod.git
    targetRevision: HEAD
~~~

5. Next, you must update the `kustomization.yaml` file to let ArgoCD aware of the new `namespace`.

* Update for the newly added **test-namespace** under > **tenants** > **cluster-name** > **namespace folder** > **kustomization.yaml** > update the `relative path` to the newly created directories as follows: > (**Note**: this pointing to the `namespace` subdirectory)

~~~
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../test-namespace/namespace/
~~~

6. Last but not least, you must update the `kustomization.yaml` file to let ArgoCD aware of the new ArgoCD `Application` added as part of step #4.

* Update for the newly added `Application` **test-namespace-service** under > **tenants** > **cluster-name** > **apps folder** > **kustomization.yaml** > update the `relative path` to the newly created directories as follows: > (**Note**: this pointing to the `app` subdirectory)

~~~
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../test-namespace/app/
~~~

7. Check the code in to git (merge into the target branch if appropriate) and sync in ArgoCD to generate the defined objects.
