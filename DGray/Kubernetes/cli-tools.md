```yaml

cat <<EOF | k -n gtlab-agent apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: cli-tools
spec:
  containers:
  - name: cli-tools
    ports:
    - name: metrics
      containerPort: 8888
      protocol: TCP
    image: artifactory.raiffeisen.ru/ext-devops-community-docker/cli-tools:latest
    resources: 
      limits: 
        cpu: 100m
        memory: 100Mi
      requests: 
        cpu: 100m
        memory: 100Mi
  imagePullSecrets:
    - name: docker-registry
EOF
 ```