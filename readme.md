Installation:

Configure minikube:
minikube config view
minikube config set memory 8096
minikube config set cpus 4
minikube config set driver docker
minikube start
minikube addons enable ingress

1) Enter dir "/mnt/ab9b68ab-426f-42d6-ab39-09fcae0b8a0a/files/projects/scdf"

2) Create namespace
   kubectl apply -f kubernetes/namespace/namespaces-config.yaml

3) Configure namespace
   kubectl config set-context --current --namespace=scdf-dev
   kubectl config view --minify | grep namespace:

4) Create zookeeper/kafka
   kubectl apply -f kubernetes/broker/zookeeper-config.yaml
   kubectl apply -f kubernetes/broker/kafka-config.yaml

5) Create postgres
   kubectl apply -f kubernetes/database/mysql-config.yaml

6) Create prometheus/grafana
   kubectl apply -f kubernetes/observability/prometheus-config.yaml
   kubectl apply -f kubernetes/observability/grafana-config.yaml

7) Create SCDF orchestrator
   kubectl apply -f kubernetes/orchestrator/scdf-account-config.yaml
   kubectl apply -f kubernetes/orchestrator/scdf-skipper-config.yaml
   kubectl apply -f kubernetes/orchestrator/scdf-server-config.yaml

8) Resolve hosts
   192.168.49.2     dashboard.server.scdf.local
   192.168.49.2     dashboard.grafana.scdf.local


Rebuild and redeploy service:
eval $(minikube docker-env) && ./gradlew clean jibDockerBuild && kl rollout restart -n drone-transport deployment metrics-adapter