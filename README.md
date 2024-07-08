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

Once loged on i'm going to Windows Event Viewer under Windows Logs > Security to look for the failed login. More specifically im looking at Event ID 4625.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/0e94138d-23c4-484b-843b-6856173c9f3a)

Id i open up the Event you can see the reason for failure and also we can see the IP Address which we are going to use later. What we will be doing with the IP Adress is get the IP adress with Powershell and then use a IP Geolocation API to give us a Longitude, latitude, city etc 

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/e3aa4be8-d393-4293-a0cf-6e0486927325)

We will use: https://ipgeolocation.io/ to get the information we need.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/78d1c1f1-5f06-49e1-ba00-e1bc4e975f3a)

Next on the WIN10 machine i need to turn off the Windows Firewall as right now i cant even ping the machine from my local PC and we want people to find this machine.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/882b5680-9224-4dc1-83c0-10a4f4c01589)

I have disabled the firewall on both the 'Domain Profile' tab, 'Private Profile' and the 'Public Profile' tab

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/29320ea9-ba61-498d-ab37-b578af2708ce)

## Step 7 - Geolocation API > Powershell

Im going to copy the attched 'Custom_Security_Log_Exporter.ps1' script into Powershell ISE on the virtual WIN10 machine adding in the API i created from 'https://ipgeolocation.io/' and save it to the desktop as 'Log_Exporter.ps1'

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/5b20f67b-d01a-4b78-a6d8-de21579b31e7)

What this script will do is contantly looks through the event log grabs all the failed login details like IP address and then runs it throught the Geolocate api and outputs this data to a new log file located in 'C:\ProgramData\$($LOGFILE_NAME)' This script needs to be running contantly to allow this to happen. This script has sample data that will be output to the log that will alow me to train Log Analytics Workspace to accept and parse out the date we want to our log.

Lets run the script.

We can see in purple the EventID 4625 failed login events.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/17647a0b-e0cf-491c-8b0b-68bca74a6338)

Opening up the log file we can see the sample data plus the failed logins from myself with all the extras data i mentioned earlier like Longitude and Latitude, City etc.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/6a3cd009-da20-4a98-83d7-3d4ea5972723)

## Step 8 - Azure Custom Log

In this step i will bring in this cutom log into the Log analytics Workspace, we need to go to our workspace and select our workspace we created 'LAW_Honeypot1' and then under settings select

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/97c4489d-0180-42a2-97cb-0412d6952a1e)

The first this it will ask for a sample log. Here will add a copy of the contents of the new log file we created, were copying this log as its on our virtual machine not on our host computer. This is going to be used to train the log analytics to choose the corect fields for the logs.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/465f9ea0-ea08-4c1d-bc40-9b8257d6a2c5)

After importing we need to tell it where the logs are on our Virtual Machine. then give this a name, im naming mine 'Failed_RDP_GEO'

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/ace26c5a-b9ec-4200-acc7-9ddd7a2054dc)

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/dc32c08d-9c0e-4746-abd2-9df0ffd8ad2b)

Within the logs we need to wait for the custom rule to show up, im to make a coffee then will check back in :)

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/852deb52-519e-4639-bd86-470c509bfe23)

Were back and we have logs.

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/dad1f507-897d-4c9f-8e23-7f01e1320239)

## Step 9 - Extract the data

In this step i need to extract the data from the above log from within the 'RawData' field to create my own fields with the data sperated. To do this i need to create a custom KQL Query 

```
Failed_RDP_GEO_CL
| parse RawData with * "latitude:" Latitude ",longitude:" Longitude ",destinationhost:" Destinationhost ",username:" Username ",sourcehost:" Sourcehost ",state:" State ",country:" Country ",label:" Label ",timestamp:" Timestamp
| project
    Latitude,Longitude,Destinationhost,Username,Sourcehost,State,Country,Label,Timestamp
```

This custom query parses out the fields i want to see and we can save this query.

## Step 10 - Sentinel Map

In this step i need to open Sentinel and select 'Workbooks' from the menu then 'Add Workbook'

![image](https://github.com/Matt4llan/SIEM-Azure-Sentinel/assets/156334555/2cdbfb1a-1535-400e-9424-eaff70a1e651)

Now were going to click 'Add' then were going to 'Add Query' here again were going to use a custom KQL query to give us the data we need and add in 'summarize event_count=count() by' so we can use the 'Event Count' to Change the size of the dots on the map to show attack count size.

```
Failed_RDP_GEO_CL
| parse RawData with * "latitude:" Latitude ",longitude:" Longitude ",destinationhost:" Destinationhost ",username:" Username ",sourcehost:" Sourcehost ",state:" State ",country:" Country ",label:" Label ",timestamp:" Timestamp
| project
    Sourcehost,Latitude,Longitude,Country,Label,Destinationhost
| summarize event_count=count() by Sourcehost,Latitude,Longitude,Country,Label,Destinationhost
```

















