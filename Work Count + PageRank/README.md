# Spark Application Deployment on Kubernetes (GKE)

## Overview
This project demonstrates deploying Apache Spark applications on Google Kubernetes Engine (GKE). It includes setups for running Wordcount and PageRank tasks using Spark.

## Prerequisites
- Google Cloud Platform account
- `gcloud` CLI configured
- Kubernetes cluster created on GKE

## Steps to Deploy

### 1. Create GKE Cluster
```bash
gcloud container clusters create spark --num-nodes=1 --machine-type=e2-highmem-2 --region=us-west1
```

### 2. Install NFS Server Provisioner
```bash
helm repo add stable https://charts.helm.sh/stable
helm install nfs stable/nfs-server-provisioner --set persistence.enabled=true,persistence.size=5Gi
```

### 3. Create Persistent Volume and Pod
Create `spark-pvc.yaml` with:
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: spark-data-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  storageClassName: nfs

---
apiVersion: v1
kind: Pod
metadata:
  name: spark-data-pod
spec:
  volumes:
    - name: spark-data-pv
      persistentVolumeClaim:
        claimName: spark-data-pvc
  containers:
    - name: inspector
      image: bitnami/minideb
      command:
        - sleep
        - infinity
      volumeMounts:
        - mountPath: "/data"
          name: spark-data-pv
```
Apply the YAML:
```bash
kubectl apply -f spark-pvc.yaml
```

### 4. Copy Application Files
```bash
# Copy JAR file and test data to the pod
kubectl cp /tmp/my.jar spark-data-pod:/data/my.jar
kubectl cp /tmp/test.txt spark-data-pod:/data/test.txt
```

### 5. Deploy Spark using Helm Chart
Create `spark-chart.yaml`:
```yaml
service:
  type: LoadBalancer
worker:
  replicaCount: 3
  extraVolumes:
    - name: spark-data
      persistentVolumeClaim:
        claimName: spark-data-pvc
  extraVolumeMounts:
    - name: spark-data
      mountPath: /data
```
Install Spark:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install spark bitnami/spark -f spark-chart.yaml
```

### 6. Run Wordcount Task
```bash
kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
  --image docker.io/bitnami/spark:3.0.1-debian-10-r115 \
  -- spark-submit --master spark://LOAD-BALANCER-External-ip-ADDRESS:7077 \
  --deploy-mode cluster --class org.apache.spark.examples.JavaWordCount \
  /data/my.jar /data/test.txt
```

### 7. Run PageRank using PySpark
```bash
# Connect to Spark master pod
kubectl exec -it spark-master-0 -- bash

# Navigate to PySpark examples directory
cd /opt/bitnami/spark/examples/src/main/python

# Run PageRank
spark-submit pagerank.py /opt 2
```

## Viewing Results
To view job results:
```bash
kubectl exec -it spark-worker-0 -- bash
cd /opt/bitnami/spark/work
ls -l
cd driver-*/  # Check the exact directory
cat stdout
```
