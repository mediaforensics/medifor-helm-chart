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
- **Chart.yaml** - Contains a description of the chart and of of its dependencies (which are known as *subcharts* ).
- **values.yaml** - Contains the default values for the chart that are used in the template files.
- **templates/** - Directory for all template files. When helm evaluates a chart it will send all of the files in this directory
                     through a template rendering engine. It then collects the results of those templates and sends them to Kubernetes.
- **charts/** - Directory that contains other charts, known as *subcharts*, which this chart depends on. Subcharts must be able to exist independently
                  from their parent chart.