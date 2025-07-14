```yaml
image: {{ $image := print "artifactory.raiffeisen.ru/ext-rbru-techimage-container-docker/prometheus/prometheus:" .Values.prometheus.image_tag }}{{ $image }}
```
