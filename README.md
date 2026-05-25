This contains all the config for the K8S cluster to host my application to learn portuguese. It use K3S on a single VM to reduce cost and a minimal monitoring stack. My app should work on any K8S but this is helpful for me to have a minimalist K8S stack. It run on a Ubuntu VM on OVH (cheap)

# OS

Use ansible to install K3S and helm:

```
never tested: cd cm-ansible
ansible-playbook -i inventory.yml site.yml --ask-become-pass
```

i never tested it so some notes on how it was done manualy:

Micro K3s
https://docs.k3s.io/quick-start
not sure but probably
curl -sfL https://get.k3s.io | sh -

note: we use local-path rancher plugin for local storage for light solution everywhere instead of volumes

helm
https://helm.sh/docs/



==
# K8S 
I use K3S on single node for costs reasons but everything should work on K8S

use rancher local-path for storage as cleaner than raw hostpath

## certmanager
i did not keep traces on what i did. TODO
## coredns
i did not keep traces on what i did. TODO

## Log
I m pretty confident on this part as i did rely on helm/value and no manual actions. It s a light solution based on Victoria/Grafana

Everything was install with helm. value files are in the repo.
TODO: argoCD app or flux (more light)

### VictoriaLog
VictoriaLogs for log storage

```
helm repo add victoriametrics https://victoriametrics.github.io/helm-charts
helm repo update

helm upgrade --install victoria-logs vm/victoria-logs-single   --namespace logging   --values Victlogs.yaml   --wait
```

### FluentBit 
Fluent Bit for log collector

```
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update

helm upgrade --install fluent-bit fluent/fluent-bit \
  --namespace logging \
  --values fluentbit-values.yaml \
  --wait
```

### Grafana

FI: install community one not the old deprecated one.
```
helm repo add grafana-community https://grafana-community.github.io/helm-charts
helm repo update

helm upgrade --install grafana grafana-community/grafana   --namespace logging   --values grafana-values.yaml   --wait
```

I had some issue with SSL:

note: can not use wildcard cert as letsencrypt only allow DNS challenge for them so 2 certs: 1 for app and 1 for grafana

coredns-override.yaml is to make grafana subdomain know in coredns for SSL cert
sudo kubectl apply -f coredns-override.yaml
sudo kubectl rollout restart deployment coredns -n kube-system

check with:
ubuntu@vps-68586aba:~/PortugueseLearning$ sudo kubectl run dns-test --image=busybox --rm -i --restart=Never -n logging --   sh -c "nslookup grafana.dialecthub.net && echo '=== SUCCESS ===' || echo '=== FAILED ==='"
Server:         10.43.0.10
Address:        10.43.0.10:53


Name:   grafana.dialecthub.net
Address: 149.56.132.188

=== SUCCESS ===


### Nodeexporter
get VM metrics
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install node-exporter prometheus-community/prometheus-node-exporter \
  --namespace logging 
```
### VictoriaMetrics
node exporter link to victoriametrics (prom too big for small cluster):

```
helm repo add victoria-metrics https://victoriametrics.github.io/helm-charts/
helm repo update

helm install vm victoria-metrics/victoria-metrics-single \
  --namespace logging \
  -f values-metrics.yaml
```# PortugueseLearningDeploy
