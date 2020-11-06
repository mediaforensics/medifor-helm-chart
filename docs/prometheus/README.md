# Prometheus Monitoring Setup
This document will guide you through the steps to:
- Set up Prometheus on your cluster
- Set up Grafana on your cluster
- Create metrics endpoints for the analytic containers

These instructions assumes you are using Minikube for your cluster.

## Prerequisites
- Kubernetes Cluster 1.10+
- Helm 3.0.0+

<br />

## Install Prometheus and Grafana
### Add prometheus-community Helm chart
```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
```
### Install Chart
Install the `kube-prometheus-stack` chart with a release name of `prometheus` in the default kubernetes namespace. <br/>
```bash
$ helm install prometheus prometheus-community/kube-prometheus-stack
```
For more information, visit the [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) repository.

### Update Your Medifor Release
Now that Prometheus is running on the cluster you can update your `medifor` helm release to create the analytic services and service monitors which expose the metrics endpoints to Prometheus.
```bash
$ helm upgrade --set prometheus.enabled=true medifor <medifor-helm-chart path>
```
See the *[helm upgrade](https://helm.sh/docs/helm/helm_upgrade/)* docs for more information.

### Verify Metrics Services 
You'll want to check that the services for the analytic metrics were created.
```bash
$ kubectl --namespace=<medifor-namespace> get services

NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
analyticworkflow   ClusterIP   10.109.13.158    <none>        50051/TCP            11m
ela-base-metrics   ClusterIP   10.102.93.150    <none>        2112/TCP             5m58s
eqmedifor          ClusterIP   10.100.142.199   <none>        37706/TCP,8080/TCP   10s
exif-metrics       ClusterIP   10.97.49.240     <none>        2112/TCP             5m58s
medifor-ui         NodePort    10.100.251.0     <none>        3000:30000/TCP       11m
pgmedifor          ClusterIP   None             <none>        5432/TCP             11m
```
All analytics in the system should have a service with a name `analyticname-metrics`, they will also have service monitors of the same name `-monitor` appended. These services are what communicate with the prometheus service monitors.

```bash
$ kubectl get servicemonitoras

NAME                                                 AGE
ela-base-metrics-monitor                             12s
eqmedifor-metrics-monitor                            12s
exif-metrics-monitor                                 12s
prometheus-kube-prometheus-alertmanager              98s
prometheus-kube-prometheus-apiserver                 98s
prometheus-kube-prometheus-coredns                   98s
prometheus-kube-prometheus-grafana                   98s
prometheus-kube-prometheus-kube-controller-manager   98s
prometheus-kube-prometheus-kube-etcd                 98s
prometheus-kube-prometheus-kube-proxy                98s
prometheus-kube-prometheus-kube-scheduler            98s
prometheus-kube-prometheus-kube-state-metrics        98s
prometheus-kube-prometheus-kubelet                   98s
prometheus-kube-prometheus-node-exporter             98s
prometheus-kube-prometheus-operator                  98s
prometheus-kube-prometheus-prometheus                98s
```

<br />

## Expose Services
Since this example is using Minikube you can port forward Prometheus and Grafana out of the minikube cluster to your localhost. <br/>
If using an alternate cluster setup you will need to modify how you expose these services. <br />
See *[http://kubernetes.io/docs/user-guide/services/](http://kubernetes.io/docs/user-guide/services/)* for more information.

### Prometheus
Expose Prometheus service
```bash
$ kubectl port-forward service/prometheus-kube-prometheus-prometheus 9090:9090
```
##### In the `Targets` view you should be able to see metrics endpoints for every running analytic along with the default cluster metrics.
<br />

![](assets/metrics.png)

##### These metrics endpoint will also provide you with three additional queries found below.
<br />

![](assets/queries.png)

### Grafana
Expose Grafana service
```bash
$ kubectl port-forward service/promtheus-grafana 80:80
```
You will be prompted for a username and password which are defaulted to:
- Username: admin
- Password: prom-operator

Grafana ships with a variety of dashboards and monitoring tools. Please visit their *[user guide](https://grafana.com/docs/grafana/latest/dashboards/)* for more information.