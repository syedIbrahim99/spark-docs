# Spark Standalone Cluster Setup (Multi-Node)

## 🔗 Official Documentation

- [Spark Downloads](https://spark.apache.org/downloads.html)
- [Spark 4.0.0 Direct Download (Hadoop 3)](https://www.apache.org/dyn/closer.lua/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz)
- [Archive Versions](https://archive.apache.org/dist/spark/)

## 🖥️ Machine Details

- **Machine A (Master)**: `192.168.30.95`
- **Machine B (Worker)**: `192.168.30.96`

---

## ✅ Common Setup for Both Machines (Master & Worker)

### 🔹 Java Installation

```bash
sudo apt install openjdk-17-jdk -y
java --version
```

---

### 🔹 Download and Install Spark

```bash
wget https://www.apache.org/dyn/closer.lua/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz
tar -xvf spark-4.0.0-bin-hadoop3.tgz
sudo mv spark-4.0.0-bin-hadoop3 /opt/spark
readlink -f $(which java)
```

---

### 🔹 Python and PySpark Installation

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.9 python3.9-distutils -y
curl -sS https://bootstrap.pypa.io/get-pip.py | sudo python3.9
pip3.9 install pyspark
```

---

### 🔹 Update `~/.bashrc`

```bash
nano ~/.bashrc
```

Add:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export PYSPARK_PYTHON=/usr/bin/python3.9
```
* These are for our user environment (terminal session).
* Helps Linux know where Java and Spark are installed.
* Adds Spark commands like spark-submit, start-master.sh, etc., to our terminal path.
* Tells PySpark to use Python 3.9, not the default version.
* Adding to .bashrc makes the settings available in every terminal session automatically.

Apply changes:

```bash
source ~/.bashrc
```

---

### 🔹 Configure Spark Environment (Common Part)

```bash
cd /opt/spark/conf
cp spark-env.sh.template spark-env.sh
nano spark-env.sh
```

Add this common line:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_MASTER_HOST=192.168.30.95
export SPARK_WORKER_CORES=2
export SPARK_WORKER_MEMORY=2g
export SPARK_MASTER_URL=spark://192.168.30.95:7077
```

* Tells Spark which Java to use.
* Tells Spark which IP address to run the Master on.
* Limits the Worker to use only 2 CPU cores.
* Limits the Worker to use only 2 GB of RAM.
* Specifies the full Master URL used by Workers to connect.

Create logs directory:

```bash
sudo mkdir -p /opt/spark/logs
sudo chown -R $USER:$USER /opt/spark/logs
```

---

## 🖥️ Machine A (Master) - Specific Setup

### ▶️ Start Spark Master

```bash
start-master.sh
```

Check with:

```bash
jps
```

Output should show:

```
Master
```

Access Spark Master Web UI:  
👉 [http://192.168.30.95:8080](http://192.168.30.95:8080)

---

## 🖥️ Machine B (Worker) - Specific Setup


### ▶️ Start Spark Worker

```bash
start-worker.sh spark://192.168.30.95:7077
```

Check with:

```bash
jps
```

Output should show:

```
Worker
```

You can confirm the Worker is connected by visiting:  
👉 [http://192.168.30.95:8080](http://192.168.30.95:8080)

---

## 🚀 Submitting Applications

### 🔸 JAR - Client Mode

```bash
spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://192.168.30.95:7077 \
  --deploy-mode client \
  /opt/spark/examples/jars/spark-examples_2.12-4.0.0.jar 10
```

- Driver runs on **Machine A (Master)**
- JAR file is served over HTTP to workers

---

### 🔸 JAR - Cluster Mode

```bash
spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://192.168.30.95:7077 \
  --deploy-mode cluster \
  /opt/spark/examples/jars/spark-examples_2.12-4.0.0.jar 10
```

- Driver runs on **Machine B (Worker)**
- JAR file must be present on the worker

**If not:**  
You will get:  
`java.nio.file.NoSuchFileException`

**Solutions:**

- Copy the JAR to all workers manually
- Use NFS/shared volume
- Use HTTP Server (e.g., NGINX)
- Use S3/MinIO bucket

---

## 🐍 PySpark Files

### 🔸 Client Mode (Supported)

```bash
spark-submit \
  --master spark://192.168.30.95:7077 \
  --deploy-mode client \
  /home/syed/spark-app.py
```

Driver runs on **Machine A**

---

### 🔸 Cluster Mode (Not Supported)

```bash
spark-submit \
  --master spark://192.168.30.95:7077 \
  --deploy-mode cluster \
  /home/syed/spark-app.py
```

❌ **Not supported in standalone mode for Python apps**  
❗ Error: `Cluster deploy mode is currently not supported for python applications on standalone clusters.`

---

## ⚙️ `spark-defaults.conf` (Machine A - Optional)

```bash
spark.master                     spark://192.168.30.95:7077
spark.driver.memory              1g
spark.driver.cores               1
spark.executor.memory            1g
spark.executor.cores             1
spark.executor.instances         1
```
📌 **Explanation:**

The **driver** is responsible for:

- Coordinating the execution of the application.
- Reading the job or input files (either from the local system or a shared/distributed location).
- Sending tasks to executors for actual processing.

Since the driver's job is primarily orchestration and not data-heavy computation, it does not require heavy resources. However, it should still have enough memory and CPU to manage tasks and job scheduling efficiently.

The **executors**, on the other hand, are:

- Responsible for executing tasks assigned by the driver.
- Doing the actual computation, shuffling, and storing intermediate results.

Therefore, executors need more resources (memory and cores), especially when working with large datasets or parallel processing.

we can allocate resources for executors based on the size of the job, dataset, and system capacity.

---


