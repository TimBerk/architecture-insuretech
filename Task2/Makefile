image:
	docker pull ghcr.io/yandex-practicum/scaletestapp
start:
	minikube start
	minikube status
	minikube addons enable metrics-server
man_1:
	kubectl apply -f ./manifests/namespace.yaml
	kubectl apply -f ./manifests/deployment.yaml
	kubectl apply -f ./manifests/service.yaml
	kubectl apply -f ./manifests/hpa.yaml
man_2:
	kubectl apply -f ./manifests/namespace.yaml
	kubectl apply -f ./manifests/deployment.yaml
	kubectl apply -f ./manifests/service.yaml
	kubectl apply -f ./manifests/service-monitor.yaml

	helm install prometheus-adapter prometheus-community/prometheus-adapter -n monitoring -f ./manifests/prometheus-adapter.yaml
	kubectl apply -f ./manifests/hpa-rps.yaml
prom:
	helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace \
	--set prometheus.service.type=NodePort \
	--set prometheus.service.nodePort=30090 \
	--set grafana.service.type=NodePort \
	--set grafana.service.nodePort=30091
dash:
	minikube dashboard
clean:
	kubectl delete pods,deployments -A --all
	kubectl delete ns monitoring --wait
	kubectl delete ns insuretech --wait