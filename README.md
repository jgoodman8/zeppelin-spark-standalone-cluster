# Zeppelin with Spark Standalone Cluster

- Apache Zeppelin version: 0.8.0
- Apache Spark version: 2.2.0

### Before launching the containers

```bash
docker run --name=master -d bde2020/spark-master:2.2.0-hadoop2.7
docker cp master:/spark spark
docker stop master
```

### Launch docker compose

- Spark

It launch a cluster with a master node and two worker nodes. The UI is running at the 8181 (moved due collision with the zeppelin UI port).

```
spark-master:
    image: bde2020/spark-master:2.2.0-hadoop2.7
    container_name: spark-master
    expose:
        - "7077"
    ports:
        - "8181:8080"
        - "7077:7077"
    environment:
        - INIT_DAEMON_STEP=setup_spark
    volumes:
        - ./spark:/spark
spark-worker-1:
    image: bde2020/spark-worker:2.2.0-hadoop2.7
    container_name: spark-worker-1
    depends_on:
        - spark-master
    ports:
        - "8082:8082"
    environment:
        - "SPARK_MASTER=spark://spark-master:7077"
spark-worker-2:
    image: bde2020/spark-worker:2.2.0-hadoop2.7
    container_name: spark-worker-2
    depends_on:
        - spark-master
    ports:
        - "8083:8083"
    environment:
        - "SPARK_MASTER=spark://spark-master:7077"
```

- Zeppelin

The UI is defined to run at the 8080. All the created notebooks are mapped to a volume at the `./notebooks` local directory in the host.

```
zeppelin:
    image: apache/zeppelin:0.8.0
    ports:
        - "8080:8080"
    volumes:
        - ./spark:/zeppelin/spark
        - ./notebooks:/zeppelin/notebook
    environment:
        - "MASTER=spark://spark-master:7077"
        - "SPARK_MASTER=spark://spark-master:7077"
        - "SPARK_HOME=/zeppelin/spark"
```