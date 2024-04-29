# Kubernetes Usage

From the `./kube` folder

Deploy your config :

```
kubectl apply -k .
```

Deploy your ressources:

```
kubectl apply -f .
```

If you want to change the autoscaling params :

```
kubectl autoscale deployment vote-hpa --cpu-percent=70 --min=2 --max=10
```

The job should complete in around 10 seconds.

You can connect trough vote and result services

```
kubectl get svc

vote => htpp://<vote-public-ip>:8000
result => htpp://<result-public-ip>:3000
```
