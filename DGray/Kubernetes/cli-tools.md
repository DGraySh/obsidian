
```yaml

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

```yaml

cat <<EOF | k -n monitoring-team-managed apply -f -
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
      - name: ca  
        mountPath: /etc/ssl/certs
    resources: 
      limits: 
        cpu: 100m
        memory: 100Mi
      requests: 
        cpu: 100m
        memory: 100Mi
  volumes:
    - name: ca  
      secret:  
        secretName: bank-ca
    - name: tls
      csi:
        driver: csi.cert-manager.io
        readOnly: true
        volumeAttributes:
          csi.cert-manager.io/issuer-name: vaultpki-clusterissuer
          csi.cert-manager.io/issuer-kind: ClusterIssuer
          csi.cert-manager.io/dns-names: cli-tools
          csi.cert-manager.io/fs-group: "2001"
  imagePullSecrets:
    - name: docker-registry
EOF
 ```
grpcurl thanos-storegateway.monitoring-team-managed.svc.prollact-v-dmz5v-msk34-c05.kaas.raiffeisen.ru:10901 list

```
cat <<EOF | k -n tracing apply -f -
apiVersion: v1
data:
  ca.pem: |- 
-----BEGIN CERTIFICATE-----  
MIIFITCCAwmgAwIBAgIQFp+mZdK8Yo9HEAvscgIgmDANBgkqhkiG9w0BAQsFADAj  
MSEwHwYDVQQDExhBTyBSYWlmZmVpc2VuYmFuayBSb290Q0EwHhcNMTYxMjIyMDgx  
NzUzWhcNMzYxMjIyMDgyNzUxWjAjMSEwHwYDVQQDExhBTyBSYWlmZmVpc2VuYmFu  
ayBSb290Q0EwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQC8vzJWfsM9  
7yL9dRlx0SLdWm67/BPu5/YOBgKfwgj4RRFsG6/M+Eq9LB4p1kTvAayBEHJtnOiH  
TSNWU07ztuBSdPvPABT+KGPpcGfPWDN3z046LekjCq2QexjA5wPBvQNgVxVO1XMt  
ikSnU2nZZWSVN1zvagHzLk2RxknadVTssVxMGSVW63XTle3XFef/tpYjUT7MSEeL  
Umq5hnOoNH3D85sG1s3JYqR1nyMtoe50j7urT/Pif/Yv/nG/pcZayRjuZRRLVP0w  
4SCXK8om70vAFElkOnGcZq36Y5h7iLLhodRrMN6MFXefOyKf4UTTuONdXJCLzQ35  
Ssq1JdgxcCugzkbMMS3IZV7h6mPhZfC7wZSxu6hVa36jldmXBiXXk1U6Kgx5axs4  
9ckCr/qqNezPEFqXW8Xqe7UJ59hw2bSrg6zrNqEfkPhT2NWiVmmLzA0CaYngRHFN  
daekEpbtjJQvU2x/p5+7AEoCakRR/Z2BxzfaE0JVonIAK3Pd+Cz2xZmEWSP1aTdd  
nUlAGJ0MsEI5I2hLAeCAUe9QampkpW+LY8x7mlLZDNAvzqgrwAqFT/2coMusX29j  
3Cn/YBocKg7A665EcAyUulwNWn7hc1s7pSt290XCs7nipX01dlWAwXL3iqjzfZDL  
ipYRJynP0f71m8x1Xm1NFjyWvPDfraJYeQIDAQABo1EwTzALBgNVHQ8EBAMCAYYw  
DwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUoEteDogZfe79/KB0Qn0eSqkIS64w  
EAYJKwYBBAGCNxUBBAMCAQAwDQYJKoZIhvcNAQELBQADggIBABe7iRnfCt+s/MCb  
ABf42B4B5bAfT+mJA2gObv3UmfI/rhfs0c7bATYcK8NeAAUodfJwgaXvQGBQi6Gg  
1excdNRFYy4NgAfFhyePFpxocAtcaiBc4qg/atBm6SeidnN8K9HVZZ2aChvZlOt5  
5WqEZQTx2rsus7fkLs6WDvA2H051amx5FWZbGDNKeUlcrepe1ulI4XxZQcBOs2jd  
guho2rekWhd1jPyAs+QjVAh0H5K2NydAIuXHMPDhEvXYyjDt8UoLZaUmZ1By3A5l  
FMOfKCXd7gLZ3bkueVGBDaGUuBtRE12MlsJG2cTrf/KVy2DiDhNWLmLUfIjyuzbN  
9T6i6mHMh8AJEvT2kLMfnlhm9RUgR6SzSwLAa60iaKo1BA9hdAGQFzIWmTxvsae8  
RdYSv9Yui6h6G/SQU7YMGmVygrL0KG+a7oi59f2mhDszaDvgUK5fEzfKz6lCOYFL  
AGXECmfKEijw98HcEHGxCm3Xv1B1DANryXJ/LXBESXXeLxousYhibd7hyA1+jF2R  
e8vGZLr5d+n009SEgZoQpgvfIMapjJxEXQfrfH5QtPylwhLCs6K2EGqkpB05tV5n  
i6kqbjtYjZwViqe5t/whEEqiKGpVA0wYGj4JkUSgyM1Lyo0b1M9sjoYnTZHA7A5G  
r4rgIobAVejcm872a5S/tIQX31ZZ  
-----END CERTIFICATE-----
kind: Secret
metadata:
  name: bank-ca
  namespace: tracing
type: Opaque
EOF
```