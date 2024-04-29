[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/karpenter-bundle/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/karpenter-bundle/tree/main)

[Read me after cloning this template (GS staff only)](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# karpenter-bundle chart

Giant Swarm offers a karpenter-bundle App which can be installed in workload clusters.
Here we define the karpenter-bundle chart with its templates and default configuration.

**What is this app?**

Karpenter bundle will install 3 applications:
- Karpenter App: Deployed in workload cluster and deploys the upstream chart with some additional configuration.
- Kapenter Taint Remover: Removes CAPI taint as the nodes are managed our of band.
- Karpenter Crossplane Resources: deployed on the management cluster, deploys all AWS resources needed for Karpenter.

**How to use Karpenter to provision nodes?**

## Creating a custom provisioner:
We will create a provisioner that will reuse the Launch Template and Subnets of the previously created by a Nodepool.

Replace `<CLUSTER_ID>` and `<NODEPOOL_ID>` in the following template to create the first provisioner. 

In CAPA, using TTL is the only way to renew nodes, we recomend a maximum of 3 days of TTL for the provisioner in order to make sure nodes are not stale for too long even after upgrades.

Apply this resource on the Workload Cluster after having installed Karpenter:

```
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: spot-provisioner
spec:
  consolidation:
    enabled: true

  ttlSecondsUntilExpired: 86400 # 1 Days = 60 * 60 * 24 Seconds;

  # Make sure pods are not scheduled until cilium is running
  startupTaints:
  - key: node.cilium.io/agent-not-ready
    value: "true"
    effect: NoExecute
  - key: node.cluster.x-k8s.io/uninitialized
    value: "true"
    effect: NoExecute

  # Labels are arbitrary key-values that are applied to all nodes
  labels:
    managed-by: karpenter
    nodepool: <NODEPOOL_ID>
    cluster: <CLUSTER_ID>
    node.kubernetes.io/worker: ""
    role: worker

  # Requirements that constrain the parameters of provisioned nodes.
  # These requirements are combined with pod.spec.affinity.nodeAffinity rules.
  # Operators { In, NotIn } are supported to enable including or excluding values
  requirements:
    - key: "karpenter.k8s.aws/instance-family"
      operator: NotIn
      values: ["t3", "t3a", "t2"]
    - key: "karpenter.k8s.aws/instance-size"
      operator: NotIn
      values: ["xlarge"]
    - key: "karpenter.k8s.aws/instance-hypervisor"
      operator: In
      values: ["nitro"]
    - key: "kubernetes.io/arch"
      operator: In
      values: ["amd64"]
    - key: "karpenter.sh/capacity-type" # If not included, the webhook for the AWS cloud provider will default to on-demand
      operator: In
      values: ["spot"]


  # Resource limits constrain the total size of the cluster.
  # Limits prevent Karpenter from creating new instances once the limit is exceeded.
  limits:
    resources:
      cpu: "4000"
      memory: 4000Gi

  # CAPA provider example
  provider:
    subnetSelector:
      giantswarm.io/cluster: <CLUSTER_ID>
      sigs.k8s.io/cluster-api-provider-aws/role: private
    # For EKS the launchtemplate dos not follow this convention and need to be gathered from the AWS Console.
    launchTemplate: <CLUSTER_ID>-<NODEPOOL_ID>
    tags:
      managed-by: karpenter
      nodepool: <NODEPOOL_ID>
      cluster: <CLUSTER_ID>
      giantswarm.io/cluster: <CLUSTER_ID>
      Name: <CLUSTER_ID>-karpenter-spot-worker
```


## Use the default provisioner:
Use these values to install this app and a default Spot only provisioner will be deployed in the cluster. Only works for CAPA clusters, not EKS.

```
karpenter:
  defaultProvisioner:
    enabled: true
    nodePoolName: nodepool0
```