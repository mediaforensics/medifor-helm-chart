# Working with the chart

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

- **Chart.yaml** - Contains a description of the chart and its dependencies (which are known as _subcharts_ ).
- **values.yaml** - Contains the default values for the chart that are used in the template files.
- **templates/** - Directory for all template files. When helm evaluates a chart it will send all of the files in this directory
  through a template rendering engine. It then collects the results of those templates and sends them to Kubernetes.
- **charts/** - Directory that contains other charts, known as _subcharts_, which this chart depends on. Subcharts must be able to exist independently
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
  type: {{ .Values.ui.service.type }}
  ports:
    - name: http
      protocol: TCP
      port: {{ .Values.ui.port }}
      targetPort: {{ .Values.ui.port }}
      {{- if eq .Values.ui.service.type "NodePort" }}
      nodePort: {{ .Values.ui.service.nodePort }}
      {{- end }}
  selector:
    app: {{ .Values.ui.name }}
```

Every field within double brackets `{{ }}` will be populated be the corresponding value from the `values.yaml` file.

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
        cpu: .125
        mem: 100Mi
        gpu: 0
      limits:
        cpu: .25
        mem: 200Mi
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

When modifying and saving your templates it is important to verify that they follow Helm/yaml convention.

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

Helm makes it incredibly simply to deploy your kubernetes resource files.

To deploy your helm chart to your cluster (in this example we use _medifor_ as the release name)

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
kubectl --namespace=medifor get pods
```

If successful you should see something similar to the following. _NOTE: If spinning up pods for the first time this can take a while._

```bash
NAME                               READY   STATUS      RESTARTS   AGE
analyticworkfow-76f75cdf65-7jjwk   1/1     Running     1          3m27s
ela-base-9d5c45446-bvmcd           2/2     Running     0          3m27s
eqmedifor-0                        1/1     Running     0          3m27s
exif-5589d65f85-kn9b9              2/2     Running     0          3m27s
medifor-ui-54c55b8649-kkksk        1/1     Running     0          3m27s
pgmedifor-0                        1/1     Running     0          3m27s
ta2avg-57ddd4b596-75mjn            2/2     Running     0          3m27s
vol-init-dsbsc                     0/1     Completed   0          3m27s
```

If the status of any of the pods is _ImagePullBackoff_ then there is an issue with your credentials.

### Overriding default values

When templates are installed they will populate their fields with the contents of `values.yml`. However, these values can be overriden on the helm install commands.

For example, if you wanted to change the value of a specfic element in `values.yml` you can use the `--set` flag.

```bash
helm install --set namespace=new-namespace medifor .
```

This will change the `namespace` field in `values.yml` from _medifor_ to _new-namespace_ in the `medifor` helm release.

You can also override values by providing a file to to helm install with the `-f` flag.

```bash
helm install -f newanalytics.yaml medifor .
```

Assuming that `newanalytics.yaml` contained a list of analytics that matched the format of the `analytics` field in `values.yaml` then those values would be overriden. This is an easy way to have seperate analytic configurations for seperate releases.

For more info on helm install see [here](https://helm.sh/docs/helm/helm_install/).

## Persistence

Persistent Volume Claims are used to maintain the data across deployments. This system uses the default storage classes provided with your cluster and is verified to work on GCE, AWS and minikube. If you would like to provide your own storage then please populate the storageClass values in `values.yml` and create your own storage class resources files.

```yaml
persistence:
  medifor:
    # storageClass:
    accessMode: ReadWriteMany
    size: 2Gi
  postgres:
    # storageClass:
    accessMode: ReadWriteOnce
    size: 5Gi
```

Dynamically allocated Persisent Volumes default to a reclaim policy of 'Delete', to avoid data loss it is recommended to change this reclaim policy to 'Retain'.

```bash
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                           STORAGECLASS   REASON   AGE
pvc-0541deb9-fe88-4ced-aa18-d840bc2caf8a   5Gi        RWO            Delete           Bound    medifor/postgres-pvc       standard                2m19s
pvc-ba4ebe21-3be1-4adc-afe4-27cb75080c2a   2Gi        RWX            Delete           Bound    medifor/medifor-data-pvc   standard                2m19s

$ kubectl patch pv <pvc-0541deb9-fe88-4ced-aa18-d840bc2caf8a> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
$ kubectl patch pv <pvc-ba4ebe21-3be1-4adc-afe4-27cb75080c2a> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

Please see the official K8s docs on [persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) for more information.

<br/>
<br/>
