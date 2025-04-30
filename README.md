# Monitoring K8s with the Kubernetes Dashboard, Prometheus, and Grafana

Monitoring in Kubernetes is essential for monitoring the health and behavior of your applications and infrastructure. Below, I'll explain how Prometheus has become the essential tool for achieving this goal and how you can implement it through these simple steps.


## Introduccion

## What is Prometheus?

It is an open-source monitoring tool designed for reliability and scalability in distributed environments.
To function properly, Prometheus relies on several components.
Prometheus Server: Collects and stores monitoring data.
Exporters: Expose metrics from systems and applications (example: Node Exporter for system metrics).


## goals
With this tutorial, you will be able to:
Deploy and instrument a sample Python Flask web-based API on Kubernetes, instrumented to provide metrics that Prometheus will collect and display in Grafana.

-Install and configure the Kubernetes dashboard using Helm.
-Install and configure Prometheus on Kubernetes using Helm.
-Configure Prometheus for service discovery.
-Install and configure Grafana on Kubernetes using Helm.
-Import a pre-built dashboard for real-time visualizations.


The api.py file contains the Python source code that implements the sample API.

Open a new terminal
Deploy the API application by running the following command:

```bash
kubectl apply -f ./code/k8s 
```

Confirm that the API pods are running. In the terminal, run the following command:

```bash
kubectl get pods 
```

Confirm that the service has been created

```sh
kubectl get svc
```

To generate traffic to the deployed API, start a single generator pod. In the terminal, run the following command:

```sh
kubectl run generator --env="API_URL=http://api-service:5000" -
image=cloudacademydevops/api-generator --image-pull-policy IfNotPresent
```

Confirm that the generator capsule is in working order.

```sh
kubectl get pods
```

So far, you've implemented a simple Python Flask web-based API, which has been instrumented to automatically collect and provide metrics for Prometheus to collect. You've also implemented a single generator pod that will continuously make HTTP requests to the API. In the next step, you'll install and configure Prometheus.

Create a new monitoring namespace within the cluster.

```sh
kubectl create ns monitoring
```

With Helm, install the Kubernetes dashboard using the publicly available Kubernetes Dashboard Helm chart. Deploy the dashboard to the monitoring namespace within the cluster.

```sh
{ 
helm repo add k8s-dashboard https://kubernetes.github.io/dashboard 
helm repo update 
helm install k8s-dashboard --namespace monitoring k8s-dashboard/kubernetes-dashboard -
set=protocolHttp=true --set=serviceAccount.create=true --set=serviceAccount.name=k8sdash
serviceaccount --version 3.0.2 
}
```

Set permissions within the cluster to allow the Kubernetes dashboard to read and write all cluster resources.

```sh
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --
serviceaccount=monitoring:k8sdash-serviceaccount 
```

The Kubernetes dashboard web interface must now be exposed to the internet so you can browse it. To do this, create a new NodePort-based service and expose the web management interface on port 30990.

```sh
{ 
kubectl expose deployment k8s-dashboard-kubernetes-dashboard --type=NodePort -
name=k8s-dashboard --port=30990 --target-port=9090 -n monitoring 
kubectl patch service k8s-dashboard -n monitoring -p '{"spec":{"ports":[{"nodePort": 30990, 
"port": 30990, "protocol": "TCP", "targetPort": 9090}]}}' 
}
```

Obtain the public IP address of the Kubernetes cluster where Prometheus is deployed.

```sh
export | grep K8S_CLUSTER_PUBLICIP 
```

Copy the public IP address from the command above, and then, using your local browser, navigate to the URL: http://PUBLIC_IP:30990.

foto


In this step, you installed the Kubernetes dashboard in the monitoring namespace within your Kubernetes cluster. You then configured and exposed the dashboard using a NodePort-based service. You then logged in to the dashboard and confirmed it was working. In the next step, you'll install and configure Prometheus to begin collecting metrics.

