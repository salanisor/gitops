### Remote bases

How to convert a cluster over from a local to a remote base.

In the traditional deployment, we've been deploying cluster configuration inside the local mono-repo at path > **components** > **infra** >

Moving forward, we have a git branch cluster-config dedicated to managing cluster specific component configurations.

Under the target environment path **clusters** > **overlays** > **ocpapp-np** > **component** > **namespace** > **kustomization.yaml** definition. You will now configure the remote base to the aforementioned **cluster-config** branch specific to the component being configured (see [example](https://github.com/salanisor/openshift-gitops/wiki/How-to:-Referencing-external-bases-in-GitOps#example)).

The breakdown of the remote base is as follows based on a `SSH` connection:

 * `git@github.com:salanisor/openshift-gitops.git` - is the github repository containing the cluster configuration.
 * `/components/infra/openshift-gitops/base` - the directory path at the github repository pointing to the **openshift-gitops** component base.
 * `?ref=cluster-config` - states that for the given repository and directory path to consume the `cluster-config` branch.

`git@github.com:salanisor/openshift-gitops.git/components/infra/openshift-gitops/base?ref=cluster-config`

### Example 
~~~
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- git@github.com:salanisor/openshift-gitops.git/components/infra/openshift-gitops/base?ref=cluster-config

patches:
- path: rbac-policy.json
  target:
    group: argoproj.io
    kind: ArgoCD
    name: openshift-gitops
    version: v1beta1
~~~
