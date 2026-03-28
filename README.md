# Kubernetes Horizontal Scaling Demo (Minikube)

This project demonstrates horizontal scaling with Kubernetes using a `Deployment`, a `Service`, and an `HPA` (Horizontal Pod Autoscaler).

The demo runs an example backend app and scales pods based on CPU utilization.

## Project structure

- `k8s/deployment.yaml`
- `k8s/service.yaml`
- `k8s/hpa.yaml`

## What this setup uses

- `Deployment`: `scaling-app`
- `Service`: `scaling-app-service`
- `HPA`: `scaling-app-hpa`
- CPU target in HPA: `50%`
- Min replicas: `1`
- Max replicas: `2` (you can increase this in `k8s/hpa.yaml`)

## Prerequisites

Make sure you have:

1. [Minikube](https://minikube.sigs.k8s.io/docs/start/)
2. `kubectl`

## 1. Start Minikube

```bash
minikube start
```

## 2. Enable metrics-server (required for HPA)

```bash
minikube addons enable metrics-server
```

Optional check:

```bash
kubectl get pods -n kube-system | grep metrics-server
```

## 3. Deploy the application resources

From the project root:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/hpa.yaml
```

## 4. Verify resources

```bash
kubectl get deploy
kubectl get svc
kubectl get hpa
kubectl get pods
```

To watch scaling in real time:

```bash
kubectl get hpa -w
```

## 5. Stress the application (load generator)

Run the following command to generate load and trigger autoscaling:

```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://scaling-app-service; done"
```

Keep this running for a short period, and in another terminal monitor:

```bash
kubectl get hpa -w
kubectl get pods -w
```

You should see replicas increase (up to `maxReplicas`) when CPU utilization crosses the target.

## 6. Stop the load and observe scale down

After stopping the load generator (`Ctrl + C`), Kubernetes should eventually scale pods back down toward `minReplicas`.

## Cleanup

Delete all created resources:

```bash
kubectl delete -f k8s/hpa.yaml
kubectl delete -f k8s/service.yaml
kubectl delete -f k8s/deployment.yaml
```

Or delete the whole Minikube cluster:

```bash
minikube delete
```
