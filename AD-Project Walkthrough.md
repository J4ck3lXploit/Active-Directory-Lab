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

---

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

#### **Part 3:** Installing and Configuring Splunk and Sysmon  

**VirtualBox and Networking Issues**
Before setting up Splunk, make sure you're using the latest version of **Oracle VirtualBox**. I spent hours troubleshooting **NAT network issues**, only to discover that updating VirtualBox was the simple solution. To create a NAT network, navigate to the **Tools** icon, click **Create**, assign a name and an IPv4 prefix, then assign all devices to the newly created NAT network.
![NAT](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20114704.png)

Once we've assigned the correct network adapter type (ActiveDirectory which is our NAT network name) to our VMs, we can boot up the **Splunk server** and start configuring it. As shown in the image, the server is assigned a **random IP address** within the NAT subnet we created. However, we need to assign it a **static IP** that matches our network diagram.
![random IP](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-13%20190722.png)

**Configuring a Static IP for Splunk**

To change the assigned IP, navigate to:  
`/etc/netplan/*.yaml` and modify the file as follows:
- Assign the **Splunk server** the IP `192.168.10.10`.
- Set **Google's DNS** to `8.8.8.8`.
- Define the **default route and gateway**.

![static IP](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-13%20191738.png)
![ping](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-13%20191718.png)

**Downloading and Installing Splunk**

Once the network is properly configured, we can contunite to **downloading Splunk**:
- Go to this [link](https://www.splunk.com/), register an account, and download the `.deb` package.

However, before we install Splunk, we need to **enable file sharing** between the **host and VM**.

**Setting Up Shared Folders in VirtualBox**

To enable file sharing, we need to install the following packages on the **Splunk VM**:

`sudo apt-get install virtualbox-guest-additions-iso virtualbox-guest-utils`

These packages allow **shared folder access** and let us add users to different groups.

After installing the virtual machines, **add a shared folder** from the host to the **Splunk VM** by opening VirtualBox settings, navigating to Shared Folders, clicking the blue folder (+) icon, selecting a folder, enabling Auto-mount, and clicking OK.
![shared folder](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20121032.png)

**Accessing Shared Folders in VirtualBox**

When we create a **shared folder** in VirtualBox, it gets mounted under:  
`/media/sf_sharedfoldername`

However, only members of the **vboxsf** group can access it.  
To give our user (`joe`) access, run:

`sudo adduser joe vboxsf`

Once we've added **joe** to the **vboxsf** group, we can **create a directory** and **mount** the shared folder:

`sudo mount -t vboxsf -o uid=1000,gid=1000 Downloads /share`

This mounts the **Downloads** shared folder to `/share`, setting correct permissions.

**Installing Splunk**

Now that we have the Splunk `.deb` file inside our shared folder, we can install it:

`cd /share, sudo dpkg -i splunk.deb`

Once installed, switch to the **Splunk user** and navigate to the **Splunk binary directory**:

`sudo -u splunk ~/bin`

Run the Splunk binary to start the service:

`./splunk start`

After **accepting the terms of service**, you'll be prompted to **set up a password** for the **Splunk admin user**.

**Ensuring Splunk Starts on Boot**

To ensure Splunk starts **automatically after a reboot**, exit the **Splunk user**, navigate to the bin directory where the binaries are located, and run the following command:

`./splunk enable boot-start`

This ensures Splunk will always start when the machine powers on.

**Installing Splunk Universal Forwarder and Sysmon**

After successfully installing Splunk on the Splunk server, we can proceed with installing the **Splunk Universal Forwarder** and **Sysmon** on both our **target machine** and **AD server**.

We need to ensure the IP address assigned to the machine matches the one in our network diagram. You can check this by opening the **Command Prompt** and typing `ipconfig`. This will display the network information.
![network](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20122005.png)

**Assign a Correct IP Address**  

To fix this, follow these steps:

- Right-click the **Network/Internet** icon and select **Open Network & Internet Settings**.
- Scroll down and click on **Change adapter settings**.
- Right-click the network adapter you’re using and select **Properties**.
- Double-click **Internet Protocol Version 4 (TCP/IPv4)**, and enter the IP address of your choice.

![Static IP](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20122056.png)
![network](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20122106.png)

We can access our **Splunk server** locally by navigating to its IP address, `192.168.10.10`, on port `8000`, as this is the default port where Splunk listens for incoming connections. You should see the **Splunk login page**, but if you don't you might need to keep the Splunk server running. 
![splunk login](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20131606.png)

**Installing Splunk Universal Forwader**

Since Splunk is working correctly, we can now proceed with installing the Splunk Universal Forwarder. To do this, visit [splunk](https://www.splunk.com), log in or sign up for an account, go to the "Products" section, then navigate to "Free Trials and Downloads." Scroll all the way down to find the **Universal Forwarder** section.

After downloading the Splunk Universal Forwarder, we can open the installer, check the License Agreement box, and keep the first option selected. For the credentials, use **admin** as the username and leave the "Generate Password" option selected. Since we don’t have a Deployment Server, we can skip that step.

Next, we’ll configure the **Receiving Server**. Set this to the IP address of our **Splunk server**, which listens on port **9997** by default when receiving events.

![UF](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20131829.png)

**Installing Sysmon and Configuring It with a Custom Rule Set**

On the target machine, download Sysmon from the [Sysmon download page](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon). For configuration, visit the [Sysmon Modular GitHub repository](https://github.com/olafhartong/sysmon-modular) and download the XML configuration file (sysmonconfig.xml) from the bottom of the repository onto the downloads directory.


