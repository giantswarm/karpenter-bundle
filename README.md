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
