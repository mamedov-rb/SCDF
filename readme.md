### Installation:

- Configure minikube:
  minikube config set memory 8096
  minikube config set cpus 4
  minikube config set driver docker
  minikube config view
  minikube start
  minikube addons enable ingress

- Resolve hosts
  192.168.49.2 dashboard.kubernetes.local
  192.168.49.2 dashboard.grafana.scdf.local
  192.168.49.2 dashboard.grafana.scdf.local

## Helm installation
- Install Helm:
  curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
  sudo apt-get install apt-transport-https --yes
  echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee
  /etc/apt/sources.list.d/helm-stable-debian.list
  sudo apt-get update
  sudo apt-get install helm

- Install Prometheus:
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  helm repo add stable https://charts.helm.sh/stable
  helm repo update
  helm install prometheus prometheus-community/kube-prometheus-stack

- Create db
  kubectl apply -f kubernetes/database/postgres-config.yaml

- helm install scdf \
  --set server.ingress.enabled=true \
  --set server.ingress.hostname=dashboard.server.scdf.local \
  --set rabbitmq.enabled=false \
  --set kafka.enabled=true \
  --set mariadb.enabled=false \
  --set externalDatabase.dataflow.username=scdf_admin \
  --set externalDatabase.skipper.username=scdf_admin \
  --set externalDatabase.dataflow.url=jdbc:postgresql://postgres:5432/scdf_db \
  --set externalDatabase.skipper.url=jdbc:postgresql://postgres:5432/scdf_db \
  --set externalDatabase.password=B2U1vS8ry^h8wNb@M@NB \
  --set externalDatabase.hibernateDialect=org.hibernate.dialect.PostgreSQLDialect \
  bitnami/spring-cloud-dataflow

## Kubectl installation
- Enter dir "/mnt/ab9b68ab-426f-42d6-ab39-09fcae0b8a0a/files/projects/scdf"

- Create namespace/ingress
  kubectl apply -f kubernetes/namespace/namespaces-config.yaml
  minikube dashboard #initilized kubernetes-dashboard ns
  kubectl apply -f kubernetes/ingress-config.yaml

- Configure namespace
  kubectl config set-context --current --namespace=scdf-dev
  kubectl config view --minify | grep namespace:

- Create zookeeper/kafka
  kubectl apply -f kubernetes/broker/zookeeper-config.yaml
  kubectl apply -f kubernetes/broker/kafka-config.yaml

- Create db
  kubectl apply -f kubernetes/database/mysql-config.yaml
  kubectl apply -f kubernetes/database/postgres-config.yaml

- Create prometheus/grafana
  kubectl apply -f kubernetes/observability/prometheus-config.yaml
  kubectl apply -f kubernetes/observability/grafana-config.yaml

- Create SCDF orchestrator
  kubectl apply -f kubernetes/orchestrator/scdf-account-config.yaml
  kubectl apply -f kubernetes/orchestrator/scdf-skipper-config.yaml
  kubectl apply -f kubernetes/orchestrator/scdf-server-config.yaml

- Rebuild image and push to minikube:
  cd "/mnt/ab9b68ab-426f-42d6-ab39-09fcae0b8a0a/files/projects/{service-name}"
  eval $(minikube docker-env) && ./gradlew clean jibDockerBuild

### Deletion
minikube delete --all
docker volume rm minikube