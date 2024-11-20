```
kubectl wait --for='jsonpath={.status.conditions[?(@.type=="Ready")].status}=True' externalsecrets.external-secrets.io -n tracing docker-registry-es --timeout=180s
```