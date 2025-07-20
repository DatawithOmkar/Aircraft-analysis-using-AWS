## 🔧 Setting Up AWS EC2 & EMR Cluster

Follow these steps to configure your AWS environment and launch an EMR cluster for processing the airline delay dataset with Hadoop and Hive.

---

### ✅ Prerequisites

- An AWS account
- IAM user with permission to use EMR, EC2, and S3
- An existing **Key Pair (.pem)** file (or create one in EC2 > Key Pairs)
- SSH client or PuTTY (for Windows)
- WinSCP (for file transfer)

---

### 🖥️ Step 1: Launch an EMR Cluster

1. Go to the [AWS EMR Console](https://console.aws.amazon.com/elasticmapreduce)
2. Click **“Create cluster”**
3. Choose **“Go to advanced options”**
4. Under **Software Configuration**, select:
   - Application: **Hadoop**, **Hive** (you can also select Spark if needed)
   - Release: Choose latest **Amazon EMR release** (e.g., emr-6.15.0)
5. Click **Next**

---

### ⚙️ Step 2: Configure Hardware

1. **Number of nodes**:
   - 1 Master (e.g., m5.xlarge)
   - 2 Core nodes (m5.xlarge or adjust based on dataset size)
2. For testing, you can use smaller instances like `m4.large`
3. Set **EC2 Key Pair**: Select your existing `.pem` key pair

---

### 🔐 Step 3: Networking & Permissions

- VPC: Default (or a custom one if you have it set up)
- Subnet: Choose default or a specific subnet
- Leave default security groups, or allow port `22` for SSH if needed
- Choose a role with **EMR_DefaultRole** and **EMR_EC2_DefaultRole**

---

### 🚀 Step 4: Launch the Cluster

- Click **Create cluster**
- Wait 5–10 minutes for the cluster to change status from **STARTING** to **WAITING**
- Copy the **Master Public DNS** — you’ll use this to connect via SSH or WinSCP

---

### 🧳 Step 5: Transfer Files to EMR Master Node (WinSCP)

1. Open **WinSCP**
2. Select SCP/SFTP protocol
3. Enter:
   - Host: `ec2-xxx.compute.amazonaws.com` (from EMR Master Public DNS)
   - Username: `hadoop`
   - Private key file: your `.pem` key
4. Upload your CSV (e.g., `1987.csv`) to the `/home/hadoop/` directory

---

### 🖧 Step 6: SSH into the Master Node

Use your terminal (macOS/Linux) or PuTTY (Windows):

```bash
ssh -i path/to/your-key.pem hadoop@<your-master-dns>
```

###  7: Interact with HDFS & Hive
```
hdfs dfs -mkdir -p /user/omkar_airlines
hdfs dfs -put /home/hadoop/1987.csv /user/omkar_airlines/

```
Open Hive shell:
```
hive
```

###  Important Cleanup (After Completion)

To avoid ongoing AWS charges:

Go to EMR Console → Select your cluster → Click Terminate
Delete your S3 buckets or EC2 Key Pairs if not needed
