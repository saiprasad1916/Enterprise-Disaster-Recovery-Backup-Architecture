# 📘 Enterprise Disaster Recovery & Backup Architecture (AWS)

## 📌 Project Overview

This project demonstrates the design and implementation of a **robust Enterprise Disaster Recovery (DR) and Backup Architecture on AWS**.

It ensures:

* High availability
* Data protection
* Business continuity

Designed for a **financial services organization**, the system meets strict SLA requirements:

* **RPO (Recovery Point Objective):** 1 hour
* **RTO (Recovery Time Objective):** 4 hours

---

## 🏗️ Architecture Components

The solution uses the following AWS services:

* **Amazon EC2** – Compute resources
* **Amazon EBS** – Block storage (Multi-Attach)
* **Amazon EFS** – Shared scalable storage
* **Amazon S3** – Object storage
* **AWS Backup** – Centralized backup management
* **AWS Storage Gateway** – Hybrid cloud integration
* **S3 Cross-Region Replication (CRR)** – Disaster recovery

---

## 🚀 Step-by-Step Implementation

---

## 🔹 STEP 1: Launch EC2 Instances

### Navigate:

`AWS Console → EC2 → Launch Instance`

### Configuration:

* Name: `AppServer1`, `AppServer2`
* AMI: Amazon Linux 2
* Instance Type: `t2.micro` / `t3.micro`
* Network:

  * Select VPC
  * Enable Auto-assign Public IP
* Security Group:

  * Allow SSH (22)
  * Allow HTTP (80)

👉 Launch **2 instances** (required for Multi-Attach demo)

### 📸 Screenshots:
<img width="691" height="202" alt="image" src="https://github.com/user-attachments/assets/e90f8d81-8db6-4477-91b4-deca1c44f1eb" />

---

## 🔹 STEP 2: Create & Attach EBS Volume (Multi-Attach)

### Navigate:

`EC2 → Volumes → Create Volume`

### Configuration:

* Type: `io2`
* Size: `100 GB`
* Enable: **Multi-Attach**
* Same AZ as EC2

### Attach Volume:

* Attach to `AppServer1`
* Attach to `AppServer2`

### 📸 Screenshots:

* Volume creation
* <img width="691" height="214" alt="image" src="https://github.com/user-attachments/assets/76b225e5-0ea7-489c-a223-70ef2842b049" />

* Attach to EC2
* <img width="691" height="232" alt="image" src="https://github.com/user-attachments/assets/682769f8-d541-41e5-8d11-325bf8c56a9e" />

* Multi-Attach enabled
* <img width="692" height="296" alt="image" src="https://github.com/user-attachments/assets/1d759a5e-4b9b-4dce-a9eb-6a0fdc38df0c" />
<img width="691" height="300" alt="image" src="https://github.com/user-attachments/assets/31205b84-3693-40fb-a6c8-00cb8f672b0f" />


---

## 🔹 STEP 3: Mount EBS Volume

### SSH into Instance:

```bash
ssh ec2-user@<public-ip>
```

### Commands:

```bash
lsblk
sudo mkfs -t ext4 /dev/xvdf
sudo mkdir /data
sudo mount /dev/xvdf /data
```
<img width="692" height="187" alt="image" src="https://github.com/user-attachments/assets/578fcf0f-5f7e-4a9b-a453-4d245d15633d" />

<img width="692" height="337" alt="image" src="https://github.com/user-attachments/assets/4ca339a0-42a0-4b04-b1a8-ba3efa424504" />
<img width="692" height="175" alt="image" src="https://github.com/user-attachments/assets/4bb86342-f4d4-42d4-98f8-00ce7d6861be" />

👉 On **second instance**, DO NOT format again — only mount.

### 📸 Screenshots:

* `lsblk` output
* Filesystem creation
* Mount verification

---

## 🔹 STEP 4: Resize EBS Volume (100GB → 150GB)

### Console:

* Select Volume → Modify Volume
* Change size → `150 GB`

### Instance Commands:

```bash
lsblk
sudo growpart /dev/xvdf 1
sudo resize2fs /dev/xvdf1
```

### 📸 Screenshots:
<img width="692" height="196" alt="image" src="https://github.com/user-attachments/assets/6082732b-3dbf-48a7-8604-e5f7951c2b1e" />
<img width="692" height="172" alt="image" src="https://github.com/user-attachments/assets/20464c23-cd10-48b5-a889-89fffc2cdb59" />



---

## 🔹 STEP 5: Create EFS File System

### Navigate:

`AWS Console → EFS → Create File System`
<img width="691" height="258" alt="image" src="https://github.com/user-attachments/assets/cd65eb25-b68e-4a74-a91b-18bb01ca556b" />


### Configuration:

* Name: `EnterpriseEFS`
* Select VPC

---

### Mount EFS on EC2:

#### Install:

```bash
sudo yum install -y amazon-efs-utils
```

#### Mount:

