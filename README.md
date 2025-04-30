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

continuar
