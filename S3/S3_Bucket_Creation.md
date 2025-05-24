

## 🔐 Amazon S3 WORM (Write Once, Read Many) with Object Lock

### 📌 What is WORM?

**WORM** stands for **Write Once, Read Many** — a data protection model that prevents any modification or deletion of data for a fixed retention period. AWS S3 supports WORM through the **Object Lock** feature.

> 🔒 Commonly used for compliance with financial and healthcare regulations (e.g., SEC Rule 17a-4(f), FINRA, HIPAA).

----------

### 🎯 Why Use WORM?

-   ✅ Guarantee **immutability** of mission-critical records.
    
-   🛡️ Prevent **accidental or malicious data deletion or updates**.
    
-   📁 Support **legal and regulatory data retention** mandates.
    
-   📊 Enable tamper-proof **audit trails** and data archiving.
    

----------

### 🔧 Object Lock Modes

| Mode        | Description                                                                 |
|-------------|-----------------------------------------------------------------------------|
| Governance  | Write/delete protection is enforced, but **authorized users** can override. |
| Compliance  | Absolute protection — **no one**, including root, can delete/alter the data. |


> 📌 **Versioning must be enabled** on the bucket to use Object Lock.

----------

## 🛠️ Step-by-Step Guide: Create an S3 Bucket with WORM + Transfer Acceleration

### 🔹 Step 1: General Configuration

Set bucket name, AWS region, and availability zone options.

![General Configuration](https://github.com/user-attachments/assets/32dd858a-99e2-43dc-9a53-4fdc2d26378b)

----------

### 🔹 Step 2: Set Object Ownership

Leave default or customize object ownership as needed.

![Object Ownership](https://github.com/user-attachments/assets/508a683f-52de-49f6-a7fe-921dcff50fe4)

----------

### 🔹 Step 3: Set Public Access & Enable Versioning

-   **Block all public access** (or customize based on your needs).
    
-   **Enable versioning** – required for Object Lock.
    

![Public Access & Versioning](https://github.com/user-attachments/assets/4e009803-d483-48a7-b2b4-800eacedec93)

----------

### 🔹 Step 4: Enable Object Lock

Scroll to the "Advanced settings" and **enable Object Lock**.

![Encryption & Object Lock](https://github.com/user-attachments/assets/435cca52-7eae-4bf7-9d05-74d51d45c007)

----------

### 🔹 Step 5: Create Bucket

Click **Create bucket**. Your new WORM-enabled bucket is now ready.

![Bucket Created](https://github.com/user-attachments/assets/358b0ee0-1137-4f21-a8a1-7e1a109e4245)

----------

## 📤 Upload and Lock an Object

### 🔹 Step 6: Upload a File

Upload an image or folder into your S3 bucket.

![Upload File](https://github.com/user-attachments/assets/a5f22d13-df52-47ee-9344-284334f4fde6)

----------

### 🔹 Step 7: Choose Storage Class

Select a storage class (e.g., Standard, Intelligent-Tiering, Glacier).

![Choose Storage Class](https://github.com/user-attachments/assets/b91db847-65fb-47d3-943f-eadaae8b64b3)

----------

### 🔹 Step 8: Set Object Lock Settings

Choose the **Object Lock mode** (`Governance` or `Compliance`) and define a **retention period** (e.g., 1 year).

----------

### 🔹 Step 9: Confirm Upload

Once the file is uploaded, it becomes read-only and tamper-proof per your lock settings.

![Image Uploaded](https://github.com/user-attachments/assets/7197f721-7a28-4e13-88be-ccd9fce13bd2)

![Preview Uploaded File](https://github.com/user-attachments/assets/9086c3dc-a317-4b39-9dd5-e11b70ee1f25)

----------

## ⚡️ Enable Transfer Acceleration

To speed up uploads/downloads globally using Amazon CloudFront edge locations:

### 🔹 Step 10: Activate Transfer Acceleration

1.  Go to **Bucket → Properties**.
    
2.  Scroll to **Transfer acceleration**.
    
3.  Enable it and save changes.
    

![Transfer Acceleration](https://github.com/user-attachments/assets/869a9bba-5a97-456b-b63b-8d4282e781ed)

> 🧠 Transfer acceleration works best when uploading large objects from geographically distant locations.

----------

## 🏁 Summary

```mermaid
graph TD
    A[Feature Summary]

    A --> B[Object Lock]
    B --> B1[Enables WORM protection<br>to ensure data immutability]

    A --> C[Versioning]
    C --> C1[Prerequisite for Object Lock<br>Retains object history]

    A --> D[Compliance Mode]
    D --> D1[Strict data retention<br>No deletions/modifications during retention]

    A --> E[Transfer Acceleration]
    E --> E1[Improves data transfer speeds<br>using CloudFront edge locations]

```
