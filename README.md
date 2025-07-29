[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/karpenter-bundle/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/karpenter-bundle/tree/main)

# karpenter-bundle chart

Karpenter bundle will install:
- Karpenter App: Deployed in workload cluster and deploys the upstream chart with some additional configuration.
- Karpenter Crossplane Resources: deployed on the management cluster, deploys all AWS resources needed for Karpenter.