```bash
sudo mkdir /efs
sudo mount -t efs fs-xxxx:/ /efs
```

👉 Shared storage across both instances

---

### Lifecycle Policy:

* IA: 30 days
* Archive: 90 days

### 📸 Screenshots:

* File system creation
* Mount success

---

## 🔹 STEP 6: Create S3 Buckets

### Primary Bucket:

* Name: `enterpriseprimarybucket`
* Region: `us-east-1`
* Enable Versioning

### DR Bucket:

* Name: `enterprisedrbucket`
* Region: `us-west-2`
* Enable Versioning

### 📸 Screenshots:

* Bucket creation
<img width="691" height="197" alt="image" src="https://github.com/user-attachments/assets/2c5f5f82-af20-4efb-bd69-3168549dc4e2" />

---

## 🔹 STEP 7: Enable Cross-Region Replication (CRR)

### Steps:

1. Open Primary Bucket
2. Go to **Management → Replication**
3. Create Rule:

   * Destination → DR bucket
   * Replicate all objects

### 📸 Screenshots:

* Replication rule setup

---

## 🔹 STEP 8: Configure AWS Backup

### Create Backup Vault:

* Name: `EnterpriseBackupVault`
* Enable KMS Encryption

---

### Backup Plan:

#### Rule 1: Hourly EBS

* Frequency: Hourly
* Retention: 24 hours
<img width="691" height="288" alt="image" src="https://github.com/user-attachments/assets/b0d7c527-e785-4196-a794-4b5c7f4d303a" />

#### Rule 2: Daily EFS + AMI

* Frequency: Daily
* Retention: 30 days
<img width="691" height="275" alt="image" src="https://github.com/user-attachments/assets/d5cffc16-2585-4afe-8d25-d698c514779f" />

#### Rule 3: Cross-Region Copy

* Destination: `us-west-2`
* Retention: 1 year
<img width="692" height="290" alt="image" src="https://github.com/user-attachments/assets/c5ba53e8-16aa-451d-ad76-752bbd93eafb" />
<img width="691" height="256" alt="image" src="https://github.com/user-attachments/assets/98ef0430-2023-409d-8a24-de944bf03267" />

---

### Assign Resources:

* EC2
* EBS
* EFS

### 📸 Screenshots:
<img width="691" height="307" alt="image" src="https://github.com/user-attachments/assets/dd33c281-c6a2-442e-a7dd-695a33433c1e" />

* Backup rules
* Resource assignments

---

## 🔹 STEP 9: Setup Storage Gateway (Hybrid)

### Navigate:

`AWS Console → Storage Gateway`

### Steps:

1. Create Gateway
2. Select **File Gateway**
3. Deploy as VM (on-prem simulation)
4. Configure:

   * NFS / SMB share
5. Link to:

   * S3 primary bucket

👉 Enables **On-Prem → AWS sync**

### 📸 Screenshots:
<img width="691" height="265" alt="image" src="https://github.com/user-attachments/assets/000f9a83-3159-4dee-841d-745149682672" />
<img width="692" height="287" alt="image" src="https://github.com/user-attachments/assets/1eb3b81a-e98e-49c6-8318-5e9f8b01c049" />
<img width="692" height="186" alt="image" src="https://github.com/user-attachments/assets/2ca315f2-96d6-45eb-8e22-d8ee39034813" />

---

## 🔹 STEP 10: Validate DR Architecture

### Test Scenarios:

* Upload file → S3 → Verify replication in DR bucket
* Create file in EFS → Check AWS Backup
* Stop EC2 → Restore using AMI
* Verify EBS snapshots

### 📸 Screenshots:
<img width="692" height="186" alt="image" src="https://github.com/user-attachments/assets/0c6d39b3-478f-4aab-badd-c71518077372" />

* Validation results
* 
<img width="691" height="245" alt="image" src="https://github.com/user-attachments/assets/cacc1283-d3a1-4956-b2e7-b6d449a5930e" />

---

## 🧩 Final Architecture Flow

```
EC2 (App Servers)
   ↓
EBS (Multi-Attach)
   ↓
EFS (Shared Storage)

   ↓
AWS Backup → Vault → Cross-region copy

On-Prem Data
   ↓
Storage Gateway
   ↓
S3 (Primary)
   ↓ (CRR)
S3 (DR Region)
```

---

## ✅ How This Meets Requirements

| Requirement       | Implementation              |
| ----------------- | --------------------------- |
| RPO = 1 hour      | Hourly EBS backups          |
| RTO = 4 hours     | AMI + cross-region recovery |
| Shared Storage    | Amazon EFS                  |
| Hybrid Backup     | Storage Gateway             |
| Disaster Recovery | S3 CRR + Backup copy        |

---

## 📌 Conclusion

This architecture provides:

* Fault tolerance
* Automated backups
* Cross-region disaster recovery
* Hybrid cloud integration

It ensures **minimal downtime and data loss**, making it suitable for enterprise-grade financial systems.

