image:
  repository: "artifactory.raiffeisen.ru/remote-quay-docker/oauth2-proxy/oauth2-proxy"
  tag: "v7.6.0"
  pullPolicy: "IfNotPresent"
imagePullSecrets:
  - name: docker-registry

config:
  configFile: |-
    client_id = "jaeger-query"
    cookie_secret = "cynlO4FS4zwGAfYkr3DBzvTV0t1pz9WIBv64Ln2lnbU=" //dd if=/dev/urandom bs=32 count=1 2>/dev/null | base64 | tr -d -- '\n' | tr -- '+/' '-_' ; echo
    client_secret_file = "/etc/creds/client-secret-key"
    provider = "keycloak-oidc"
    http_address = "http://:4180"
    upstreams = "file:///dev/null"
    redirect_url = "https://oauth2-proxy.payroll-tracing.compute-managed-v-dmz5v-msk34-c02.compute.raiffeisen.ru/oauth2/callback"
    oidc_issuer_url = "https://keycloak-preview.raiffeisen.ru/realms/PayPGh"
    cookie_secure = "false"
    email_domains = "raiffeisen.ru"
    cookie_domains = ".raiffeisen.ru"
    whitelist_domains = ".raiffeisen.ru"
    skip_provider_button = "true"
    ssl_insecure_skip_verify = "true"
    code_challenge_method = "S256"
    insecure_oidc_allow_unverified_email = "true"
    scope = "openid email profile"
    silence_ping_logging = "true"
proxyVarsAsSecrets: false
service:
  type: ClusterIP
  portNumber: 80
  appProtocol: http

serviceAccount:
  enabled: true

ingress:
  enabled: true
  className: nginx
  path: /
  pathType: ImplementationSpecific
  hosts:
    - oauth2-proxy.payroll-tracing.compute-managed-v-dmz5v-msk34-c02.compute.raiffeisen.ru
  annotations:
    cert-manager.io/cluster-issuer: vaultpki-clusterissuer
    nginx.ingress.kubernetes.io/backend-protocol: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: 'true'
  tls:
    - secretName: oauth2-proxy-tls
      hosts:
        - oauth2-proxy.payroll-tracing.compute-managed-v-dmz5v-msk34-c02.compute.raiffeisen.ru

resources: 
  limits:
    cpu: 100m
    memory: 300Mi
  requests:
    cpu: 100m
    memory: 300Mi

extraVolumes: 
  - name: client-secret-key
    secret:
      secretName: keycloak-client-secret

extraVolumeMounts:
  - name: client-secret-key
    mountPath: /etc/creds

livenessProbe:
  enabled: true
  initialDelaySeconds: 5
  timeoutSeconds: 1

readinessProbe:
  enabled: true
  initialDelaySeconds: 5
  timeoutSeconds: 5
  periodSeconds: 10
  successThreshold: 1

securityContext:
  enabled: false

metrics:
  enabled: true
  port: 44180
  service:
    appProtocol: http
  serviceMonitor:
    enabled: true

