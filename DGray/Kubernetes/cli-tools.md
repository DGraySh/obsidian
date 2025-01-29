```yaml

cat <<EOF | k -n sandbox apply -f -
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
    volumeMounts:
      - mountPath: "/tls"
        name: tls
    resources: 
      limits: 
        cpu: 100m
        memory: 100Mi
      requests: 
        cpu: 100m
        memory: 100Mi
  volumes:
    - name: tls
      csi:
        driver: csi.cert-manager.io
        readOnly: true
        volumeAttributes:
          csi.cert-manager.io/issuer-name: vaultpki-clusterissuer
          csi.cert-manager.io/issuer-kind: ClusterIssuer
          csi.cert-manager.io/dns-names: cli-tools
          csi.cert-manager.io/fs-group: 2001
  imagePullSecrets:
    - name: docker-registry
EOF
 ```