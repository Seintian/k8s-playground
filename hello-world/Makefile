.PHONY: cluster-up cluster-down start stop get logs status test-heal

CLUSTER_NAME := desktop
APP_LABEL := app=webserver

cluster-up:
	@echo "=> Booting Kind cluster '$(CLUSTER_NAME)'..."
	@kind create cluster --name $(CLUSTER_NAME) --config kind-config.yaml --image kindest/node:v1.34.0
	@echo "=> Waiting for the Kubernetes control plane to be Ready..."
	@kubectl wait --for=condition=Ready nodes --all --timeout=120s
	@echo "=> Cluster is up and ready!"

cluster-down:
	@echo "=> Tearing down Kind cluster '$(CLUSTER_NAME)'..."
	@kind delete cluster --name $(CLUSTER_NAME)

start:
	@echo "=> Applying Kubernetes manifests..."
	@kubectl apply -f deployment.yaml
	@kubectl apply -f service.yaml
	@echo "=> Waiting for Deployment to reach desired state..."
	@kubectl rollout status deployment/nginx-deployment --timeout=90s
	@echo "=> Deployment is healthy! Access it natively at http://localhost:30080"

stop:
	@echo "=> Deleting Kubernetes resources..."
	@kubectl delete -f deployment.yaml --ignore-not-found
	@kubectl delete -f service.yaml --ignore-not-found

get:
	@echo "=> Current Pods:"
	@kubectl get pods -l $(APP_LABEL)
	@echo "\n=> Current Services:"
	@kubectl get services

logs:
	@echo "=> Fetching logs for the first available Pod..."
	@POD_NAME=$$(kubectl get pods -l $(APP_LABEL) -o jsonpath='{.items[0].metadata.name}' 2>/dev/null); \
	if [ -z "$$POD_NAME" ]; then echo "Error: No Pods found."; else kubectl logs -f $$POD_NAME; fi

status:
	@echo "=> Pod Status Overview:"
	@kubectl get pods -l $(APP_LABEL) -o custom-columns=NAME:.metadata.name,PHASE:.status.phase,READY:.status.containerStatuses[0].ready

test-heal:
	@echo "=> Initiating Chaos Test..."
	@POD_NAME=$$(kubectl get pods -l $(APP_LABEL) -o jsonpath='{.items[0].metadata.name}' 2>/dev/null); \
	if [ -z "$$POD_NAME" ]; then \
		echo "Error: No Pods running to kill."; \
	else \
		echo "=> Destroying Pod: $$POD_NAME"; \
		kubectl delete pod $$POD_NAME; \
		echo "=> Watch the ReplicaSet spawn a replacement (Press Ctrl+C to exit):"; \
		kubectl get pods -l $(APP_LABEL) -w; \
	fi
