# SIEM - Azure Sentinel Lab

## Objective

In this lab, I will setup Azure Sentinel (SIEM) and connect it to a live virtual machine acting as a honey pot. We will observe live attacks (RDP Brute Force) from all around the world. We will use a custom PowerShell script to look up the attackers Geolocation information and plot it on the Azure Sentinel Map!

![Azure Sentinel RDP Attacks](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/956b2742-405c-4ddf-9a07-40cbbb0f072a)

## Skills Learned
- Setting up Azure Sentinel
- Creating a Honeypot Server
- Importing and using Powershell scripts

## Tools Used
- Azure Sentinel

## Step 1 - VM Setup (Win10 Honeypot Server)

logging in at https://portal.azure.com/#home i need to create a Virtual Machine

I'm setting up a basic Virtual machine with everything as standard the only part im changing is the 'Security type' to standard and the firewall setting within the setup to allow all ports. This will essentially allow all traffic from the internet to our virtual machine. We want this virtual machine to b open to anything RDP, SMB, pings, to get as much attacks as possible.
 
![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/7f2f57a5-b221-4fb0-9c33-907b483386b3)

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/e3421eb4-7561-4afd-b70e-d72cecfa41f1)

## Step 2 - Create Log Analytics Workspace

This needs to be setup to ingest logs from the virtual machine from Event Viewer. I will set up my own custom log that contains geographic data so we can plot on a map where the attacks are from.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/ea2a27f0-fd21-4dd3-bc19-a6f09f60bed1)

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/4f924588-cd06-4c39-a716-f72b12e4b039)

## Step 3 - Enable Defender for cloud

I want to enable Defender for cloud for this virtual machine, i'm doing this to allow the ability to gather the logs from the virtual machine to the log analytics workspace.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/a366b65e-3062-4078-b13a-61a79d4608f4)

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/2e940bcd-6e2f-4b80-9e14-eeabd75d1fe5)

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/245d9c14-63fb-4f55-8aae-3f61d471111f)

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/fc992495-9d13-4112-ac83-4ea3b3f33b8c)

## Step 4 - Connect logs to virtual machine

In this step were going back tot he Log Analytics Workspace and were going to connect it to the virtual machine.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/92938d9c-05ef-4f3a-a9af-51346128327b)

## Step 5 - Sentinel Setup

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/87e77383-51c3-4b04-a252-c52e68e0a1b6)

After clicking create we can see the server we created earlier so we will add Sentinel to this server

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/d014e3fa-9f95-41ea-af01-3a9cb00cf76b)

## Step 6 - Connect to the VM

In this step we will connect to our WIN10 server using RDP from my personal PC with the admin credentials we created earlier. I have failed 1 login on purpose using the wrong password just so we have a failed login event.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/4bf34c90-c6e2-4330-a251-e81127d331b4)













