# MediFor Helm Chart

This repository provides a helm chart to deploy the MediFor system to a small Kubernetes cluster.
For our purposes we will use minikube as an example.

Custom configuration may be needed depending on your specific cluster setup.

For development instructions see **[here](docs/development.md).**

## Prerequisites

- Kubernetes Cluster 1.10+
- Helm 3.0.0+

### Generate Gitlab Token

1.  Navigate to [this page](https://gitlab.mediforprogram.com/profile/personal_access_tokens) to generate a new personal
    access token to the MediFor gitlab.
2.  Name the token `medifor-registry-token`, leave the expiration date blank, limit the
    scop to `read_repository` and `read_registry` then create the personal access token.
3.  Save the generated token somewhere safe.

## Installation

### Add Helm Chart

```bash
$ git clone https://gitlab.mediforprogram.com/medifor/medifor-helm-chart.git
$ cd medifor-helm-chart
```

### Add Credentials

Add your credentials in `values.yaml` and save.

```yaml
registrycredentials:
  registry: https://gitlab-registry.mediforprogram.com
  username: <GITLAB_USERNAME>
  password: <REGISTRY_TOKEN>
  email: <MEDIFOR_EMAIL>
```

### Deploy Chart

Install the medifor helm chart with a release name `medifor`

```bash
$ helm install medifor .
```

## Access UI

### Check Pods

Verify that all pods are running and containers have been created. If status is _ImagePullBackOff_ then there is an error with credentials.

```bash
$ kubectl --namespace=medifor-mini get pods
```

### Expose UI Service

By default the UI service will be exposed on port `30000` on every node in the cluster.

If you want to set up load balancing or use an ingress controller then custom configuration will be needed. See the [kubernetes services docs](https://kubernetes.io/docs/concepts/services-networking/service/) for more information.

Expose the UI Service on the minikube cluster to your local machine.

```bash
$ minikube service medifor-ui -n medifor-mini
```

## Uninstallation

Uninstall Helm chart with release name `medifor`.

```bash
$ helm uninstall medifor
```

## Persistence

Persistent Volume Claims are used to maintain the data across deployments. This system uses the default storage classes provided with your cluster and is verified to work on GCE, AWS and minikube.

If running your cluster on microK8s you may need to **[enable the default storage class](https://igy.cx/posts/setup-microk8s-rbac-storage/).** <br/>

If you are running in minikube then your data will be hosted in `/tmp` and it is recommended that you mount a directory from your host machine into the minikube instance.

## Troubleshooting

For more detailed instructions on working with this chart see **[here](docs/development.md).**

## Parameters

The following table lists the configurable parameters of the Medifor Helm Chart and their default values.

| Parameter                                     | Description                                                                                                                                                                             | Default                                                                                                      |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `analytics`                                   | List of analytics to run in the system                                                                                                                                                  | `ela-base` and `exif` are run by default. Contact your system admin to get access to the analytic repository |
| `analyticmounts.input.external`               | External input path of the medifor-data persistent volume                                                                                                                               | `medifor/input`                                                                                              |
| `analyticmounts.input.internal`               | Internal input path of the medifor-data persistent volume (This is location inside the container)                                                                                       | `mnt/medifor/input`                                                                                          |
| `analyticmounts.output.external`              | External output path of the medifor-data persistent volume                                                                                                                              | `medifor/output`                                                                                             |
| `analyticmounts.output.internal`              | Internal output path of the medifor-data persistent volume (This is location inside the container)                                                                                      | `mnt/medifor/output`                                                                                         |
| `analyticworker.id`                           | Id for the analyticworkflow resource                                                                                                                                                    | `analyticworkflow`                                                                                           |
| `analyticworker.version`                      | Version for the analyticworker resource                                                                                                                                                 | `develop`                                                                                                    |
| `analyticworker.config_mount`                 | Location where the analytic list will be mounted inside the workflow container                                                                                                          | `/mnt/config/analytic_config.json`                                                                           |
| `analyticworker.container`                    | Docker image URL for the analyticworker resource                                                                                                                                        | `gitlab-registry.mediforprogram.com/medifor/analytic-worker analyticworker:develop`                          |
| `analyticworker.ports.grpc.port_number`       | GRPC port for the analyticworker container <br/> **RECOMMEND NOT CHANGING**                                                                                                             | `50051`                                                                                                      |
| `analyticworker.ports.http.port_number`       | HTTP port for the analyticworker container <br/> **RECOMMEND NOT CHANGING**                                                                                                             | `50052`                                                                                                      |
| `analyticworker.ports.prometheus.port_number` | Port for prometheus metrics on analyticworker container <br/> **RECOMMEND NOT CHANGING**                                                                                                | `2112`                                                                                                       |
| `autoscaler.enabled`                          | Boolean flag to turn on/off the autoscaler resource                                                                                                                                     | `false`                                                                                                      |
| `entroq.ports.grpc`                           | Port that the entroq service listens on inside the container for grpc requests <br/> **RECOMMEND NOT CHANGING**                                                                         | `37706`                                                                                                      |
| `entroq.ports.metrics`                        | Port that the entroq service listens on inside the container for http requests for prometheus metrics <br/> **RECOMMEND NOT CHANGING**                                                  | `8080`                                                                                                      |
| `fusers`                                      | List of fusion analytics to run in the system                                                                                                                                           | `ta2-avg` is run by default. Contact your system admin to get access to the analytic repository              |
| `namespace`                                   | The k8s namespace that will be created for the medifor system resources                                                                                                                 | `false`                                                                                                      |
| `persistence.medifor.storageClass`            | Name of the medifor-pvc custom storage class resource                                                                                                                                   | `undefined`                                                                                                  |
| `persistence.medifor.accessMode`              | Access mode for the medifor-pvc                                                                                                                                                         | `ReadWriteMany`                                                                                              |
| `persistence.medifor.size`                    | Size of the medifor-pvc                                                                                                                                                                 | `2Gi`                                                                                                        |
| `persistence.postgres.storageClass`           | Name of the postgres-pvc custom storage class resource                                                                                                                                  | `undefined`                                                                                                  |
| `persistence.postgres.accessMode`             | Access mode for the postgres-pvc                                                                                                                                                        | `ReadWriteOnce`                                                                                              |
| `persistence.postgres.size`                   | Size of the postgres-pvc                                                                                                                                                                | `5Gi`                                                                                                        |
| `prometheus.enabled`                          | Boolean enable/disbale prometheus montoring                                                                                                                                             | `false`                                                                                                      |
| `postgres.database`                           | Name of postgres database                                                                                                                                                               | `postgres`                                                                                                   |
| `postgres.username`                           | Postgres username                                                                                                                                                                       | `postgres`                                                                                                   |
| `postgres.database`                           | Postgres password                                                                                                                                                                       | `postgres`                                                                                                   |
| `postgres.port`                               | Port that postgress listens on inside the container                                                                                                                                     | `5432`                                                                                                       |
| `registrycredentials`                         | These are your credentials for the gitlab registry                                                                                                                                      | See above documentation                                                                                      |
| `replicas.ui`                                 | Number of pods to create for UI Deployment                                                                                                                                              | `1`                                                                                                          |
| `replicas.fusion`                             | Number of pods to create for each Fusion Deployment                                                                                                                                     | `1`                                                                                                          |
| `replicas.analytic`                           | Number of pods to create for each Analytic Deployment                                                                                                                                   | `1`                                                                                                          |
| `ui.env.CONTAINER_ROOT`                       | Directory that the UI container will write static resources to                                                                                                                          | `/medifor`                                                                                                   |
| `ui.env.WORKFLOW_HOST`                        | Name of the analyticworkflow service. <br/> **This needs to match `analyticworker.id` <br/>RECOMMEND NOT CHANGING**                                                                     | `analyticworkflow`                                                                                           |
| `ui.env.WORKFLOW_PORT`                        | GRPC port that the analyticworklow service listens on. <br/> **This needs to match `analyticworker.ports.grpc.port_number` <br/>RECOMMEND NOT CHANGING**                                | `50051`                                                                                                      |
| `ui.env.PORT`                                 | Port that the node server listens on. <br/> **This needs to match `ui.port` <br/>RECOMMEND NOT CHANGING**                                                                               | `3000`                                                                                                       |
| `ui.env.CACHE_TTL_SECONDS`                    | Number of seconds that the node server will cache the analytic list response from the pipeline before issuing a new request                                                             | `600`                                                                                                        |
| `ui.env.UI_ENABLE_GROUPS`                     | Boolean flag to enable the user grouping feature in the UI. <br/> **Note that groups must be defined in the request headers for full functionality**                                    | `false`                                                                                                      |
| `ui.env.UI_ENABLE_DELETE`                     | Boolean flag to allow users to delete their own uploaded media                                                                                                                          | `true`                                                                                                       |
| `ui.env.UI_DEFAULT_FUSER_ID`                  | ID of a fusion model that the system will default to on initial load. This would need to be consistent will a value in `fusers.id`. UI will auto-select first in list if left undefined | `""`                                                                                                         |
| `ui.env.UI_UNKNOWN_USERS`                     | "Deny"/"Allow" on how the system handles users that aren't verified                                                                                                                     | `deny`                                                                                                       |
| `ui.env.UI_GROUP_PREFIX`                      | Needs to match prefix that usergroups are appended with in the request headers **RECOMMEND NOT CHANGING**                                                                               | `medifor-uigrp-`                                                                                             |
| `ui.env.UI_USER_TAG_PREFIX`                   | Prefix to append to system level user tags <br/> **RECOMMEND NOT CHANGING**                                                                                                             | `user`                                                                                                       |
| `ui.env.UI_GROUP_TAG_PREFIX`                  | Prefix to append to system level group tags <br/> **RECOMMEND NOT CHANGING**                                                                                                            | `group`                                                                                                      |
| `ui.env.UI_TAG_PREFIX_FLAG`                   | Flag to append to system level group/user tags `__user/__group` <br/> **RECOMMEND NOT CHANGING**                                                                                        | `__`                                                                                                         |
| `ui.name`                                     | Name for the ui resource                                                                                                                                                                | `medifor-ui`                                                                                                 |
| `ui.container`                                | Docker image URL for the UI resource                                                                                                                                                    | `gitlab-registry.mediforprogram.com/medifor/medifor-demo-ui:develop`                                         |
| `ui.service.type`                             | Type of service used to expose the UI deployment resource                                                                                                                               | `NodePort`                                                                                                   |
| `ui.service.nodePort`                         | Port to expose the UI service on across all cluster nodes                                                                                                                               | `30000`                                                                                                      |
| `ui.port`                                     | Port that the node server listens on inside the container <br/> **RECOMMEND NOT CHANGING**                                                                                              | `3000`                                                                                                       |
