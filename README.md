# spark-docker
Esay-to-use spark-docker(Spark 3.5.7), better deployed using docker-compose.yaml, good practices for **local development**, **testing** and **learning**.

I made this cause bitnami **no longer** provides free Spark docker image on docker hub.

**Three** modes provided:
- master
- worker
- thrift(Specially for kafka, dbt and Delta Lake)

# Environment variables
## environment variables
### Customizable environment variables
|Name|Description|Default Value|
|-|-|-|
|**`SPARK_MODE`**|Spark cluster mode to run \[ master, worker, thrift \]|**`master`**|
|**`SPARK_MASTER_URL`**|Url where the worker can find the master. (Have to be set for worker and thrift mode)|**`spark://spark-master:7077`**|
|**`SPARK_CONF_DIR`**|Dir of `spark-defaults.conf`|**`/opt/spark/conf`**|
|**`SPARK_MASTER_WEBUI_PORT`**|Master web ui port|**`8080`**|
|**`SPARK_MASTER_HOST`**|Master host name|**`spark-master`**|
|**`SPARK_WORKER_WEBUI_PORT`**|Worker web ui port|**`8081`**|
|**`SPARK_WORKER_CORES`**|CPU cores used by worker|**`2`**|
|**`SPARK_WORKER_MEMORY`**|Max memories used by worker|**`2g`**|
|**`HIVE2_THRIFT_PORT`**|Port of thrift server|**`10000`**|
|**`HIVE2_THRIFT_DOAS_ENABLE`**|Whether to require client authentication.|**`false`**|

### Read-only environment variables

|Name|Description|Default Value|
|-|-|-|
|**`SPARK_HOME`**|Spark home|**`master`**|
|**`SPARK_IVY_DIR`**|Ivy packages cache, used to add customized jars into `SPARK_PATH`, better be mounted to a volume.|**`/opt/spark/.ivy2`**|
|**`SPARK_USER`**|	Spark user.| **`spark`**|

## Usage
### docker-compose.yaml example
```yaml
spark-master:
    build:
      context: ./docker/spark-docker/3.5.7
    environment:
      SPARK_MODE: master
    ports:
      - "7077:7077"
      - "8080:8080"
    volumes:
      - ./conf/spark-defaults.conf:/opt/spark/conf/spark-defaults.conf

spark-worker:
  build:
    context: ./docker/spark-docker/3.5.7
  depends_on:
    - spark-master
  environment:
    SPARK_MODE: worker
    # Set SPARK_MASTER_URL if you have customized url.
    SPARK_MASTER_URL: spark://spark-master:7077
    SPARK_WORKER_CORES: 4
    SPARK_WORKER_MEMORY: 8g
    SPARK_WORKER_WEBUI_PORT: 8082
  ports:
    # Should be as same as env $SPARK_WORKER_WEBUI_PORT
    - "8082:8082"
  volumes:
    - ./ivy2:/opt/spark/.ivy2
    - ./spark-data:/opt/spark/data
    # Defaul offical Apache Spark work-dir is /opt/spark/work-dir
    - ./spark-warehouse:/opt/spark/work-dir/warehouse
    - ./metastore_db:/opt/spark/work-dir/metastore_db
    - ./conf/spark-defaults.conf:/opt/spark/conf/spark-defaults.conf

spark-thrift:
  build:
    context: ./docker/spark-docker/3.5.7
  depends_on:
    - spark-master
    - spark-worker
  environment:
    SPARK_MODE: thrift
    SPARK_MASTER_URL: spark://spark-master:7077
  volumes:
    - ./ivy2:/opt/spark/.ivy2
    - ./conf/spark-defaults.conf:/opt/spark/conf/spark-defaults.conf
```

### spark-defaults.conf
An spark-defaults.conf file s provided for kafka, dbt, and delta lake.

### More

It is recommended to use an indepentend jupyter-pyspark server or dbt server.
