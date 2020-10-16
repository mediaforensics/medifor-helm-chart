# MediFor Helm Chart

This repository provides a helm chart to deploy the MediFor system to a small Kubernetes cluster. 
For our purposes we will use minikube as an example.

Custom configuration may be needed depending on your specific cluster setup.

For development instructions see **[here](development.md).**

## Prerequisities 
- Kubernetes Clusters 1.10+
- Helm 3.0.0 +

### Generate Gitlab Token
1.  Navigate to [this page](https://gitlab.mediforprogram.com/profile/personal_access_tokens) to generate a new personal
access token to the MediFor gitlab.
2. Name the token `medifor-registry-token`, leave the expiration date blank, limit the
scop to `read_repository` and `read_registry` then create the personal access token.
3. Save the generated token somewhere safe.


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
Verify that all pods are running and containers have been created. If status is *ImagePullBackOff* then there is an error with credentials.
```bash
$ kubectl --namespace=medifor-mini get pods 
```
### Expose UI Service
Expose the UI Service on the minikube cluster to your local machine.
```bash
$ minikube service medifor-demo-ui -n medifor-mini
```

## Uninstallation
Uninstall Helm chart with release name `medifor`.
```bash
$ helm uninstall medifor
```

## Persistence
Persistent Volume Claims are used to maintain the data across deployments. This system uses the default storage classes provided with your cluster and is verified to work on GCE, AWS and minikube.

If running your cluster on microK8s you may need to **[enable the default storage class](https://igy.cx/posts/setup-microk8s-rbac-storage/).** <br/>

If you are running in minikube then your data will be hosted in `/tmp` and it is recommended that you mount a directory from your host machine into the minikube instance* 

## Troubleshooting
For more detailed instructions on working with this chart see **[here](development.md).**