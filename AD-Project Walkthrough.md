---
Active Directory Lab in VirtualBox
---

#### **Project Objectives:**
- Set up an **Active Directory** environment
- Deploy **Splunk** to monitor AD and Windows activities

#### **Steps:**
**Part 1:** Designing a Lab Diagram  
**Part 2:** Installing Virtual Machines (Windows & Linux)  
**Part 3:** Installing and Configuring Sysmon and Splunk  
**Part 4:** Configuring Active Directory  
**Part 5:** Generating Telemetry with Kali 

---

### **Part 1**: Designing a Lab Diagram
The goal of this project is to set up an Active Directory (AD) lab to simulate attacks on AD users and capture the activity on our Splunk server for analysis. The lab will consist of the following components:
- **Windows Machines** (victims)
- **Kali Linux Machine** (attacker)
- **Windows Server** (Active Directory and Domain Controller)
- **Splunk Server** (to monitor and analyze the attacks)

![Diagram](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-12%20155412.png)

---

### **Part 2:** Installing Virtual Machines (Windows & Linux)  

**Note**: The specifications for the machines heavily depend on your systems capabilties.

**Windows 10**
To install Windows 10, visit this [link](https://www.microsoft.com/en-ca/software-download/windows10) and click **Download Tool Now** to get the Media Creation Tool, which will generate a Windows ISO image. Once downloaded, open the tool, accept the agreement, select **Create installation media (ISO file)**, and proceed until you reach the **Choose which media to use** page, then select **ISO file**. After the ISO file is created, open VirtualBox, click **New**, enter a name (demo), select the ISO image, skip unattended installation, and configure the necessary system specifications before installation.

**Kali Linux**
To install Kali Linux, visit this [link](https://www.kali.org/get-kali/#kali-virtual-machines), select the **VirtualBox 64-bit** option (or 32-bit if applicable), and click the download arrow. Since the file is in **7z** format, you'll need **7-Zip** installed; you can download it from [here](https://www.7-zip.org/). Once installed, open the Kali Linux file, proceed through the prompts, and navigate to the saved **7z** file. Right-click the file, select **7-Zip > Extract to kali-linux**, and once extraction is complete, you’ll see a **.vbox** file. Double-clicking it will automatically import Kali Linux into VirtualBox, setting up the virtual machine for use.

**Windows Server**
To install Windows Server, visit this [link](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022) and download the **64-bit ISO image** after filling out the required information. Once the download is complete, open VirtualBox, click **New**, enter a name (ADDC01), add the ISO image, and skip unattended installation.

**Splunk Server**
To install a Splunk Server, visit this [link](https://ubuntu.com/) and download the latest **Ubuntu Server** ISO. Once the download is complete, open VirtualBox, click **New**, enter a name (Splunk), add the ISO image, and skip unattended installation. 

---

### **Part 3:** Installing and Configuring Splunk and Sysmon  

**VirtualBox and Networking Issues**
Before setting up Splunk, make sure you're using the latest version of **Oracle VirtualBox**. I spent hours troubleshooting **NAT network issues**, only to discover that updating VirtualBox was the simple solution.

To create a NAT network, navigate to the **Tools** icon, click **Create**, assign a name and an IPv4 prefix, then assign all devices to the newly created NAT network.
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

- Once those two things are successfully downloaded, extract the Sysmon ZIP file and copy the file path of the extracted folder. 
- Open **PowerShell as Administrator**, navigate to the copied path, and install Sysmon using the downloaded configuration file. This command installs Sysmon and applies the configuration.

![powershell](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20133210.png)

**Configuring the Splunk Universal Forwarder (inputs.conf)**

This step is **crucial** because we need to configure the **Splunk Universal Forwarder** to specify **which logs** we want to send to our **Splunk Server**.

**1. Locate the Configuration Directory**

The `inputs.conf` file is located in:
`C:\Program Files\SplunkUniversalForwarder\etc\system\default\`

However, we **don’t** want to modify the default configuration files directly. Instead, we will create a new `inputs.conf` file in the **local** directory:
`C:\Program Files\SplunkUniversalForwarder\etc\system\local\`

**2. Create and Edit `inputs.conf`**

Since **administrator privileges** are required to create or modify files in this directory, follow these steps:

Open **Notepad** as Administrator:
- In the **search bar**, type **Notepad**, right-click it, and select **Run as administrator**.

Visit [github](https://github.com/MyDFIR/Active-Directory-Project) to get a pre-configured Splunk universal forwarder config file. This file is set up to forward logs related to:
- **Application logs**
- **Security logs**
- **System logs**
- **Sysmon logs**

![config](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20140836.png)

Save the file and the content from the **Notepad** into the directroy,`C:\Program Files\SplunkUniversalForwarder\etc\system\local\`, with the filename **inputs.conf**

![notepad](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20141243.png)

**3. Editing Services**

After modifying `inputs.conf`, you must **restart the Splunk Universal Forwarder service** to apply the changes. 
- Open the **Services** application:
- In the **search bar**, type **Services**, and hit **Enter**.
- Find **SplunkForwarder** in the list.
- Right-click it and select **Restart**.

![services](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20150713.png)

Also by default, Splunk runs under the **NT SERVICE** account, which may **not have sufficient permissions** to collect all necessary logs. To fix this:

- Right-click and select **Properties** for **SplunkForwader**.
- Navigate to the **Log On** tab.
- Select **“This account”**, and enter credentials for an account with **higher privileges**.
- Restart the service to apply the changes.

![privs](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20150725.png)

**Finalizing Splunk Server Configuration**

At this point, we have installed **Sysmon**, configured **Splunk Universal Forwarder**, and updated our **inputs.conf** file. Now, we can finish the Splunk setup.

**1. Logging into Splunk**

Log into **Splunk Web** using the credentials set during installation.  

![Splunk UI](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20151612.png)

**2. Creating an Index**

We need to create an index that matches the one specified in `inputs.conf`.

![index](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20151623.png)

Looking back at our `inputs.conf` file, all events are being sent to an index called **"endpoint"**. Since it’s not listed, we must create it manually.  

![endpoint](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20151634.png)

Click **New Index**, and name it **endpoint**.  

![adding](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20151642.png) 

**3. Configuring Data Reception**

To enable the Splunk server to receive forwarded data:

- Navigate to **Settings** → **Forwarding and Receiving**
- Under **Receiving Data**, click **Configure Receiving**
- **Add a new receiving port** (default is `9997`) and **Save**  

![config](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20152132.png)

**4. Verifying Data Ingestion**

To confirm that logs are being received:

- Click the **Apps** icon → **Search & Reporting**
- In the search bar, enter:
`index=endpoint`
- If data is being received, you should see log events appearing.  

![data](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20152139.png)
![data](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20152147.png)

**After we've successfully installed Splunk and Sysmon on the target machine, you need to do the same thing for the AD server.**

---

### **Part 4:** Configuring Active Directory  

Before we start configuring anything, we need to make sure the Windows Server has the correct IP address, as shown in the network diagram. In our case, it was already set correctly, but if it wasn’t, we would have followed the same steps we used to configure the Windows target machine.

**Configuring Server Manager**

- Open **Server Manager** by searching for it in the Windows search bar.
- Click **Manage** > **Add Roles and Features**.
- Click **Next**, make sure the first option is selected, and click **Next** again.

![server managaer](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20161248.png)

- Select **Active Directory Domain Services (AD DS)**, then click **Add Features** and **Next**.

![Services](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20161255.png)

- Keep clicking **Next** until you reach the install page, then click **Install**.

![install](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20161306.png)

- After the download is complete, we can proceed to promote the server to a domain controller.

![DC](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20161541.png)

- Select **Add a new forest domain**, then click **Next**.

![forest](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20161556.png)

- Keep the default settings, set a password, and click **Next**.

![password](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20161604.png)

- Once the prerequisites check is complete, we can click **Install**.

![install](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20161819.png)

- After installation, the system will log us out. When we return to the login page, we should see the domain listed, indicating that AD DS was successfully installed and the server has been promoted to a domain controller.

![login](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20161839.png)

**Adding Users in Active Directory**

Now that everything is installed and configured, we can start adding users.

 **1. Opening Active Directory Users and Groups**

- Click the **Tools** icon in **Server Manager** and select **Active Directory Users and Computers (ADUC)**.
- This is where we can create and manage **users, groups, computers, and more**.

**2. Exploring the Default Folders**
- Expanding the domain reveals several key folders:
- **Built-in**: Contains default security groups automatically created by Active Directory.
- **Users**: Stores all user accounts along with additional default groups.

**3. Creating a New User**
- To add a user, right-click inside the **Users** folder, select **New → User**, and fill in the required details.
- In a real-world scenario, users are typically organized into departments like **HR, IT, or Finance**, rather than being stored in a single folder.

**4. Organizing Users with Organizational Units (OUs)**
- To better manage users, we can create **Organizational Units (OUs)**.
- Right-click the domain, select **New → Organizational Unit**, and name it based on a department or team.

**5. Adding Users to an OU**
- Once an OU is created, you can add users by right-clicking the unit and selecting **New → User**.

![IT](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20162251.png)
![HR](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20162301.png)


**Joining Windows Machine to the Domain & Fixing DNS Issues**

- Open **System Properties** (pc in search bar → Properties → Advanced system settings).
- Under Computer Name, click Change, then enter the domain name.
- You will run into this issue.

![issue](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20162916.png)

Fixing **MY`name`.local** Resolution Issue:

- Open Network & Internet Settings (right-click network icon → Open settings).
- Click Change adapter options.
- Right-click the active network adapter → Properties → IPv4.
- Replace the current DNS (e.g., 8.8.8.8) with the Domain Controller’s IP.
- Verify changes in Command Prompt (cmd).
- Now, **MY`name`.local** should resolve correctly.

Next, we can proceed with adding the machine to the domain. When prompted, enter the **administrator login credentials** to complete the process.

![login](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20163117.png)
![changhe](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20163128.png)
**Note:** In a real-world scenario, it's best practice to assign users to custom groups with the necessary permissions to join computers to the domain, rather than using administrator credentials.

Once the machine is successfully joined, we can return to the login page and sign in to **Jenny Smith’s account** using her credentials on the target machine.

---

### **Part 5:** Generating Telemetry with Kali

To begin, we need to assign the correct IP address in our kali machine as specified in our diagram (192.168.10.250). To do this, search for **Advanced Network Settings**, select **Wired connection 1**, and click on the **settings** icon located on the bottom left side of the screen. Next, navigate to the IPv4 tab, set the **Method** option to **Manual**, and enter the correct IPv4 address and Gateway.

![static IP](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20163455.png)
![confirm](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20163503.png)

To simulate a brute-force attacks on our users we created, we can use a tool called **Crowbar** [GitHub](https://github.com/galkan/crowbar) to launch attacks on our AD server or Windows machine. I install the Python version of Crowbar from GitHub, as Linux Debian does not support a specific library (maybe it is possible, but I couldn't find a way).

![crowbar](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20163512.png)

In addition, I manually downloaded the **rockyou.txt** wordlist [GitHub Link](https://github.com/praetorian-inc/Hob0Rules/blob/master/wordlists/rockyou.txt.gz).

Since I assigned strong passwords for the users, I'll speed up the cracking process by placing the password within the first 20-50 words of rockyou.txt. 

Before we can launch any attacks, we need to enable **RDP** on our target Windows machine. This can be done by searching for **PC**, selecting **Properties**, then **Advanced system settings**. In the **Remote** tab, choose the second option in the **Remote Desktop** section.

![rdp](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20163526.png)

Click **Select Users** and add the two users we created on our domain.

![users](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20163534.png)

At this point, we can launch the brute-force attack by running the following command: 

`python3 crowbar.py -b rdp -C rockyou.txt -s 192.168.100.100/32`  

Within a few seconds, we successfully obtain the credentials.

We can head over to Splunk to examine the telemetry we've generated. To do this, navigate to **Search and Reporting**, then type **index=endpoint tsmith**. This will display all the data associated with the specific user. From here, we can dive deeper into the EventCode logs for more detailed information. In this case, we focus on Event ID `4625`, which corresponds to a failed logon attempt. This is a direct result of our brute-forcing attack on the user's login credentials. For further context on Event ID 4625, you can check the detailed explanation [link](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4625)

![event id](https://github.com/J4ck3lXploit/Active-Directory-Lab/blob/main/images/Screenshot%202025-02-21%20163542.png)

### Beyond

r
