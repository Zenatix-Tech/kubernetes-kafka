### Deployment Guidelines
**1. Configure Storage Class**
```bash
kubectl apply -f configure/aks-storageclass-zookeeper-managed.yml
kubectl apply -f configure/aks-storageclass-broker-managed.yml

#Delete
kubectl delete -f configure/aks-storageclass-zookeeper-managed.yml
kubectl delete -f configure/aks-storageclass-broker-managed.yml
```

**2. Configure Namespace**
```bash
kubectl apply -f 00-namespace.yml

#Delete
kubectl delete -f 00-namespace.yml
```

**3. Configure persistent zookeeper**
```bash
kubectl apply -f zookeeper/10zookeeper-config.yml
kubectl apply -f zookeeper/20pzoo-service.yml
kubectl apply -f zookeeper/30service.yml
kubectl apply -f zookeeper/50pzoo.yml

#Delete
kubectl delete -f zookeeper/
```

**4. Configure kafka**
```bash
#Apply all
kubectl appply -f kafka/

#Apply services
kubectl apply -f kafka/10broker-config.yml
kubectl apply -f kafka/20dns.yml
kubectl apply -f kafka/30bootstrap-service.yml
kubectl apply -f kafka/50kafka.yml

#Delete
kubectl delete -f kafka/
```

**5. Configure kafka-manager**
```bash
kubectl apply -f yahoo-kafka-manager/kafka-manager.yml
kubectl apply -f yahoo-kafka-manager/kafka-manager-service.yml
```

**6. Configure schema-registry**
```bash
helm install --name cp-schema-registry -f cp-schema-registry/values.yaml --namespace kafka ./cp-schema-registry
```

**7. Configure kafka-rest-proxy**
```bash
helm install --name cp-kafka-rest -f cp-kafka-rest/values.yaml --namespace kafka ./cp-kafka-rest
```

**8. Building and pushing kafka-connect custom image with mqtt plugin**
```bash
docker-compose -f cp-kafka-connect/lib/docker-compose-kafka-connect.yml build
docker-compose -f cp-kafka-connect/lib/docker-compose-kafka-connect.yml push
```

**9. Configure kafka-connect**
```bash
helm install --name cp-kafka-connect -f cp-kafka-connect/chart/values.yaml --namespace kafka ./cp-kafka-connect/chart
```

**10. Add worker to mqtt connector**
```bash
kubectl exec -n kafka -it cp-kafka-connect-5c6f848c87-x99v5 -c cp-kafka-connect-server -- /bin/bash

curl -s -X POST -H "Content-Type: application/json" --data '{"name": "smap-mqtt-source-lenses", "config": {"connector.class": "com.datamountaineer.streamreactor.connect.mqtt.source.MqttSourceConnector", "tasks.max":"1", "connect.mqtt.hosts":"tcp://104.211.220.105:1883", "connect.mqtt.username":"zenatix_mqtt_client", "connect.mqtt.password":"xitanez123", "connect.mqtt.service.quality":"1", "connect.mqtt.kcql":"INSERT INTO smap_telemetry_data SELECT * FROM telemetry/+/+ WITHCONVERTER=`com.datamountaineer.streamreactor.connect.converters.source.BytesConverter`"}}' http://localhost:8083/connectors

curl -s -X GET http://kafka-connect:8082/connectors/

```

**10. Metrics collection**
```bash
kubectl --namespace kafka apply -f prometheus/10-metrics-config.yml 
kubectl --namespace kafka patch statefulset kafka --patch "$(cat prometheus/50-kafka-jmx-exporter-patch.yml )"
```

**Install Grafana dashboard**

Import - `https://github.com/confluentinc/cp-helm-charts/blob/master/grafana-dashboard/confluent-open-source-grafana-dashboard.json`

**9. Delete Deployments**
```bash
helm list
helm delete --purge my-kafka-connect

```

**Accessing Services**
```bash
kubectl 
```