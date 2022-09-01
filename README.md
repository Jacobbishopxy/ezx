# EZX

Easy transaction for data infrastructure.

## Charts

- [kubeapps](https://github.com/vmware-tanzu/kubeapp)
- [postgresql](https://github.com/bitnami/charts/tree/master/bitnami/postgresql)
- [minio](https://github.com/bitnami/charts/tree/master/bitnami/minio)
- [spark](https://github.com/bitnami/charts/tree/master/bitnami/spark)

## Notes

### Postgresql

- Needs pre-declared PV & PVC for `my-values.yaml`:

  ```yaml
  primary:
  persistence:
    existingClaim: "postgres-pvc"
  ```

- Needs service as `NodePort` to expose internal IP:

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: xy-pg-port
    namespace: dev
  spec:
    selector:
      statefulset.kubernetes.io/pod-name: <pod name>
    type: NodePort
  ```

### Minio

- Needs pre-declared PV & PVC for `my-values.yaml`:

  ```yaml
  primary:
  persistence:
    existingClaim: "minio-pvc"
  ```

- Check username/password of a deployed standalone MinIO:

  Due to `hostPath` persistent, login to node IP then execute:

  ```sh
  cat <minio path>/.root_user
  cat <minio path>/.root_password
  ```

### Spark

- Since we are using k8s spark cluster (see [detail](https://stackoverflow.com/a/68779353)), we need [bcpkix-jdk15on](https://mvnrepository.com/artifact/org.bouncycastle/bcpkix-jdk15on) & [bcprov-jdk15on](https://mvnrepository.com/artifact/org.bouncycastle/bcprov-jdk15on) for `spark-submit`. In other words, these two dependencies must be included in `$SPARK_HOME/jars`.

- In addition, we need [hadoop-aws](https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-aws) as an extra package while executing `spark-submit`.

#### Client Mode

- NFS share volume (Only required in Spark Client Mode, which used for uploading local JARs).

  On the client server:

  ```sh
  sudo apt update
  sudo apt install nfs-common
  ```

  Check available mounting directories:

  ```sh
  showmount -e <HOST_IP>
  ```

  Make the share directory and grant permission:

  ```sh
  sudo mkdir <YOUR_MOUNT_DIRECTORY> -p
  sudo chown nobody:nogroup <YOUR_MOUNT_DIRECTORY>
  ```

  Mount host directory:

  ```sh
  sudo mount <HOST_IP>:<HOST_SHARE_ADDRESS> <YOUR_MOUNT_DIRECTORY>
  ```

- Submit an application by `kubectl run` (k8s PVC is required for `${SHARE_VOLUME_ADDR}`):

  ```sh
  kubectl run \
    --namespace ${K8S_NAMESPACE} ${JOB_NAME} \
    --rm \
    --tty -i \
    --restart='Never' \
    --image ${K8S_CONTAINER_IMAGE} \
    -- spark-submit \
    --master ${SPARK_ADDR} \
    --deploy-mode cluster \
    --class ${CLASS_NAME} \
    ${SHARE_VOLUME_ADDR}/${PROJECT_JAR}
  ```

#### Cluster Mode

Submit an application by `spark-submit` (MinIO is required for `s3a`):

```sh
spark-submit \
 --master k8s://${MASTER_ADDR} \
 --deploy-mode cluster \
 --name ${JOB_NAME} \
 --class ${CLASS_NAME} \
 --packages org.apache.hadoop:hadoop-aws:3.3.4 \
 --conf spark.kubernetes.driverEnv.SPARK_MASTER_URL=${SPARK_ADDR} \
 --conf spark.kubernetes.file.upload.path=s3a://${S3_BUCKET} \
 --conf spark.kubernetes.container.image=${K8S_CONTAINER_IMAGE} \
 --conf spark.kubernetes.namespace=${K8S_NAMESPACE} \
 --conf spark.hadoop.fs.s3a.endpoint=${SPARK_HADOOP_ENDPOINT} \
 --conf spark.hadoop.fs.s3a.connection.ssl.enabled=false \
 --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
 --conf spark.hadoop.fs.s3a.fast.upload=true \
 --conf spark.hadoop.fs.s3a.path.style.access=true \
 --conf spark.hadoop.fs.s3a.access.key=${SPARK_HADOOP_ACCESS_KEY} \
 --conf spark.hadoop.fs.s3a.secret.key=${SPARK_HADOOP_SECRET_KEY} \
 file://${TARGET_PATH}/target/scala-2.12/${PROJECT_JAR}
```

## Materials

- [How to run Spark with Minio in K8s](https://www.youtube.com/watch?v=ZzFdYm_DqEM)

## Todo

- argo-workflows
- grafana
- jupyterhub
- jenkins
- kafka
- mongodb
- redis
