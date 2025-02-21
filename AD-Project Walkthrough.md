---
AD-Project
---

#### **Overall Project Objectives:**
- Set up an **Active Directory** environment
- Deploy **Splunk** to monitor AD and Windows activities

#### **Steps:**
**Part 1:** Designing a Lab Diagram  
**Part 2:** Installing Virtual Machines (Windows & Linux)  
**Part 3:** Installing and Configuring Sysmon and Splunk  
**Part 4:** Configuring Active Directory  
**Part 5:** Generating Telemetry with Kali 

---

#### **Part 1**: Designing a Lab Diagram
The goal of this project is to set up an Active Directory (AD) lab to simulate attacks on AD users and capture the activity on our Splunk server for analysis. The lab will consist of the following components:
- **Windows Machines** (victims)
- **Kali Linux Machine** (attacker)
- **Windows Server** (Active Directory and Domain Controller)
- **Splunk Server** (to monitor and analyze the attacks)

![Diagram](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-12%20155412.png)

#### **Part 2:** Installing Virtual Machines (Windows & Linux)  

**Windows 10**
To install Windows 10, visit this [link](https://www.microsoft.com/en-ca/software-download/windows10) and click **Download Tool Now** to get the Media Creation Tool, which will generate a Windows ISO image. Once downloaded, open the tool, accept the agreement, select **Create installation media (ISO file)**, and proceed until you reach the **Choose which media to use** page, then select **ISO file**. After the ISO file is created, open VirtualBox, click **New**, enter a name (demo), select the ISO image, skip unattended installation, and configure the necessary system specifications before installation (The specs heavily depend on your systems capabilities).

**Kali Linux**
To install Kali Linux, visit this [link](https://www.kali.org/get-kali/#kali-virtual-machines), select the **VirtualBox 64-bit** option (or 32-bit if applicable), and click the download arrow. Since the file is in **7z** format, you'll need **7-Zip** installed; you can download it from [here](https://www.7-zip.org/). Once installed, open the Kali Linux file, proceed through the prompts, and navigate to the saved **7z** file. Right-click the file, select **7-Zip > Extract to kali-linux**, and once extraction is complete, you’ll see a **.vbox** file. Double-clicking it will automatically import Kali Linux into VirtualBox, setting up the virtual machine for use.

**Windows Server**
To install Windows Server, visit this [link](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022) and download the **64-bit ISO image** after filling out the required information. Once the download is complete, open VirtualBox, click **New**, enter a name (e.g., ADDC01), add the ISO image, and skip unattended installation. Finally, adjust the system specifications based on your hardware capabilities before proceeding with the installation.

**Splunk Server**
To install a Splunk Server, visit this [link](https://ubuntu.com/) and download the latest **Ubuntu Server** ISO. Once the download is complete, open VirtualBox, click **New**, enter a name (e.g., Splunk), add the ISO image, and skip unattended installation. Finally, adjust the system specifications based on your hardware capabilities before proceeding with the installation.

---

#### **Part 3:** Installing and Configuring Sysmon and Splunk  

**VirtualBox and Networking Issues**
Before setting up Splunk, make sure you're using the latest version of **Oracle VirtualBox**. I spent hours troubleshooting **NAT network issues**, only to discover that updating VirtualBox was the simple solution. To create a NAT network, navigate to the **Tools** icon, click **Create**, assign a name and an IPv4 prefix, then assign all devices to the newly created NAT network.
![NAT](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20114704.png)


