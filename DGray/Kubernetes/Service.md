```yaml
cat <<EOF | k -n payroll-tracing apply -f -
apiVersion: v1
kind: Service
metadata:
  name: cli-tools-service
  namespace: payroll-tracing
spec:
  ports:
    - name: metrics
      protocol: TCP
      port: 8888
      targetPort: 8888
  selector:
    app.kubernetes.io/name: cli-tools
EOF
```
```
```