With Helm, install Prometheus using the publicly available Prometheus Helm chart. Deploy Prometheus in the monitoring namespace within the cluster.

```sh
{ 
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
helm repo add stable https://charts.helm.sh/stable 
helm repo update 
helm install prometheus --namespace monitoring --values ./code/prometheus/values.yml 
prometheus-community/prometheus --version 13.0.0 
}
```

Confirm that Prometheus has been successfully deployed to the cluster. 

```sh
kubectl get deployment -n monitoring -w
```

Confirm that the Prometheus Node Exporter DaemonSet resource has been created successfully. The Prometheus Node Exporter is used to collect memory and CPU metrics from each node in the Kubernetes cluster.

```sh
kubectl get daemonset -n monitoring 
```

Patch the Prometheus Node Exporter DaemonSet to ensure that Prometheus can collect CPU and memory node metrics.

```sh
kubectl patch daemonset prometheus-node-exporter -n monitoring -p 
'{"spec":{"template":{"metadata":{"annotations":{"prometheus.io/scrape": "true"}}}}}' 
```

The Prometheus web administration interface must now be exposed to the Internet so you can browse it. To do this, create a new NodePort-based service and expose the web administration interface on port 30900.

```sh
{ 
kubectl expose deployment prometheus-server --type=NodePort --name=prometheus-main -
port=30900 --target-port=9090 -n monitoring 
kubectl patch service prometheus-main -n monitoring -p '{"spec":{"ports":[{"nodePort": 
30900, "port": 30900, "protocol": "TCP", "targetPort": 9090}]}}' 
}
```

Obtain the public IP address of the Kubernetes cluster where Prometheus is deployed.

```sh
export | grep K8S_CLUSTER_PUBLICIP
```

## Copy the public IP address from the command above, and then, using your local browser, navigate to the URL: http://PUBLIC_IP:30900.


In this step, you installed Prometheus in the monitoring namespace within the Kubernetes cluster. Next, you configured and exposed the Prometheus web management interface using a NodePort-based service. You then logged in to the Prometheus web management interface and confirmed that service discovery was working correctly. In the next step, you'll install and configure Grafana and import a pre-built dashboard that pulls real-time data from Prometheus.

With Helm, install Grafana using the publicly available Grafana Helm chart. You'll deploy Grafana to the monitoring namespace within the cluster.

```sh
{ 
helm repo add grafana https://grafana.github.io/helm-charts 
helm repo update 
helm install grafana --namespace monitoring grafana/grafana --version 6.1.14 
} 
```

Confirm that the Grafana deployment has been successfully implemented.


```sh
kubectl get deployment grafana -n monitoring -w
```

The Grafana web management interface must now be exposed to the internet. To do this, create a new NodePort-based service, exposing the web management interface on port 30300.

```sh
{ 
kubectl expose deployment grafana --type=NodePort --name=grafana-main --port=30300 -
target-port=3000 -n monitoring 
kubectl patch service grafana-main -n monitoring -p '{"spec":{"ports":[{"nodePort": 30300, 
"port": 30300, "protocol": "TCP", "targetPort": 3000}]}}' 
}
```

Extract the default administrator password that will be required to log in.


```sh
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | 
base64 --decode ; echo
```

Get the public IP address of the Kubernetes cluster where Grafana is deployed.

```sh
export | grep K8S_CLUSTER_PUBLICIP
```
## Copy the public IP address from the command above, and then, using your local browser, navigate to the port http://PUBLIC_IP:30300.

In this step, you installed Grafana in the monitoring namespace within the Kubernetes cluster. Next, you configured and exposed the Grafana web management interface using a NodePort-based service. Next, you logged in to the Grafana web management interface and configured Prometheus as the data source. Next, you imported a pre-built dashboard. Grafana then loaded the dashboard and began pulling real-time monitoring data from the Prometheus data source.

# Follow me on my YouTube channel for more exercises.

https://www.youtube.com/@canaldedepplearningaprendi7105

