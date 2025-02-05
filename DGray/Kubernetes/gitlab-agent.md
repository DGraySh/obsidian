
```bash
kubectl create secret docker-registry "docker-registry" --docker-server="https://artifactory.raiffeisen.ru/" --docker-username="srv_t_prl_ansible" --docker-password="A62/DA=HkjTRBaSogWPQ6L=drZO3Y5" --namespace="gitlab-agent"
```

```bash
noglob helm upgrade --install gitlab-agent agent-helm/gitlab-agent --namespace="gitlab-agent" --set serviceAccount.imagePullSecrets[0].name="docker-registry" --set config.token="glagent-7QzS5Asf7yv1o6GnKD_Ty4D5gJK_2X5jQ7cCSyc7gsde5iv2iw" --set image.repository="artifactory.raiffeisen.ru/ext-devops-community-docker/agentk" --set image.tag="16.11.5"

```

