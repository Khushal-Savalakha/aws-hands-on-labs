# VPC Peering Connection Setup Guide

This guide demonstrates how to establish a VPC peering connection between two VPCs in different AWS regions (N.Virginia and Singapore) and enable communication between EC2 instances.

## Overview

VPC peering allows you to connect two VPCs privately, enabling resources in different VPCs to communicate with each other as if they were within the same network. This setup uses private IP addresses and doesn't require internet gateways or VPN connections.

![VPC1 Sending request to VPC2](https://github.com/user-attachments/assets/d2108fb1-6717-47d0-9c81-4ce7734e0c82)

## Architecture

Our setup involves:
- **VPC 1**: Located in N.Virginia region (10.77.0.0/16)
- **VPC 2**: Located in Singapore region (172.31.0.0/16)
- **VPC Peering Connection**: Connecting both VPCs
- **EC2 Instances**: One in each VPC for testing connectivity

![Diagram of our 2 vpc Virginia and Singapore connected via VPC peering](https://github.com/user-attachments/assets/e5386faf-7dc7-4dd2-bc7a-f166196ea1bd)

## Prerequisites

- AWS account with appropriate permissions
- Access to both N.Virginia and Singapore regions
- Basic understanding of VPC concepts

## Step-by-Step Implementation

### Step 1: Create VPC in N.Virginia

Create the first VPC in the N.Virginia region with CIDR block 10.77.0.0/16.

![Create VPC in N.Virginia](https://github.com/user-attachments/assets/b021ed29-c3ec-473e-b23a-0a5f08c859b8)

### Step 2: Create VPC in Singapore

Create the second VPC in the Singapore region with CIDR block 172.31.0.0/16.

![Create VPC in Singapore](https://github.com/user-attachments/assets/91efe8f7-8339-4abc-a935-c1dd7bea8203)

### Step 3: Launch EC2 Instances

#### Create EC2 Instance in N.Virginia

Launch an EC2 instance in the N.Virginia VPC for testing connectivity.

![Create EC2 Instance in N.Virginia](https://github.com/user-attachments/assets/601e814d-9fe7-4b0c-8b23-15c1811bee60)

#### Create EC2 Instance in Singapore

Launch an EC2 instance in the Singapore VPC for testing connectivity.

![Create EC2 instance in Singapore](https://github.com/user-attachments/assets/4d0c412f-09cd-46e1-8060-b5dda2d57e7d)

### Step 4: Create VPC Peering Connection

#### 4.1 Send Peering Request

From the N.Virginia region, create a VPC peering connection named "N.Virginia-Singapore".

![Create Peering Connection named N.Virginia-Singapore](https://github.com/user-attachments/assets/642c88a5-cc7c-4431-8b2f-4a9e8f9547e9)

#### 4.2 Request Sent Successfully

The peering request has been successfully sent and is now pending acceptance.

![Successfully sent request of VPC peering](https://github.com/user-attachments/assets/36183891-fafc-4b9a-8d4d-854f341d4c7c)

#### 4.3 Pending Acceptance Status

The peering connection shows as "Pending acceptance" status.

![Currently we can see that Pending acceptance of VPC peering](https://github.com/user-attachments/assets/2e42f2dc-371c-4f00-a9f4-b2977cf3bad5)

### Step 5: Accept the Peering Request

#### 5.1 Navigate to Singapore Region

Switch to the Singapore region and navigate to VPC > Peering Connections. Select the pending request and click "Actions" > "Accept Request".

![In Singapore VPC's Peering Connection select peering request and go to Action and click on Accept Request](https://github.com/user-attachments/assets/217bbd49-9a49-45ca-9779-374004366607)

#### 5.2 Accept the Request

Confirm the acceptance in the toggle box.

![Accept Request in Toggle Box](https://github.com/user-attachments/assets/39bf965e-8e84-4f3f-9190-6aea1ae6c637)

#### 5.3 Peering Connection Active

The VPC peering connection is now active and ready for use.

![VPC peering is Active Now](https://github.com/user-attachments/assets/1e90f7ec-162e-4acd-aac2-8bcaf744f786)

### Step 6: Configure Security Groups

Update the security group for the Singapore EC2 instance to allow ICMP traffic from the N.Virginia EC2 instance.

![Change inbound rule of Singapore EC2 instance to send ICMP request from N.Virginia EC2 Instance](https://github.com/user-attachments/assets/b5c3c634-ac56-4d4f-b98d-dcdb768d8a4f)

### Step 7: Test Connectivity (Before Route Configuration)

At this point, ping requests will fail because routing is not configured yet.

![In terminal you can see we are not able to send ping](https://github.com/user-attachments/assets/0fa78cf8-50ad-4cbe-b15c-e69e19aab71f)

### Step 8: Configure Route Tables

#### 8.1 Add Route in N.Virginia Route Table

Add a route for Singapore VPC CIDR (172.31.0.0/16) pointing to the VPC peering connection.

![Add 172.31.0.0/16 in Route table of N.Virginia](https://github.com/user-attachments/assets/ab6a4ebc-bbc3-4756-9178-73305941bbbe)

Verify the route is added successfully.

![See 172.31.0.0/16 in Routes](https://github.com/user-attachments/assets/4aad216d-adb0-4668-a8fa-f3c411b8c9bf)

#### 8.2 Add Route in Singapore Route Table

Add a route for N.Virginia VPC CIDR (10.77.0.0/16) pointing to the VPC peering connection.

![Add 10.77.0.0/16 in Route table of Singapore](https://github.com/user-attachments/assets/4166a34e-07a8-454a-a234-9d6b5a27d7be)

Verify the route is added successfully.

![See 10.77.0.0/16 in Routes](https://github.com/user-attachments/assets/ca04abb7-9f1b-496b-bad7-733fc0c9f08f)

### Step 9: Test Connectivity (After Route Configuration)

Now the ping from N.Virginia to Singapore should work successfully.

![Now we can see our ping from N.Virginia is successfully sent to Singapore](https://github.com/user-attachments/assets/f2e13bc6-1d7a-4c6c-9bf9-3c2cbdf69ff1)

## VPC Peering Process Summary

### Step 1: Sending Request For Peering
- The **owner of the requester VPC sends** a request to the owner of the accepter VPC to create the VPC peering connection.
- The accepter **VPC can be owned by you**, or another AWS account.

### Step 2: Accept the Connection
- The owner of the accepter VPC accepts the VPC peering connection request to activate the VPC peering.

### Step 3: Setup Routing
- To enable the flow of traffic between the VPCs using private IP addresses, **the owner of each VPC in the VPC peering connection must manually add a route to one or more of their VPC route tables that points to the IP address range of the other VPC.**

## Important Notes

1. **CIDR Blocks**: Ensure VPC CIDR blocks don't overlap
2. **Security Groups**: Configure appropriate inbound/outbound rules
3. **Route Tables**: Both VPCs need routes pointing to each other's CIDR blocks
4. **DNS Resolution**: Consider enabling DNS resolution for the peering connection if needed
5. **Cross-Region**: This setup works across different AWS regions

## Troubleshooting

- **Ping fails**: Check security groups, NACLs, and route table entries
- **DNS not resolving**: Enable DNS resolution in peering connection settings
- **Partial connectivity**: Verify all subnets have proper route table associations

## Best Practices

1. Use descriptive names for peering connections
2. Document IP ranges and routing configurations
3. Regularly review security group rules

## Conclusion

VPC peering provides a secure and efficient way to connect VPCs across regions or accounts. By following this guide, you can establish reliable communication between resources in different VPCs while maintaining network security and performance.
