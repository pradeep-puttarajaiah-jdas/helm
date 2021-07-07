# WMD Helm Charts

This repository represents the helm chart used to deploy [Refs](https://stash.jda.com/projects/WEB/repos/rpweb/browse) and 
[WMS](https://stash.jda.com/projects/INV/repos/wmd/browse) into a Kubernetes Cluster.

Environment specific variables will be injected/overridden in the repos that represent the Azure environments:
 - [dev](https://stash.jda.com/projects/LGS-SAASOPS/repos/lgs-dev/browse)
 - [staging](https://stash.jda.com/projects/LGS-SAASOPS/repos/lgs-staging/browse)
 - [psr](https://stash.jda.com/projects/LGS-SAASOPS/repos/lgs-psr/browse)
 - [production](https://stash.jda.com/projects/LGS-SAASOPS/repos/lgs-prod/browse)

The automation steps done by Jenkins is as follows:
1. Builds any dependencies the chart may have. `helm dependency build`
1. Lints chart `helm lint`
1. Validates the chart can produce valid yaml `helm template`
1. Package the helm chart `helm package .` into a tarball.
1. Publish the packaged helm chart into artifactory located at `https://jdasoftware.jfrog.io/jdasoftware/helm/logistics`
1. Push a git tag to repository equal to the `version` defined in the `Chart.yaml`.

# Requirements

Helm [3.1.0](https://github.com/helm/helm/releases/tag/v3.1.0)
