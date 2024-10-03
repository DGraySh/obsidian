cat <<EOF | k -n payroll-tracing apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: cli-tools
spec:
  containers:
  - name: cli-tools
    image: artifactory.raiffeisen.ru/ext-devops-community-docker/cli-tools:latest
    command:
      - sleep
      - "3600"
  imagePullSecrets:
    - name: docker-registry
EOF