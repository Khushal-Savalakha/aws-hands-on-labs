# **EBS Storage in EC2: A Step-by-Step Guide**

## Overview

Elastic Block Store (EBS) is a block-level storage service designed for use with Amazon EC2 instances. It provides persistent storage that remains intact even if the EC2 instance is stopped or terminated. This guide will walk you through the process of attaching an EBS volume to an EC2 instance, formatting it, and mounting it for use.

---

## **Table of Contents**

1. [Prerequisites](#prerequisites)
2. [List Attached EBS Volumes](#list-attached-ebs-volumes)
3. [Format EBS Volume](#format-ebs-volume)
4. [Mount EBS Volume](#mount-ebs-volume)
5. [Filesystem Selection](#filesystem-selection)
6. [Verify Data Availability](#verify-data-availability)
7. [Automating Mounting on Boot](#automating-mounting-on-boot)

---
![Image](https://github.com/user-attachments/assets/9c2169a4-0949-4137-9aef-43d90c234100)

![Image](https://github.com/user-attachments/assets/a07e75ea-d591-43b1-b30c-29582a091b00)

## **1. Prerequisites**

Before proceeding with the steps below, ensure that you have:

- An **EC2 instance** running in your AWS environment.
- An **EBS volume** attached to your EC2 instance.

To verify that your volume is attached, you can check the AWS EC2 console or use the following command within your EC2 instance to see the attached block devices.

```bash
lsblk
```

---

## **2. List Attached EBS Volumes**

Once you have logged into your EC2 instance, use the `lsblk` command to list all attached block devices.

```bash
lsblk
```

This will display all attached volumes along with their device names (e.g., `/dev/xvda`, `/dev/xvdf`, etc.). Identify the newly attached volume, such as `/dev/xvdf` in this example.

---

## **3. Format EBS Volume**

### **Check if the volume has any data**

Before formatting the volume, check whether it contains any data using the following command:

```bash
sudo file -s /dev/xvdf
```

If the output is `data`, it indicates that the volume is unformatted.

### **Format the volume**

Next, format the volume using the following command:

```bash
sudo mke2fs -j /dev/xvdf
```

This creates an ext3 file system on the volume. The `-j` option enables journaling, which improves the reliability and integrity of the data stored on the volume.

---

## **4. Mount EBS Volume**

### **Create a Mount Directory**

You need to create a directory where the volume will be mounted. Let's create a directory named `/newdisk`:

```bash
sudo mkdir /newdisk
```

### **Mount the Volume**

Once the directory is created, mount the EBS volume to it:

```bash
sudo mount /dev/xvdf /newdisk
```

### **Verify the Mount**

To verify that the volume is successfully mounted, run:

```bash
df -h
```

You should see `/dev/xvdf` listed under the mounted volumes.

---

## **5. Filesystem Selection**

When formatting a new EBS volume, you have several filesystem options, each with its own characteristics and use cases. The most commonly used filesystems for EBS volumes are **ext2**, **ext3**, and **ext4**. Let's explore them:

### **Difference Between ext2, ext3, and ext4**

| Feature               | ext2              | ext3                | ext4                     |
|-----------------------|-------------------|---------------------|--------------------------|
| **Journaling**         | No                | Yes (metadata & data)| Yes (faster, better journaling) |
| **Performance**        | Fast (no journaling) | Moderate (journaling overhead) | Fast and efficient        |
| **Max File Size**      | 2 TB              | 2 TB                | 16 TB                    |
| **Max Volume Size**    | 32 TB             | 32 TB               | 1 EB (Exabyte)           |
| **Recovery Time**      | Slow after crash  | Fast due to journal | Very fast                |
| **Use Case**           | USB drives, legacy systems | Older Linux systems | Modern Linux systems (default) |

### **How to Choose the Right Filesystem**

- **`ext2`**: Ideal for non-critical disks or where performance is more important than data safety (e.g., USB drives or temporary storage).
- **`ext3`**: Good for systems where you need data integrity and fast recovery after a crash but don't need the latest features.
- **`ext4`**: Recommended for modern EC2 instances as it is fast, reliable, and supports large volumes and files. It is the default choice for most Linux systems.

### **Creating Different Filesystems**

To create a different filesystem, you can use the following commands:

- **ext2**:
  
  ```bash
  sudo mkfs.ext2 /dev/xvdf
  ```

- **ext3**:
  
  ```bash
  sudo mkfs.ext3 /dev/xvdf
  ```

- **ext4**:
  
  ```bash
  sudo mkfs.ext4 /dev/xvdf
  ```

For most use cases in EC2, **ext4** is recommended due to its superior performance and data integrity features.

---

## **6. Verify Data Availability**

After mounting the EBS volume, check if data is available in the mounted directory:

```bash
ls -l /newdisk
```

You should see the contents of the EBS volume if any data has been stored there.

---

## **7. Automating Mounting on Boot**

To ensure that your EBS volume is automatically mounted after a system reboot, add the volume to the `/etc/fstab` file.

1. **Edit `/etc/fstab`**:

   ```bash
   sudo nano /etc/fstab
   ```

2. **Add the following line** (replace `/dev/xvdf` with your actual device name and `/newdisk` with your mount point):

   ```bash
   /dev/xvdf /newdisk ext4 defaults,nofail 0 0
   ```

3. **Save and exit**. This will ensure that your EBS volume is mounted automatically on startup.

---

## **Summary**

This guide has walked you through the process of attaching, formatting, and mounting an EBS volume to your EC2 instance. It also covered filesystem choices, with **ext4** being the recommended option for most modern EC2 workloads. By following these steps, you now have a persistent storage volume that can be used with your EC2 instance.


