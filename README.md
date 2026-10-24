# aws-homelab

## Objective: 
  ### Built a secure, cost-effective AWS lab demonstrating VPC design, secure remote access through a bastion host, and Linux server hardening. The project strengthens networking fundamentals while showcasing practical networking, cloud infrastructure, and system administration skills.

## Architecture 
<img width="2108" height="1640" alt="IMG_0337" src="https://github.com/user-attachments/assets/33917280-dc8b-441d-adfb-5f90675aaf0f" />

| Component | Details |
| ------- | ------ |
| VPC | 10.0.0.0/16 |
| Public Subnet | 10.0.1.0/24 (bastion host) |
| Private Subnet | 10.0.2.0/24 (private server) |
| Internet Gateway | Attached to the VPC, used by <br> the public route table only |


## Skills Demonstrated

- VPC design: CIDR planning, public vs private subnets, routes tables, internet gateway
- Security groups: least-privilege rules, security-group-to-security-group references
- Secure remote access, SSH key authentication, bastion/jump host
- Linux administration: package updates, Nginx, SSH hardening
- Cloud cost management: budgets, alerts, stopping unused resources


## Cost Controls
<img width="1906" height="993" alt="Screenshot 2026-10-23 at 12 41 50 PM" src="https://github.com/user-attachments/assets/6fd326a4-eabb-41c0-bf59-edd2a4217b58" />


Before building anything, I set up a $10 monthly budget with an email alert. This helps to keep lab costs predictable and catch any mistakes early. I review the billing dashboard weekly and make sure to end my EC2 instances when not using them to not build any unnecessary costs.  


## Build Steps

### 1. Network (VPC)
  Created a VPC using the "VPC and more" wizard with one public subnet, one private subnet, one availability zone, and no NAT gateway. 
  <img width="1078" height="394" alt="VPCrestab" src="https://github.com/user-attachments/assets/f3ca2b27-c78a-4714-b76f-3aa9d743ba65" />

  **What I learned:** A public subnet has a direct route to the internet gateway. A private subnet does not.


### 2. Bastion Host (Public Subnet)
- Amazon Linux 2023, t3.micro
- SSH allowed from my IP only
### 3. Private Server (Private Subnet)
- Same image, no public IP
- SSH allowed only from the bastion's security group

**Security group rules**
| Resource | Type | Port | Source |
| ------ | ----- | ----- | ------ |
| Bastion | SSH | 22 | My IP | 
| Bastion | HTTP | 80 | 0.0.0.0/0 | 
| Private server | SSH | 22 | Bastion security group |

### 4. Connecting Through the Bastion 
Initially, I run a code to update the permissions of the pem key to be private/ read only. 

`chmod 400 ~/Download/bastionKey.pem` 

then i run a bash script that connects to the private server through the bastion server

`ssh -i ~/Downloads/bastionKey.pem ec2-user@BASTION_PUBLIC_IP ec2-user@PRIVATE_IP`

- **ssh:** opens a secure remote connection
- **-i ~/Downloads/bastionKey.pem:** identify the key pair file
- **-J:** jump host. Tells SSH to connect to the bastion first, then hop to the private server.
- **ec2-user@BASTION_PUBLIC_IP:** the bastion login.
- **ec2-user@PRIVATE_IP:** the private server login

<img width="1917" height="554" alt="Screenshot 2026-10-24 at 1 56 51 AM" src="https://github.com/user-attachments/assets/41efdf4a-d4bf-40ae-98e1-1b6470f09581" />

## Troubleshooting Log
| Problem | Cause | Fix |
| ------ | ------ | ----- |
| Permission denied <br>(publickey) | Key file <br>permissions too <br>open | Ran `chmod 400 ~/Downloads/bastionKey.pem |

