1. create a Kubernetes Secret
You can create it using kubectl:

```bash
kubectl create secret generic datadog-api-key \
  --from-literal=key=c3adad65ab27d7f0e937923d585be0ef \
  --namespace=application
```

`config.yaml`
```yaml
...
api:
  site: "datadoghq.com"
  key: ${DATADOG_API_KEY}  # <-- reference env var here
...
```

`deployment.yaml`
```yaml
...
env:
  - name: DATADOG_API_KEY
    valueFrom:
      secretKeyRef:
        name: datadog-api-key
        key: key
...
```


