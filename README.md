# MediFor Helm Chart

This repository provides a helm chart to deploy the MediFor system to a small Kubernetes cluster. 
For our purposes we will use minikube as an example.

Custom configuration may be needed depending on your specific cluster setup.

## Prerequisities 

These deployments are running on Minikube, which is a single-node Kubernetes cluster.
If Minikube is not setup on the desired machine, use the Kubernetes documentation
to [install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).

Directions for installing the Helm CLI can be found [here](https://helm.sh/docs/intro/install/).


## Setup
### Environment
```bash
$ git clone https://gitlab.mediforprogram.com/NextCentury/medifor-helm-chart
$ cd medifor-helm-chart
```
Verify the chart has no linting errors
```bash
$ helm lint .
```
If there are no errors, the result should be
```bash
==> Linting .
INFO Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

### Running Minikube
```bash
$ minikube start
```
Verify Minikube status
```bash
$ minikube status
```
If running properly, the result should be
```bash
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### Generate Gitlab Token
1.  Navigate to [this page](https://gitlab.mediforprogram.com/profile/personal_access_tokens) to generate a new personal
access token to the MediFor gitlab.
2. Name the token `medifor-registry-token`, leave the expiration date blank, limit the
scop to `read_repository` and `read_registry` then create the personal access token.
3. Save the generated token somewhere safe 

<br/>
<br/>

## Working with the chart
### Chart structure and features
All helm charts have the following structure
```bash
mychart/
    Chart.yaml
    values.yaml
    templates/
    charts/
    ...
```
- **Chart.yaml** - Contains a description of the chart and its dependencies (which are known as *subcharts* ).
- **values.yaml** - Contains the default values for the chart that are used in the template files.
- **templates/** - Directory for all template files. When helm evaluates a chart it will send all of the files in this directory
                     through a template rendering engine. It then collects the results of those templates and sends them to Kubernetes.
- **charts/** - Directory that contains other charts, known as *subcharts*, which this chart depends on. Subcharts must be able to exist independently
                  from their parent chart.

### Adding your credentials
Pulling the various docker images from the Medifor docker registry requires the use of an imagePullSecret when working with Kubernetes. The easiest way to add your credentials would be to edit the `values.yaml` file to include them. When templates are rendered these values will be used in the `secret.yaml` file.

```yaml
registrycredentials:
  registry: https://gitlab-registry.mediforprogram.com
  username: <GITLAB_USERNAME>
  password: <REGISTRY_TOKEN>
  email: <MEDIFOR_EMAIL>
```
### Working with the templates
Every template defines a kubernetes resource file and every template that exists within the `templates/` directory will be created on `helm install`. When helm generates the templates it will pull the default values from `values.yaml` to populate the templates. Each template is provide the top level scope of the `values.yaml` file by default.

As an example we will look at `templates/ui/ui-service/yml`.

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: {{ .Values.ui.name }}
  name: {{ .Values.ui.name }}
  namespace: {{ .Values.namespace }}
spec:
  type: LoadBalancer
  ports:
    - name: http
      port: {{ .Values.ui.port }}
      protocol: TCP
      targetPort: {{ .Values.ui.port }}
  selector:
    app: {{ .Values.ui.name }}
```
Every field within double brackets`{{ }}` will be populated be the corresponding value in the `values.yaml` file.

For more information please see the offical Helm [template guide](https://helm.sh/docs/chart_template_guide/values_files/).

### Adding analytics/fusers to the system
Every analytic pod is a replica of an analytic deployment which includes two containers: a 'worker' and an 'analytic'.
The definition for these deployments can be found at `templates/analytics/analytic-deployment.yaml`. The deployment file is populated and replicated
for each analytic in the `analytics` field in the `values.yml` file.

```yaml
analytics:
  - id: ela-base
    name: Error Level Analysis
    description: Identifying areas within an image that are at different compression levels
    container: gitlab-registry.mediforprogram.com/medifor/program-registry/nextcentury_s_ela-base:2019-11-14T17-17-26f378000
    version: prod2020-03
    media: ["IMAGE", "VIDEO"]
    resources:
      requests:
        cpu: .5
        mem: 500Mi
        gpu: 0
      limits:
        cpu: 1
        mem: 1000Mi
        gpu: 0
```
To add a new analytic deployment to your cluster simple follow the above configuration with your desired analytic. **NOTE: The resource
requests and limits will vary depending on your analytic's requirements.**

The process for adding fusers is exactly the same with the definition for the fusion deployments found at `templates/fusers/fuser-deployment.yaml` and the 
values found in the `fusers` field of `values.yml`.

### Analytic Configuration File
A list of the analytics and fusers running on the system must be mounted into the `workflow` container which exists as a replica of the analyticworkflow deployment found at `templates/analyticworkflow/analyticworkflow-deployment.yaml`. 

The list is mounted through a configMap found at 
`templates/analyticworkflow/analyticlist-configmap.yml` which uses the `values.yaml` file as a single source of truth for analytics and fusers existing in the cluster. 

### Checking your templates
When modifying and saving your templates it is import to verify that they follow Helm/yaml convention.

To lint your chart
```bash
$ helm lint
```
To print rendered templates to the console
```bash
$ helm template templates/ .
```
<br/>
<br/>

## Deploying your chart
Helm makes it incredibly simply to deploy your kubernetes resources file.

To deploy your helm chart to your cluster (in this example we use medifor as the release name)
```bash
$ helm install medifor .
```

If all is succesful you should see the following 
```bash
NAME: medifor
LAST DEPLOYED: Wed Sep 30 12:23:36 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing medifor-helm-chart.

Your release is name medifor.

To learn more about the release, try:

    $ helm status medifor
    $ helm get all medifor
```

To check the status of the pod deployment
```bash
kubectl --namespace=medifor-mini get pods
```
If successful you should see something similar the following. *NOTE: If spinning up pods for the first time this can take a while.*
```bash
NAME                                 READY   STATUS              RESTARTS   AGE
analyticworkflow-868c679c6f-k8psb    0/1     ContainerCreating   0          21s
colorphenomenology-85499fb9-rsthx    0/2     ContainerCreating   0          22s
ela-base-7795fc44f9-dtzcb            0/2     ContainerCreating   0          21s
eqmedifor-0                          1/1     Running             0          21s
medifor-demo-ui-6f7fd4c8b6-sn2l9     0/1     ContainerCreating   0          21s
pgmedifor-0                          1/1     Running             0          21s
splicebuster-face-7846549b7d-fgvcr   0/2     ContainerCreating   0          21s
ta2-combo-7d888ccfb9-b8sbp           0/2     ContainerCreating   0          22s
umdgsrnet-769bdff5b8-6m9z7           0/2     ContainerCreating   0          21s
vol-init-zflgp                       0/1     Completed           0          21s
```
If the status of any of the pods it *imagePullBackoff* then there is an issue with your credentials

### Overriding default values
When templates are installed they will populate their fields with the contents of `values.yml`. However, these values can be overriden on the helm install commands.

For example, if you wanted to change the value of a specfic element in `values.yml` you can use the `--set` flag.
```bash
helm install --set namespace=new-namespace medifor .
```
This will change the `namespace` field in `values.yml` from *medifor-mini* to *new-namespace* in the `medifor` helm release.

You can also override values by providing a file to to helm install with the `-f` flag.
```bash
helm install -f newanalytics.yaml medifor .
```
Assuming that `newanalytics.yaml` contained a list of analytics that matched the format of the `analytics` field in `values.yaml` then those values would be overriden. This is an easy way to have seperate analytic configurations for seperate releases.

For more info on helm install see [here](https://helm.sh/docs/helm/helm_install/).

### Subcharts
This chart only contains a single subchart which is the autoscaler. The autoscaler allows the system to dynamically scale deployments up and down to allow a large collections of analytics to be used in resource constrained environments.

You can see that the autoscaler is a dependency in the `Chart.yml` file and that the resources definitions exist in the `charts/` subdirectory.
```yaml
dependencies:
  - name: autoscaler
    version: 0.1.0
    condition: autoscaler.enabled
```

For this default configuration the autoscaler is disabled which is noted the the `autoscaler.enabled` field in `values.yml`. Again this can be overriden either by editing `values.yml` or through the `--set` flag on `helm install`.

<br/>
<br/>

## Accessing the UI
The UI component offers a load balancer service that can be exposed to allow access to the cluster.
```bash
minkube service medifor-demo-ui -n medifor-mini
```
This exposes the medifor-demo-ui service on the medifor-mini namespace to your local machine by forwarding a port out of the minikube instance.