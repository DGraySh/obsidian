```yaml

cat <<EOF | k -n sandbox apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: docker-registry
  namespace: sandbox
data:
  .dockerconfigjson: >-
eyJhdXRocyI6eyJhcnRpZmFjdG9yeS5yYWlmZmVpc2VuLnJ1Ijp7InVzZXJuYW1lIjoic3J2X3RfcHJsX2Fuc2libGUiLCJwYXNzd29yZCI6IkE2Mi9EQT1Ia2pUUkJhU29nV1BRNkw9ZHJaTzNZNSJ9fX0=
type: kubernetes.io/dockerconfigjson
EOF

```