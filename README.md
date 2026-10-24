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

### 5. Services and SSH Hardening
**Patching and installing a service**
- `sudo dnf update -y`: applied the latest security patches.
  <img width="1917" height="205" alt="Screenshot 2026-10-24 at 3 04 51 AM" src="https://github.com/user-attachments/assets/da0f8964-f907-49bd-ae8f-afe0874f101a" />

- `sudo dnf install nginx -y`: installed the Nginx web server
  <img width="1440" height="900" alt="Screenshot 2026-10-24 at 4 04 10 AM" src="https://github.com/user-attachments/assets/f1ddfbc6-3617-42e0-b387-f325009ee64e" />

- `sudo systemctl enable --now nginx`: started Nginx and set it to launch on boot.
  <img width="1917" height="205" alt="Screenshot 2026-10-24 at 3 06 38 AM" src="https://github.com/user-attachments/assets/b0f72913-02a0-4b7f-969f-d241f1958efd" />


**Allowing web traffic**
Added an inbound security group rule for HTTP (tcp 80) from my IP only, following least privilege.

**Hardening SSH**
- `sudo nano /etc/ssh/sshd_config`: opens the SSH configuration file in order to edit.
  <img width="1279" height="96" alt="Screenshot 2026-10-24 at 4 08 28 AM" src="https://github.com/user-attachments/assets/339f4bfc-231e-4e8b-9e5e-6c5cd54ae7f8" />

- `PermitRootLogin no`: blocks direct root login.
  <img width="1920" height="1080" alt="Screenshot 2026-10-24 at 3 34 27 AM" src="https://github.com/user-attachments/assets/6f276b4e-2861-42b2-ba7f-a537460de45e" />

- `PasswordAuthentication no`: allows key-based login only, which removes the risk of password guessing.
  <img width="1920" height="1080" alt="Screenshot 2026-10-24 at 3 34 22 AM" src="https://github.com/user-attachments/assets/5306b8f4-f7c5-4bc3-b75f-96e8da8b318f" />

**Safe change process**
Validated the file with `sudo sshd -t`, restarted with `sudo systemctl restart sshd`, and tested a new login in terminal before closing the original session.

## Troubleshooting Log
| Problem | Cause | Fix |
| ------ | ------ | ----- |
| Permission denied <br>(publickey) | Key file <br>permissions too <br>open | Ran `chmod 400 ~/Downloads/bastionKey.pem` |
| SSH timed out | Security group didn't allow my current IP | Updated the inbound security group rules to my IP |


## Lessons Learned
- Why private subnets matter and how traffic reaches them
- How security groups can reference each other instead of IP addresses
- Why stopping resources and setting budgets matters in the cloud.

## Next Steps
- [ ] Add Active Directory, DNS, and DHCP on Windows Server
- [ ] Add monitoring with Grafana or Zabbix
- [ ] Automate start/stop of instances with a Python (boto3) script
- [ ] Rebuild the VPC with Terraform

## Security Notes
No keys, passwords, or account IDs are stored in this repo. IP addresses and account details in screenshots are blurred.

