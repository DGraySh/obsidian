```yaml
cat <<EOF | k -n payroll-tracing apply -f -
apiVersion: v1
kind: Service
metadata:
  name: my_service
  namespace: payroll-tracing
spec:
  ports:
    - name: metrics
      protocol: TCP
      port: 8888
      targetPort: 8888
EOF
```
```
```