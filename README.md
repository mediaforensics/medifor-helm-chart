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
Pulling the various docker images from the Medifor docker registry requires the use of an imagePullSecret when working with Kubernetes. The easiest way to add
your credentials would be to edit the `values.yaml` file to include them. When templates are rendered these values will be used in the `secret.yaml` file.

```yaml
registrycredentials:
  registry: https://gitlab-registry.mediforprogram.com
  username: <GITLAB_USERNAME>
  password: <REGISTRY_TOKEN>
  email: <MEDIFOR_EMAIL>
```
### Working with the templates
Every template defines a kubernetes resource file and every template that exists within the `templates/` directory will be created on `helm install`. When helm generates
the templates it will pull the default values from `values.yaml` to populate the templates. Each template is provide the top level scope of the `values.yaml` file by default.

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
The definition for these deployments can be found at `templates/analytics/analytic-deployment.yaml`. This deployment file is populated and replicated
depending on the `analytics` filed in the `values.yml` file.

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
To add a new analytic deployment to your cluster simple follow the above configuration with your desired analytic. **Note that the resource
requests and limits will vary depending on your analytic's requirements.**

The process for adding fusers is exactly the same with the definition for the fusion deployments found at `templates/fusers/fuser-deployment.yaml` and the 
values found in the `fusers` field of `values.yml`.



