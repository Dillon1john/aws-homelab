# Phase 2: Active Directory and DNS on Windows Server

[Back to main README](../README.md)

**Status:** In progress

## Goal
Add a Windows Server domain controller to the private subnet to practice Active Directory, DNS, and common service desk tasks, reaching it securely through the existing bastion host

## Design Decisions
- **Self-managed AD on EC2** instead of AWS Managed Microsoft AD, to keep costs low and to learn how AD is installed and configured.
- **Private subnet, no public IP** the domain controller is only reachable through the bastion.
- **DHCP not configured** AWS provides DHCP for the VPC, so running my own DHCP server isn't practical here.

## Build Steps

### 1. Launch the Windows server
<img width="440" height="629" alt="Screenshot 2026-11-09 at 11 11 28 PM" src="https://github.com/user-attachments/assets/7743cf3f-72e1-4963-8196-3098cce90a5d" />

- **Name:** 'dc01'
- **AMI:** Windows Server 2022 Base
- **Instance type:** t3.medium
- **Subnet:** private, no public IP
- **Security group:** 'dc-sg'

| Resource | Type | Port | Source |
| ------ | ----- | ----- | ------ |
| dc01 | RDP | 3389 | Bastion security group |
<img width="1440" height="851" alt="Screenshot 2026-11-09 at 10 58 40 PM" src="https://github.com/user-attachments/assets/332d9929-6d5f-4f24-9783-b637f17aed9b" />

### 2. Retrieve the Administrator password
Decrypted the Windows Administrator password in the EC2 console using my key pair. The password is stored privately and is not included in this repo. 
<img width="1413" height="773" alt="Screenshot 2026-11-09 at 11 44 19 PM" src="https://github.com/user-attachments/assets/1d4c758b-38ed-4c1d-a10b-d09a643c3795" />

### 3. Connect with RDP through the bastion
RDP can't reach the private subnet directly , so I tunneled it through SSH:
`ssh -i ~/Downloads/bastionKey.pem -L 3389:DC_PRIVATE_IP:3389 ec2-user@BASTION_PUBLIC_IP` 
- **-L 3389:DC_PRIVATE_IP:3389:** forwards my local port 3389 through the bastion to the domain controller's RDP port
- Then connect a Remote Desktop client to 'loclhost:3389'
   <img width="1413" height="349" alt="Screenshot 2026-11-09 at 11 58 30 PM" src="https://github.com/user-attachments/assets/b1a3160a-8dcf-45c8-8639-654dfb464260" />


  
