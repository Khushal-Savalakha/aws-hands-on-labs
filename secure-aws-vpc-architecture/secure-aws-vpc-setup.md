# **Designing a Secure AWS VPC Architecture: Public and Private Subnets with Route Tables, Internet Gateway, Bastion Host, and NAT Gateway**

---

## 📘 **Overview**

In this blog, we will walk through the process of designing a **custom AWS VPC** that includes both **public and private subnets**, configured with **Internet Gateway**, **Route Tables**, **NAT Gateway**, and a **Bastion Host** for secure SSH access.

This tutorial is ideal for **cloud engineers, DevOps beginners**, or anyone wanting to understand secure networking in AWS.

---

## 📚 **Key Concepts Before We Start**

Before diving into implementation, let’s clarify some core AWS VPC components:

* **VPC (Virtual Private Cloud)**: Your private network within AWS.
* **Subnet**: A segment of the IP range within the VPC (can be public or private).
* **Internet Gateway (IGW)**: Allows internet access for resources in public subnets.
* **Route Table**: Defines how network traffic is routed.
* **Elastic IP**: A static IPv4 address reachable from the internet.
* **NAT Gateway**: Enables private subnet resources to access the internet **outbound only**.
* **Bastion Host**: A public EC2 instance used to securely connect to instances in a private subnet.

---

## 🛠️ **Step-by-Step VPC Setup**

---

### ✅ Step 1: Create a Custom VPC

Create a new VPC with CIDR block `10.0.0.0/16`.

> This CIDR block allows up to 65,536 IPs.

![Create VPC](https://github.com/user-attachments/assets/332b66a4-0099-437b-8eef-ac034be4f739)

---

### ✅ Step 2: VPC Creation Successful

Check the details of your new VPC.

![VPC Success](https://github.com/user-attachments/assets/0a2a7cf4-c0fe-4437-a221-a546fd82ed1f)

---

### ✅ Step 3: Create Two Subnets

We’ll divide the VPC using **two `/24` subnets**.

#### 📍 Subnet 1: `10.0.0.0/24` (Public Subnet - az-1a)

![my-subnet-az-1a](https://github.com/user-attachments/assets/fc4acd35-151e-4947-b1bd-fff3e7e734bf)

#### 📍 Subnet 2: `10.0.1.0/24` (Private Subnet - az-1b)

> AWS reserves 5 IPs per subnet. So, 251 usable IPs remain in each `/24` subnet.

![my-subnet-az-1b](https://github.com/user-attachments/assets/550ac527-2481-4d49-a706-a157fa1ba03c)

---

### ✅ Step 4: View Both Subnets

![List of Subnets](https://github.com/user-attachments/assets/45dce359-4d4b-40a5-aec7-7afc16fdb6f1)

---

### ✅ Step 5: Create and Attach Internet Gateway

1. Create an **Internet Gateway**.
2. Attach it to the custom VPC.

![Create IGW](https://github.com/user-attachments/assets/5d546c81-21cb-4756-b536-97256050176c)

![Attach IGW](https://github.com/user-attachments/assets/79cee6e2-0026-4ccc-822e-4021527b6fdf)

---

### ✅ Step 6: Create Two Route Tables

One for each subnet:

* **Public Route Table** → Attach Internet Gateway
* **Private Route Table** → For NAT Gateway later

![Route Tables](https://github.com/user-attachments/assets/bef2aa11-42c5-4561-83fe-bcea25861cd3)

![Attach IGW in Route Table](https://github.com/user-attachments/assets/ae4b2139-16d7-49d1-80b7-fdda34afd407)

![Attach Subnet to Route Table](https://github.com/user-attachments/assets/d661340b-100c-46d9-a50c-0aa0e0496981)

---

### ✅ Step 7: Create EC2 Instances

* **2 Instances** in **Public Subnet**
* **1 Instance** in **Private Subnet**

![Create EC2 Instances](https://github.com/user-attachments/assets/a9b9556c-e2d1-47df-b29f-cd1678e7d61f)

---

### ✅ Step 8: Enable DNS Hostnames for Public Access

> Without this, public DNS is not shown on EC2.

![Enable DNS](https://github.com/user-attachments/assets/6451cfe5-5675-4379-ad8c-4ff7053d9f77)

![DNS Visible](https://github.com/user-attachments/assets/4b52f7bc-ccc3-4625-9c7f-19a0f6e857f5)

---

### ✅ Step 9: Understand Network Boundaries

You **cannot access a private EC2 instance** directly. You must use a **Bastion Host** (a public instance) to SSH into the private one.

![Private EC2 Access](https://github.com/user-attachments/assets/2af6d059-8e22-480a-9dd3-74c8fe74d8fe)

![SSH via Bastion](https://github.com/user-attachments/assets/d42e4568-336a-4358-a9b0-98754673b861)

---

### ✅ Step 10: Secure EC2 with Security Groups

> Add **inbound SSH rule** to allow access **only from the public EC2 IP** (`10.0.0.45/32`).

![Add SSH Inbound Rule](https://github.com/user-attachments/assets/e728c5e8-aace-4adf-8146-bee8934f1d1d)

---

### ✅ Step 11: Verify Private EC2 Has No Internet Access

Use `ping google.com`. It will fail because **only public subnets have IGW**.

![Ping Failed](https://github.com/user-attachments/assets/4b7e1d3c-a93b-4ea4-8282-cea491563a6d)

---

### ✅ Step 12: Add NAT Gateway for Private Subnet Internet Access

> NAT Gateway must be created in a **public subnet** because it needs access to the Internet Gateway.

#### 📌 Steps:

1. **Create NAT Gateway** in `10.0.0.0/24`
2. Allocate **Elastic IP**
3. Add it to **Private Route Table**

![Create NAT Gateway](https://github.com/user-attachments/assets/b08caa15-5c15-40b3-be1b-ed023a3b9377)

![Elastic IP](https://github.com/user-attachments/assets/426c3103-6f4c-4ad7-8112-4f7b6cb26161)

![Private Route Table + NAT](https://github.com/user-attachments/assets/4a683086-a17e-40b6-9ea8-e81739e2de2a)

---

### ✅ Step 13: Confirm Internet Access from Private Subnet

Now `ping google.com` works from the private EC2 instance!

![Ping Success](https://github.com/user-attachments/assets/d5acfdff-94b3-427d-8cb5-4977e62958f9)

---

## ✅ **Conclusion**

You’ve now built a complete, secure, and functional AWS VPC architecture with:

* Public and Private Subnets
* Internet Gateway
* Route Tables
* Bastion Host
* NAT Gateway
* Security Group Restrictions
* Secure and Scalable Network Access
