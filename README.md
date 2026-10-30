# AWS VPC Peering & Private Routing Project

## 📌 Project Overview
This project demonstrates how to securely connect two Amazon Virtual Private Clouds (VPCs) using **AWS VPC Peering**. The goal of this lab is to establish private, internal network connectivity between two isolated workloads while restricting inbound internet access and managing administrative access via a public Bastion host.

This project was completed as part of the EWB Cloud Labs (AWS Architect Associate track).

## 🏗️ Architecture & Network Design
The infrastructure spans two custom VPCs with non-overlapping IPv4 CIDR blocks.

*   **VPC A (10.10.0.0/16)**
    *   **Public Subnet (`ewb-a-public`):** 10.10.1.0/24 (Contains the Bastion host with Internet access via an Internet Gateway)
    *   **Private Subnet (`ewb-a-private`):** 10.10.2.0/24 (Contains App-A, isolated from the public internet)
*   **VPC B (10.20.0.0/16)**
    *   **Private Subnet (`ewb-b-private`):** 10.20.1.0/24 (Contains App-B, strictly private, no Internet Gateway)

### Resource Inventory
| Resource Name | Type | IP Address / CIDR |
| :--- | :--- | :--- |
| `ewb-vpc-a` | VPC | 10.10.0.0/16 |
| `ewb-vpc-b` | VPC | 10.20.0.0/16 |
| `ewb-bastion` | EC2 (t3.micro) | 10.10.1.10 (Public IP attached) |
| `ewb-app-a` | EC2 (t3.micro) | 10.10.2.10 (Private only) |
| `ewb-app-b` | EC2 (t3.micro) | 10.20.1.10 (Private only) |
| `ewb-a-b-peer`| VPC Peering | N/A |

## 🚀 Key Technologies & Concepts Demonstrated
*   **VPC Peering:** Routing traffic between two VPCs using private IPv4 addresses.
*   **Subnet Routing:** Configuring custom Route Tables to direct local, internet (IGW), and peering (PCX) traffic.
*   **Security Groups:** Implementing stateful firewall rules to allow strict ICMP (Ping) and SSH (TCP 22) traffic from specific private source IPs.
*   **Bastion Host & ProxyJump:** Securing administrative SSH access to private instances without exposing their key pairs or placing them in public subnets.

## ⚙️ Configuration Highlights
1.  **Routing:** 
    *   VPC A's private route table explicitly routes `10.20.0.0/16` to the Peering Connection.
    *   VPC B's private route table explicitly routes `10.10.0.0/16` to the Peering Connection for the return path.
2.  **Security Rules:**
    *   App-A and App-B only accept SSH (`port 22`) from the Bastion's private IP (`10.10.1.10/32`).
    *   ICMP Echo requests are strictly allowed only between App-A (`10.10.2.10/32`) and App-B (`10.20.1.10/32`).

## 🧪 Validation & Testing
To prove the architecture works as intended, the following tests were conducted:
*   **Baseline Test:** ICMP ping from App-A to App-B failed *before* VPC Peering and routes were configured.
*   **Reachability Test:** Successful 0% packet loss ping from App-A (VPC A) to App-B (VPC B) and vice-versa *after* peering.
*   **Secure Administration:** Successful SSH access into App-B (VPC B) directly from a local machine by routing through the Bastion host using `ProxyJump`.
*   **Controlled Failure:** Temporarily removing the ICMP Security Group rule on App-B immediately halted ping replies, confirming the strict firewall boundaries.

## 🧹 Cleanup
All EC2 instances, EBS volumes, VPC Peering connections, custom route tables, Internet Gateways, and VPCs were terminated and deleted after lab completion to prevent ongoing AWS billing charges.
