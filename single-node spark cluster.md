# Spark Standalone Cluster Installation (Single Node)

- **Machine IP**: `192.168.30.95`  
- **Spark Version**: `4.0.0`  
- **Cluster Mode**: Standalone on a single machine (both Master and Worker)

---

## 🔗 Official Links

- [Spark Downloads](https://spark.apache.org/downloads.html)
- [Direct Spark TGZ Download](https://www.apache.org/dyn/closer.lua/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz)
- [Archive Versions](https://archive.apache.org/dist/spark)

---

## 📦 Installation Steps

### 1. Install Java

```bash
sudo apt install openjdk-17-jdk -y
java --version
readlink -f $(which java)
```

---

### 2. Download and Set Up Spark

```bash
wget https://dlcdn.apache.org/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz
tar -xvf spark-4.0.0-bin-hadoop3.tgz
sudo mv spark-4.0.0-bin-hadoop3 /opt/spark
```

---

### 3. Install Python 3.9 and PySpark

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.9 python3.9-distutils -y
curl -sS https://bootstrap.pypa.io/get-pip.py | sudo python3.9
pip3.9 install pyspark
```

---

### 4. Configure Environment Variables

Add the following to `~/.bashrc`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export PYSPARK_PYTHON=/usr/bin/python3.9
```

- These are for our user environment (terminal session).
- Helps Linux know where Java and Spark are installed.
- Adds Spark commands like `spark-submit`, `start-master.sh`, etc., to our terminal path.
- Tells PySpark to use Python 3.9, not the default version.
- Adding to `.bashrc` makes the settings available in every terminal session automatically.

```bash
source ~/.bashrc
```

---

### 5. Configure Spark Files

```bash
cd /opt/spark/conf
cp spark-env.sh.template spark-env.sh
cp workers.template workers
```

Edit `spark-env.sh`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_MASTER_HOST=192.168.30.95
```

- Tells Spark which Java to use.  
- Tells Spark which IP address to run the Master on.

Edit `workers`:

```bash
localhost
```

- Here in this single-node cluster scenario, our `192.168.30.95` machine acts as both master and worker.
- When we go to multi-node clusters, here we can give the IPs of the worker machines.
- Once done passwordless SSH for all workers from `192.168.30.95` (master), we can start all the Spark workers from master using a single command.

---

### 6. Configure `spark-defaults.conf`

```bash
cd /opt/spark/conf
cp spark-defaults.conf.template spark-defaults.conf
```

Edit `spark-defaults.conf` and add:

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

### 7. Prepare Logs Directory

```bash
sudo mkdir -p /opt/spark/logs
sudo chown -R $USER:$USER /opt/spark/logs
```

---

## 🚀 Start Spark Services

```bash
start-master.sh
start-worker.sh spark://192.168.30.95:7077
```

Start master with custom ports:

```bash
start-master.sh --host 192.168.30.95 --port 7077 --webui-port 9090
```

After running this:

- Spark Master URL → `spark://192.168.30.95:7077`
- Web UI → [http://192.168.30.95:9090](http://192.168.30.95:9090)

> When we give `start-master.sh` without arguments, it defaults to port `8080`. Specifying like above lets us use a custom Web UI port.

---

### ✅ Check Java Processes:

```bash
jps
# Output should include Master and Worker
```

Open Spark UI:

[http://192.168.30.95:8080](http://192.168.30.95:8080)

---

## 🧪 Test Spark Jobs

### JAR - Client Mode

```bash
spark-submit --class org.apache.spark.examples.SparkPi --master spark://192.168.30.95:7077 --deploy-mode client /opt/spark/examples/jars/spark-jar.jar 10
```

- **Driver**: Host machine (95)  
- **Executors**: Worker (95)

---

### JAR - Cluster Mode

```bash
spark-submit --class org.apache.spark.examples.SparkPi --master spark://192.168.30.95:7077 --deploy-mode cluster /opt/spark/examples/jars/spark-jar.jar 10
```

- **Driver**: Worker (95)  
- **Executors**: Worker (95)

---

### PySpark - Client Mode

```bash
spark-submit --master spark://192.168.30.95:7077 --deploy-mode client /home/syed/spark-app.py
```

- **Driver**: Host machine (95)  
- **Executors**: Worker (95)

---

### PySpark - Cluster Mode

```bash
spark-submit --master spark://192.168.30.95:7077 --deploy-mode cluster /home/syed/spark-app.py
```

- ❌ Not supported in Spark Standalone for Python apps.  
- ⚠️ Error: *Cluster deploy mode is currently not supported for python applications on standalone clusters.*

---

## 🔍 Check Spark Version

```bash
spark-submit --version
```

---

## Submitting Spark Jobs Remotely: Client Machine Requirements:

- If we need to submit jobs from any other machines, we have to install Spark binary in them.

---

## 📝 Summary

| Component        | Location (192.168.30.95) |
| ---------------- | ------------------------ |
| Master           | ✅                        |
| Worker           | ✅                        |
| Driver (Client)  | ✅ (host)                 |
| Driver (Cluster) | ✅ (worker)               |
| Executors        | ✅ (worker)               |

---


