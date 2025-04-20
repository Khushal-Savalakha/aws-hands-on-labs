# 📦 How to Use EBS Snapshots in AWS

Amazon EBS (Elastic Block Store) provides persistent block storage for EC2 instances. Here's how you can understand and use **EBS volumes and snapshots** effectively.

---

## 📁 Understanding EBS Volumes

- **EBS Volumes** are used to store data in AWS.
- Two types of volumes:
  - **Root Volume**: Temporary; it stores the OS and is **mandatory** (cannot be removed).
  - **Additional Volumes**: Permanent; can be **added or removed** anytime.

---

## 🔁 What is an EBS Snapshot?

- A **snapshot** is like a **copy / backup / xerox** of a volume.
- Used to **back up EBS volumes**.
- You cannot directly attach a snapshot to an EC2 instance.
- Flow: `EBS Volume → EBS Snapshot → EBS Volume`

---

## ☁️ Key Points About Snapshots

- **Stored in**: AWS S3 (managed by AWS).
- **Snapshots are regional**, not tied to Availability Zones (AZs).
- **Cross-region copy** is possible within the same AWS account.
- Each snapshot corresponds to **one specific volume**.
- **You must create a volume from a snapshot** to use it with EC2.

![Image](https://github.com/user-attachments/assets/7a532551-c07e-4ea5-a7c3-3dff26d7da55)

![Image](https://github.com/user-attachments/assets/42743573-699d-4554-8379-17abc655c617)

![Image](https://github.com/user-attachments/assets/801fd2a3-f068-4d2f-bd77-0c67da337dd4)

![Image](https://github.com/user-attachments/assets/029fc371-f7b7-4983-be99-78769a025440)
---

## 🔗 Volume Attachment Logic

- You can attach **multiple volumes** to a **single EC2 instance**.
- If a volume type is **IOPS**, it can be shared across **multiple instances**.

---

## 🛠️ Practical Example

### Step-by-Step:

```bash
# Check attached volumes
lsblk

# Output will show volumes like /dev/xvda (Root), /dev/xvdf (Additional)
```

### Mount Additional Volume:

```bash
# Create a directory for mounting
cd /mnt
sudo mkdir backup

# Mount the volume
sudo mount /dev/xvdf /mnt/backup

# Check contents
cd /mnt/backup
ls -a
```

### Sample File in Volume:

```bash
cat welcome.sh
# Output:
# Sum : 10 + 10
# Sum is a=10 + b=10 is 20
# Variables are a and b which values are 10 and 10.
```

---

## 💡 Do You Need to Format the Volume?

Usually, **yes** — if it's a **new volume**:

```bash
sudo mkfs.ext4 /dev/xvdf
```

> This command formats the disk with the ext4 file system so it can store files.

But in **our case**, it's not needed because:

- We created a **new volume from an existing snapshot**.
- That snapshot was already formatted and had a valid file system.
- So we can **directly mount and use it** without formatting again.

---

## ✅ Summary

| Step | Action |
|------|--------|
| 1 | Create a volume |
| 2 | Format it using `mkfs.ext4` (if new) |
| 3 | Mount it to a directory (e.g., `/mnt/backup`) |
| 4 | Store data |
| 5 | Take a **snapshot** (backup) of the volume |
| 6 | Create a **new volume** from the snapshot |
| 7 | Attach new volume to EC2 |
| 8 | Mount it (No need to format again) and use it! |