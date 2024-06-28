# Deploying Spark Applications on Kubernetes (GKE)

This guide demonstrates deploying Apache Spark applications on Google Kubernetes Engine (GKE). It covers setting up a Kubernetes cluster, deploying Spark, running Wordcount and PageRank tasks, and viewing job results.

## Prerequisites
- Google Cloud Platform account with billing enabled
- `gcloud` CLI installed and configured
- Helm installed on your local machine
- Basic understanding of Kubernetes and Spark

## Step-by-Step Guide

### 1. Create a GKE Cluster
```bash
gcloud container clusters create spark --num-nodes=1 --machine-type=e2-highmem-2 --region=us-west1
```
This command creates a GKE cluster named `spark` with one node of type `e2-highmem-2` in the `us-west1` region.

### 2. Install NFS Server Provisioner
```bash
helm repo add stable https://charts.helm.sh/stable
helm install nfs stable/nfs-server-provisioner --set persistence.enabled=true,persistence.size=5Gi
```
This installs an NFS Server Provisioner using Helm, providing persistent storage for Kubernetes pods with a 5Gi volume size.

### 3. Create Persistent Volume and Pod
Create `spark-pvc.yaml` with the following content:
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
Apply the YAML file:
```bash
kubectl apply -f spark-pvc.yaml
```
This creates a PersistentVolumeClaim (`spark-data-pvc`) and a Pod (`spark-data-pod`) that mounts the NFS volume, allowing multiple pods to access shared data.

### 4. Copy Application Files
Assuming you have built a JAR file for your Spark application (`my.jar`) and have a test file (`test.txt`), copy them to the mounted PVC:
```bash
kubectl cp /path/to/my.jar spark-data-pod:/data/my.jar
kubectl cp /path/to/test.txt spark-data-pod:/data/test.txt
```
This copies `my.jar` and `test.txt` into the `/data` directory of the `spark-data-pod` pod.

### 5. Deploy Spark Using Helm Chart
Create `spark-chart.yaml` to configure Spark deployment:
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
Install Spark using the Helm chart:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install spark bitnami/spark -f spark-chart.yaml
```
This deploys Apache Spark on Kubernetes with 3 worker replicas, configured to use the shared PVC (`spark-data-pvc`) for data storage.

### 6. Run Wordcount Task
Submit a Wordcount job to the Spark cluster:
```bash
kubectl run --namespace default spark-client --rm --tty -i --restart='Never' \
  --image docker.io/bitnami/spark:3.0.1-debian-10-r115 \
  -- spark-submit --master spark://<LOAD-BALANCER-EXTERNAL-IP>:7077 \
  --deploy-mode cluster --class org.apache.spark.examples.JavaWordCount \
  /data/my.jar /data/test.txt
```
Replace `<LOAD-BALANCER-EXTERNAL-IP>` with the actual external IP of the Spark load balancer. This command runs the Wordcount application on the Spark cluster.

### 7. Run PageRank Using PySpark
Access the Spark master pod to run PageRank using PySpark:
```bash
kubectl exec -it spark-master-0 -- bash
```
Inside the pod, navigate to the PySpark examples directory:
```bash
cd /opt/bitnami/spark/examples/src/main/python
```
Run the PageRank algorithm:
```bash
spark-submit pagerank.py /opt 2
```
Adjust `/opt` and `2` as needed for your specific inputs and iterations.

### 8. Viewing Results
To view job results, access a worker node pod:
```bash
kubectl get pods -o wide  # Identify worker node pod
kubectl exec -it <worker-pod-name> -- bash
cd /opt/bitnami/spark/work/driver-*/  # Navigate to the appropriate directory
cat stdout  # View the output of the job
```
Replace `<worker-pod-name>` with the actual name of the worker node pod where the job ran.

# Test(Please follow the pdf below):
[Implementation Instruction (PDF)](https://github.com/ASD-Are/Big_Data/blob/main/Work%20Count%20%2B%20PageRank/WordCount__PageRank_Implementation_Guide.pdf)

# Appendix
[Google Slide](https://docs.google.com/presentation/d/1v4RjcHWXDSsFP7fNxkmHR-Ha0ivCokpLYhtMXGAvDXA/edit?usp=sharing)
