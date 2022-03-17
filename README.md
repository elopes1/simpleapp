## Procedimento para subir o ambiente
## Instalação do minikube (Existe um bug nas versões mais recentes do minikube que tive problemas https://github.com/kubernetes/kubernetes/issues/107297 por isso estou usando uma versão antiga)
git clone https://github.com/kubernetes/minikube.git && cd minikube
git checkout v1.18.0
make && sudo cp out/minikube /usr/local/bin/

## Aumentei a qunatidade de cpus para subir as aplicações mais rápido (default 2 cpus)
minikube start --vm-driver=virtualbox --cpus=6

## Ajuste para subir a imagem local no k8s
eval $(minikube docker-env)

## Build do simpleapp
docker build -t simpleapp-python:1 .

## Apply dos arquivos yaml para subir
for file in $(ls *.yaml); do kubectl apply -f $file ; done

## Url para o simpleapp
minikube service --url simpleapp

## Adicionar repositório de charts do grafana
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

## Instalar a stack do loki-grafana (grafana, prometheus, alertmanager e loki)
helm upgrade --install grafana grafana/loki-stack  --set grafana.enabled=true,prometheus.enabled=true,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false

## Alterar o tipo do service para loadbalancer
kubectl patch services grafana -p '{"spec": {"type": "LoadBalancer"}}'

## URL para grafana
minikube service --url grafana

## Senha do admin do grafana
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo


## Acessar a url do grafana e importar o dashboard usando o json (simpleapp-dashboard.json)

