# Check if Kubernetes cluster is running
kubectl cluster-info

# Verify Helm is installed
helm version

# Add Strimzi Helm repository (Kafka operator charts)
helm repo add strimzi https://strimzi.io/charts/

# Update Helm repositories
helm repo update

# Create namespace for Kafka resources
kubectl create namespace kafka

# Install Strimzi Kafka Operator in kafka namespace
helm install strimzi-kafka-operator strimzi/strimzi-kafka-operator --namespace kafka

# Watch operator pod until it is Running
kubectl get pods -n kafka -w

# Apply KafkaNodePool (defines Kafka nodes)
kubectl apply -f kafka-nodepool.yaml

# Apply Kafka cluster (creates Kafka broker + services)
kubectl apply -f kafka-cluster.yaml

# Watch Kafka pods startup (broker + entity operator)
kubectl get pods -n kafka -w

# Check Kafka cluster status (should be READY=True)
kubectl get kafka -n kafka

# Create Kafka topic using CRD
kubectl apply -f kafka-topic.yaml

# List Kafka topics managed by Strimzi
kubectl get kafkatopics -n kafka

# Run temporary Kafka CLI pod to list topics (test connectivity)
kubectl run kafka-test -n kafka --rm -it \
  --image=quay.io/strimzi/kafka:0.49.1-kafka-4.1.1 \
  --restart=Never \
  -- bin/kafka-topics.sh --bootstrap-server task-events-kafka-bootstrap:9092 --list

# Check Kafka bootstrap service (connection endpoint)
kubectl get svc -n kafka | grep bootstrap

# Force delete stuck test pod (if needed)
kubectl delete pod kafka-test -n kafka --force --grace-period=0

# Run debug pod for manual testing
kubectl run kafka-test -n kafka \
  --image=quay.io/strimzi/kafka:latest-kafka-3.7.0 \
  --restart=Never \
  -- sleep 3600

# Exec into debug pod shell
kubectl exec -it kafka-test -n kafka -- bash
