# kafka-clickhouse

## 1. Clone the Repository

```bash
git clone https://github.com/lakshyarj/kafka-clickhouse.git
cd kafka-clickhouse/
```

## 2. Deploy Kafka Cluster
Apply the Kubernetes manifests to set up the Kafka cluster:

```bash
kubectl apply -f kafka-cluster/
kubectl apply -f kafka-cluster/zookeeper/
kubectl apply -f kafka-cluster/kafka/
```
## 3. Deploy ClickHouse Operator and Cluster
Install the ClickHouse Operator and deploy the ClickHouse cluster:
```bash
kubectl apply -f clickhouse-cluster
#Install ClickHouse Operator
curl -s https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator/clickhouse-operator-install-bundle.yaml \
  | sed 's/namespace: kube-system/namespace: clickhouse/g' \
  | kubectl apply -f -

# Deploy ClickHouse Zookeeper and Cluster
kubectl apply -f clickhouse-cluster/zookeeper/
kubectl apply -f clickhouse-cluster/clickhouse/
```
## 4. Deploy Kafka Connect
Set up Kafka Connect with the ClickHouse Sink Connector:
```bash
kubectl apply -f kafka-cluster/kafka-connect/
```
## 5. Configure ClickHouse Sink Connector
First, port-forward the Kafka Connect service so you can interact with it from your local machine:

```bash
kubectl port-forward svc/kafka-connect 8083:8083 -n kafka
```
Create a configuration.json file with the following content:
```bash
{
  "name": "clickhouse-sink",
  "config": {
    "connector.class": "com.clickhouse.kafka.connect.ClickHouseSinkConnector",
    "tasks.max": "1",
    "topics": "test",
    "clickhouse.url": "http://chi-clickhouse-main-0-0-0.chi-clickhouse-main-0-0.clickhouse.svc.cluster.local:8123",
    "clickhouse.user": "default",
    "clickhouse.password": "",
    "clickhouse.database": "default",
    "insert.mode": "INSERT",
    "auto.create.tables": "true",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "false"
  }
}
```
Apply the connector configuration using the Kafka Connect REST API:
```bash
curl -X POST -H "Content-Type: application/json" \
  --data @configuration.json \
  http://localhost::8083/connectors
```
