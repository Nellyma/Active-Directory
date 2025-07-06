# Active-Directory
## Objective
In this lab we will set up a virtual environment using Oracle VM VirtualBox with Windows server with Active Directory, a Splunk server, a Windows target machine, and a Kali Linux instance. We will arrange network configurations to enable communication via IP addresses and NAT Networks. We will also set up a Splunk instance to ingest events from both a Windows Server running Active Directory and a target Windows machine, Universal Forwarder for data forwarding, and Sysmon for endpoint monitoring. We will perform a brute force attack using a Kali Linux machine to observe the telemetry generated, and we will also use Atomic Red Team for additional testing.

![Screenshot 2025-07-06 160152](https://github.com/user-attachments/assets/d7423336-04a1-4e0c-a33d-40a0bb611af3)

Ref 1: Network Diagram
## Skills Learned

- Setting up VMs (Windows 10, Kali Linux, Windows Server, Ubuntu Server) in Oracle VM VirtualBox.
- Configuring IP addresses, NAT Networks for VMs.
- Troubleshooting network connectivity (ping, DNS settings).
- Installing Splunk Server and Universal Forwarder.
- Installing Sysmon for endpoint monitoring.
- Analyzing security logs (event codes 4625, 4624) in Splunk.
- Joining Windows machines to a domain.
- Enabling Remote Desktop on Windows.
- Using Hydra for brute force attacks.
- PowerShell scripting for tasks (Invoke-WebRequest, Set-ExecutionPolicy).
- Using Atomic Red Team (ART) to simulate tests.
  
These skills covered topics relating to Virtualization, Networking, Software Installation and Configuration, Security Tools and Practices, Operating System Configuration, Documentation and Reporting, Scripting and Automation, Troubleshooting, Active Directory Management, as well as Endpoint Monitoring and Defense.

## Tools Used

- Oracle VM VirtualBox Manager: For creating and managing virtual machines (VMs).
- Microsoft Windows 10: Operating system for target machine in the lab.
- Windows Server 2022: Operating system used for Active Directory Domain Services setup.
- Ubuntu Server: Used as a Splunk server in the lab setup.
- Kali Linux: Used as an attacker machine in the lab setup.
- Splunk Server: For log analysis and monitoring.
- Splunk Universal Forwarder: For data forwarding to Splunk.
- Sysmon: Endpoint monitoring on Windows machines.
- Microsoft Windows Event Logs: Analyzed in Splunk for security monitoring.
- Hydra: Used to simulate brute force attacks.
- PowerShell: For scripting and automation tasks.
- Atomic Red Team (ART): Used for security testing and validation.

## Steps
## Part 1 - VM Installation
### 1. Download and install Oracle VM VirtualBox Manager.
Go to the official VirtualBox website https://www.virtualbox.org/, download VirtualBox and install the dependencies.

### 2. Install Windows 10
Visit https://www.microsoft.com/en-ca/software-download/windows10, get "Create Windows 10 installation media", click "Download tool now", choose "Create installation media (USB flash drive, DVD, or ISO file) for another PC", then select "ISO file" and save it. In Oracle VM VirtualBox Manager, click "Add" to create a new VM, name it, choose the previously downloaded Windows 10 ISO, select 4096MB RAM, 1 CPU, 50GB virtual disk, and finish. Start the VM, follow the installation prompts, select "Custom: Install Windows only (advanced)", and let Windows 10 install.

### 3. Install Windows Server
Download Windows Server 2022 ISO from https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022, fill out the form, download "64-bit edition", create a new VM in Oracle VM VirtualBox Manager with the ISO, 4096MB RAM, 1 CPU, 50GB virtual disk, and finish. Start the VM, select "Install now", choose "Windows Server 2022 Standard Evaluation (Desktop Experience)", customize settings, create a password, and finish.

### 4. Install Kali Linux
Get Kali Linux from https://www.kali.org/, download the VM version, and also download and install WinRAR from https://www.win-rar.com/download.html?&L=0. Extract Kali Linux via WinRAR, import it into Oracle VM VirtualBox Manager, and start the VM.

### 5. Install Ubuntu Server
Navigate to https://ubuntu.com/server. In Products, Ubuntu Server, download Ubuntu Server, download Ubuntu Server. This lab uses version 22.04.4 LTS. Create a new VM in Oracle VM VirtualBox Manager with the ISO, 8192MB RAM, 2 CPUs, 100GB virtual disk, and finish. Start the VM, select "Try or Install Ubuntu Server", continue through a series of "Done" and "Continue", then fill out the form before continuing the installation. Finally, reboot. Error messages are expected. After rebooting, login and run sudo apt-get update && sudo apt-get upgrade -y. After this completes, hit "Enter".

![Screenshot 2025-06-23 115623](https://github.com/user-attachments/assets/04b5a709-9491-49ac-873d-30f2c262348a)

Ref 2: 
## Part 2 - Configure the Windows Machines and Install Splunk
### 1. Setup NAT Network

Click Tools > Network > NAT Networks > Create and name it AD Project and change IPv4 Prefix (in this lab, we will use 192.168.10.0/24), then select Apply. Navigate to each VM > Settings > Network, change "Attached to: NAT Network" and assign the name to the NAT Network named AD Project. Navigate to Splunk VM and type "sudo nano /etc/netplan/50.cloud.init.yaml". You should modify and save parameters to resemble this:

![Screenshot 2025-07-03 122543](https://github.com/user-attachments/assets/c7d2168d-cee5-4095-a036-2ed031da66eb)

Ref 3:
Then run "sudo netplan apply" to save changes. Now run "ip a", you should see the IP address set to 192.168.10.10/24. Check connectivity by running "ping google.com".

### 2. Install Splunk

Head over to https://www.splunk.com/ and create and account and download a free trial of Splunk Enterprise for Linux and choose the (.deb) file. Go back to Splunk VM and install the guest add-ons for virtual box by running "sudo apt-get install virtualbox-guest-additions-iso".

Now go to Devices > Shared Folders > Shared Folders Settings > Create new Shared Folder. Navigate to the folder where you installed Splunk, check all three boxes, and hit OK.

Reboot the virtual machine with "sudo reboot". Run sudo apt-get install virtualbox-guest-utils then reboot once more, and then "sudo adduser <your username> vboxsf". Create a new directory called "share" by running "mkdir share".

Now run "sudo mount -t vboxsf -o uid=1000,gid=1000 <directory name where .deb file is located> share/".

Verify completion with ls -la, "share" should be highlighted. Change directory into "share" using "cd share/" and run ls -la once more to view all the files listed in that directory. To install splunk, run "sudo dpkg -i splu<hit tab to auto-complete>"

Navigate via "cd /opt/splunk/" and run ls -la. Change into the user Splunk by running "sudo -u splunk bash". Change in bin directory, "cd bin/". Run "./splunk start", to get to the end, press space and type y.

To finalize this step, exit, cd bin, and finally, sudo ./splunk enable boot-start -user splunk. This will allow Splunk to start on boot as the user Splunk.

### 3. Install Splunk Universal Forwarder and Sysmon

In the Windows 10 VM, click the Start Menu search for PC > Properties > Rename this PC. Rename it to whatever you'd like, in this lab, I will name it "target-PC", click next. Restart the system. Open the Command Prompt run ipconfig and view the current IPv4 Address. Navigate to the network icon > Open Network & Internet Settings > Change adapter options > Right click the adapter > Properties > Internet Protocol Version 4 (TCP/IPv4) > Properties > Use the following IP address. Set IP Address to 192.168.10.100, Subnet mask to 255.255.255.0, Default gateway to 192.168.10.1, and lastly the Preferred DNS server to 8.8.8.8. Running ipconfig again should showcase the new IP address.

Keep Splunk server running, open a web browser on your target machine's browser and type 192.168.10.10:8000 in as the URL to access Splunk Server. In the same web browser, visit https://www.splunk.com/ and log in using the account you created before. Navigate to Products > Free Trials & Downloads > Universal Forwarder > Get my free download and download the correct version for your OS. Go to Downloads and double-click the downloaded MSI file, set up basic information but don't create a password. Skip deployment server, but for Receiving Indexer set the IP/port to 192.168.10.10:9997 and install.

Then we install Sysmon by opening https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon and click Download Sysmon. Next, navigate to https://github.com/olafhartong/sysmon-modular scroll down and select sysmonconfig.xml. Click on "raw", Right click and select Save As, and save the file in Downloads. Head over to Downloads and extract the sysmon file, copy URL of the extracted directory, and open Powershell as administrator, and navigate to that directory by runnung "cd <file path>". Run ".\Sysmon64.exe -i ..\sysmonconfig.xml", then click agree. Close it out by clicking on Finish.

Headover to to File Explorer> Local Disk (C:) > Program Files > SplunkUniversalForwarder > etc > system > local. Open Notepad as administrator and enter the following:

[WinEventLog://Application]

index = endpoint

disabled = false

[WinEventLog://Security]

index = endpoint

disabled = false

[WinEventLog://System]

index = endpoint

disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]

index = endpoint

disabled = false

renderXml = true

source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

Save this file to C:/Program files/SplunkUniversalForwarder/etc/system/local as inputs.conf and change Save as type to All Files.

Search Services and run as administrator, double click SplunkForwarder > Log On > Local System account > OK. Select SplunkFowarder > Restart > OK > Start.

Open a web browser and navigate to 192.168.10.10:8000 like before and login using the username and password created during Splunk installation. Navigate to Settings > Indexes > New Index > name it "endpoint" > Save. Click Settings > Forwarding & receiving > Configure receiving > New Receiving Port > Set port to 9997. Click Apps > Search & Reporting and search for "index=endpoint".

![Screenshot 2025-06-22 132805](https://github.com/user-attachments/assets/2a7566ab-b512-419a-9521-933f67511b83)

Ref 4:

On the Windows Server VM, Search PC > Right click > Properties > Rename this PC. Choose a name you like, I chose "ADDC01". Restart the system. Open the Command Prompt run ipconfig and view the current IPv4 Address. Navigate to the network icon > Open Network & Internet Settings > Change adapter options > Right click the adapter > Properties > Internet Protocol Version 4 (TCP/IPv4) > Properties > Use the following IP address. Set IP Address to 192.168.10.7, Subnet mask to 255.255.255.0, Default gateway to 192.168.10.1, and lastly the Preferred DNS server to 8.8.8.8. Running ipconfig again should showcase the new IP address.

Now you can follow the same steps to install Sysmon and Splunk Universal Forwarder on your Windows Server machine. Follow all the steps and use the same inputs.conf file posted above. When you access Splunk from the Windows Server VM web browser, you should have 2 hosts as show in the picture below.

![Screenshot 2025-06-22 133333](https://github.com/user-attachments/assets/fe03b156-6a4d-4128-8305-81d2db950717)

Ref 5: 

## Part 3 - Install and Configure Active Directory on Windows Server
### 1. Setup Active Directory Domain Services on the Windows Server

On your Windows Server VM, open up Server Manager. Navigate to Manage > Add Roles and Features. Select Next > Next, and check Active Directory Domain Services > Add Features. Advance until you can select install.

Once you receive the message "Configuration required. Installation succeeded on ADDC01", you can advance to the next steps. Locate the flag icon at the top right of the window > Promote this server to a domain controller > Add a new forest, creating a brand new domain. We will name it MYDFIR.local. On the next page, leave all defaults and create a password. Advance until the Configuration Wizard verifies prerequisites, and then hit install. This should trigger you to be signed out and restart the server.

When the machine is back on, you should see MYDFIR\Administrator as the user. In Server Manager, navigate to Tools > Active Directory Users and Computers. Right-click MYDFIR.local > New > Organizational Unit > Name: "IT". Right-click in the space of this new OU > New > User. First name: "Jenny", Last name: "Smith", Full name: "Jenny Smith", User logon name: "jsmith". Create a password, then finish. Create a new Organisational Unit called "HR" and create another user here, First name: "Terry", Last name: "Smith", Full name: "Terry Smith", User logon name: "tsmith" and create a password. Rememeber to uncheck "User must change password" for both users.

### 2. Join Windows 10 Machine to New Domain

On the Windows 10 machine, head over to network adapter, right click > Open network and Internet settings > Change adapter options > Right click adapter > Properites > Double click Inter Protocol Version 4(TCP/IPv4), change Prefered DNS server to 192.168.10.7 and select OK. 

Search PC > Properites > Advanced System Setting > Computer Name > Change > Select Domain and type in domain name of MYDFIR.LOCAL > Click OK and use the admin credentials of the Windows Server to log in. Click OK and click OK again to restart. Once the system is back up, click on "Other User" and log in with the jsmith account.

![Screenshot 2025-07-04 125914](https://github.com/user-attachments/assets/69793315-0866-4ecd-ba77-b586bd48290a)

Ref 6: 

## Part 4 - Use Kali Linux to Conduct a Brute Force Attack on Target Machine.

### 1. Configure Network
Power up the Kali Linux VM and log in using default credentials of kali/kali. From the desktop, right-click the connections icon at the top right of the screen > Edit Connections > Wired connection 1 > Clog icon > IPv4 Settings > set Method to "Manual" > Add > Address: 192.168.10.250 > Netmask: 24 > Gateway: 192.168.10.1 > DNS servers: 8.8.8.8 > Save. Click on the ethernet icon in the top right corner > Disconnect > click on ethernet icon again > Wired connection 1. Right click on desktop space > select Open terminal here and run ip a commad and you should see the changes be reflected here. You can verify connectivity by running ping google.com and/or 192.168.10.10 which is Splunk. Now run "sudo apt-get update && sudo apt-get upgrade -y". This should update and upgrade all necessary repositories.

### 2. Set up Attack
Create a new directory by running "mkdir ad-project". So change directory using cd /usr/share/wordlists. Then if "rockyou.txt.gz" is present by running ls, and then we unzip it using sudo gunzip rockyou.txt.gz. Now copy this file into ad-project folder using cp rockyou.txt ~/Desktop/ad-project. Now run cd ~/Desktop/ad-project to change in ad-project directory. So to show only the first 20 lines of rockyou.txt we run head -n 20 rockyou.txt. Then we output this list a file called passwords.txt by running head -n 20 rockyou.txt > passwords.txt. Run ls to verify presence of new file and we open new file with cat passwords.txt. to edit this file, we run nano passwords.txt and scroll down to the bottom and add the password we used for tsmith account. Then save and exit. Run cat passwords.txt and check if the new password has been successfully added.

### 3. Enable Remote Desktop
On our target machine, search "PC" > Properties > Advanced System Settings > Log in in with the administrator account > Remote > check Allow remote connections to this computer > Select users > Add > Enter the usernames we created earlier, "jsmith" and "tsmith" > Apply > OK

![Screenshot 2025-06-23 135307](https://github.com/user-attachments/assets/336cade9-6d04-44cc-bdd8-04fba26208b7)

Ref 7: 

### 4. Attack With Kali
Open the Kali machine and run hydra -h to get more information about the hydra tool. Run hydra -l tsmith -P passwords.txt -f 192.168.10.100 rdp. If a password listed in passwords.txt matches a password assigned to the user-specified, you will get a success message with the password attributed to the specified user.

![Screenshot 2025-06-23 143135](https://github.com/user-attachments/assets/51e7792f-ccce-4cb9-8400-68c0001ce189)

Ref 8: 

### 5. Analyze Attack With Splunk
We can head over to our Target machine and open Splunk on the web browser to view all activity of this endpoint using index=endpoint. Select Search and Reporting, type in "index = endpoint tsmith" and choose last 30mins. Scroll down and click on EventCode there are 3 events, and one stands out. Value 4625 stands out because it occurred 20 times. A quick Google search (https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4625) verifies that this is 60 counts of an account failing to log on. When investigating further, it can be noted that all of the counts are occurring at approximately the same time which is an indication of brute force activity. We also notice an event code 4624 which when is looked into (https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4624) we know is an indication of a successful logon.

![Screenshot 2025-06-23 202327](https://github.com/user-attachments/assets/b5c2e562-4f0d-47fe-a185-7c0961c6ced9)

Ref 9: 

### 6. Use Atomic Red Team (ART) to Attack Target Machine
Open Powershell as administrator. Login with administrator account. Run Set-ExecutionPolicy Bypass CurrentUser > Y. Click the up arrow located at the bottom right of your window > Windows Security > Virus & Threat Protection > Manage Settings > Add or Remove Exclusions > Add an exclusion > Folder > This PC > Local Disk (C:) and click select folder. Now navigate back to PowerShell and run IEX ( IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing); followed by Install-AtomicRedTeam -getAtomics and Y. 

Head over to C: drive and click on AtomicRedTeam > Atomics. Head to https://attack.mitre.org/ in order to view adversary attacks and techniques, as well as use it as a key for the "T" values. Run Invoke-AtomicTest T1136.001 to T1136 is a persistence, create local account test. After running this test and checking Splunk, we can see that no events popped up with the NewLocalUser that was created. This is excellent because we have just identified a gap in our protection. We can continue this with as many tests as we please. This makes Atomic Red Team very valuable for an organization.
