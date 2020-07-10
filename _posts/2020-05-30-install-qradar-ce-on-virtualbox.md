---
full_title: "Entry 0x08 - How to install IBM QRadar CE V7.3.3 on VirtualBox"
date: "2020-05-30"
coverImage: "screen-shot-2020-05-29-at-8.08.37-pm.png"
layout: post
tags:
- Beginner
- QRadar
- SIEM 
- IBM
- Security
- Tutorial
- VM
- VirtualBox
---

In this tutorial, we will learn how to install [IBM QRadar Community Edition V7.3.3](https://developer.ibm.com/qradar/ce/) on VirtualBox.

[IBM QRadar SIEM](https://www.ibm.com/products/qradar-siem) is a world-class SIEM tool used by organizations for monitoring and correlating logs from different systems. QRadar can quickly alert SOC Analysts about potential malicious activity and prompt them to take appropriate action.

![Screen Shot 2020-05-29 at 8.08.37 PM](/assets/images/screen-shot-2020-05-29-at-8.08.37-pm.png)

QRadar Community Edition is a version of QRadar which is great for enthusiasts and learners. According to [IBM](https://developer.ibm.com/qradar/ce/):

> Community Edition is a fully-featured free version of QRadar that is low memory, low EPS, and includes a perpetual license. This version is limited to 50 events per second and 5,000 network flows a minute, supports apps, but is based on a smaller footprint for non-enterprise use.

### Pre-requisites

- Download the QRadar CE V7.3.3 OVA from the [official website](https://developer.ibm.com/qradar/ce/) 
> You will need to create an IBM account to complete the download

- Download and install VirtualBox from the [official website](https://www.virtualbox.org/)
> I am using VirtualBox 6.0 on my MacBook Pro with macOS Mojave

- According to IBM, the minimum system requirements are: 

```
    8 GB RAM (10 GB is recommended)
    250 GB free disk space
    2 CPU cores (6 cores is recommended)
    At least one network adapter with Internet connection
```

### 1\. Verify the QRadar CE OVA

Once the **QRadar CE V7.3.3 OVA** is downloaded, let us start by verifying the integrity of the file. IBM provides a button on the [QRadar CE page](https://developer.ibm.com/qradar/ce/) called **SHA256 Sum for OVA**. Click on it to open a `.txt` file with the SHA256 checksum. Use your checksum utility of choice to generate the SHA256 checksum for the downloaded OVA file. I will use `shasum` utility accessible via the Mac terminal. 

![Screen Shot 2020-05-29 at 8.08.16 PM](/assets/images/screen-shot-2020-05-29-at-8.08.16-pm.png)

![Screen Shot 2020-05-29 at 9.24.37 PM](/assets/images/screen-shot-2020-05-29-at-9.24.37-pm-e1590773317975.png)

![Screen Shot 2020-05-29 at 9.30.25 PM](/assets/images/screen-shot-2020-05-29-at-9.30.25-pm.png)

As seen in the screenshot above, the integrity of the OVA file has been confirmed. 

### 2\. Import QRadar CE OVA into VirtualBox

The next step is to launch VirtualBox.

![Screen Shot 2020-05-29 at 9.35.24 PM](/assets/images/screen-shot-2020-05-29-at-9.35.24-pm.png)

Click on the **Import** button and choose the downloaded QRadar CE OVA file. VirtualBox should automatically populate the **Appliance settings** information. At this stage, we can choose to leave the settings in their default state or make minor changes such as **VM** **name**. If required, these settings can be modified later. Click on **Import**. 

![Screen Shot 2020-05-29 at 9.39.21 PM](/assets/images/screen-shot-2020-05-29-at-9.39.21-pm.png)

![Screen Shot 2020-05-30 at 12.04.42 PM](/assets/images/screen-shot-2020-05-30-at-12.04.42-pm.png)

As seen in the screenshot above, the memory assigned to the VM is 6144 MB (6 GB). I will pump this up to 8192 MB (8 GB) as recommended by IBM. To achieve this, click on the **Settings** button and navigate to **System > Motherboard > Base Memory**. Increase the memory and press **OK**.

![Screen Shot 2020-05-30 at 12.13.34 PM](/assets/images/screen-shot-2020-05-30-at-12.13.34-pm.png)

The storage is 250 GB is by default and the number of processors is 2. I will increase this to 4 for better performance. To achieve this, click on the **Settings** button and navigate to **System > Processor > Processors** and increase the processors to 4. Press **OK** once completed. 

![Screen Shot 2020-05-30 at 12.19.18 PM](/assets/images/screen-shot-2020-05-30-at-12.19.18-pm.png)

I will leave the networking settings as the default - Bridged mode. Please take care when changing the networking settings as it is important to ensure that the VM has access to the Internet.  

### 3\. Launch QRadar CE VM

The next step is to launch the VM by clicking on **Start**. 

![Screen Shot 2020-05-30 at 12.21.22 PM](/assets/images/screen-shot-2020-05-30-at-12.21.22-pm.png)

The default username is `root`. Type in `root` and press Enter. 

![Screen Shot 2020-05-30 at 12.23.11 PM](/assets/images/screen-shot-2020-05-30-at-12.23.11-pm.png)

We are immediately prompted to change the password. Remember to use a strong password.

![Screen Shot 2020-05-30 at 12.24.35 PM](/assets/images/screen-shot-2020-05-30-at-12.24.35-pm.png)

The next step is to launch the **setup** script and complete the setup process. Run an `ls` command to verify that the setup script exists in the directory and run it using the command `./setup`

![Screen Shot 2020-05-30 at 12.27.35 PM](/assets/images/screen-shot-2020-05-30-at-12.27.35-pm.png)

You will be prompted to accept the CentOS 7 Linux EULA. Read and press Enter to accept the license terms.

![Screen Shot 2020-05-30 at 12.28.55 PM](/assets/images/screen-shot-2020-05-30-at-12.28.55-pm.png)

Press **Y** to proceed with the installation process.

![Screen Shot 2020-05-30 at 12.29.42 PM](/assets/images/screen-shot-2020-05-30-at-12.29.42-pm.png)

Let QRadar complete the installation steps. This might take a while; be patient!

After a while, you should see a message saying **Press ENTER to complete Installation**. Press Enter as directed by the message.

![Screen Shot 2020-05-30 at 1.06.09 PM](/assets/images/screen-shot-2020-05-30-at-1.06.09-pm.png)

You will be prompted to enter the new **admin** password. This is the password for the `admin` user on QRadar CE web user interface. Remember to use a strong password. Note that this is a different account from the previous `root` user account for the CentOS VM.

![Screen Shot 2020-05-30 at 1.08.23 PM](/assets/images/screen-shot-2020-05-30-at-1.08.23-pm.png)

![Screen Shot 2020-05-30 at 1.09.09 PM](/assets/images/screen-shot-2020-05-30-at-1.09.09-pm.png)

The next step is to verify the installation and access the QRadar CE user interface.

### 4\. Verify the QRadar CE Installation

The easiest way to verify if the QRadar CE user interface is up and running is to use the `curl` command on the CentOS VM.

Run the command: `curl https://localhost -k`  and the output should be as seen in the screenshot below.

![Screen Shot 2020-05-30 at 1.20.09 PM](/assets/images/screen-shot-2020-05-30-at-1.20.09-pm.png)

Note the `-k` option in the `curl` command which skips certificate validation. You can also use `--insecure`.

Now that QRadar CE is working on `localhost` (CentOS VM), we can try accessing it remotely from the host machine. To achieve this, we need to grab the IP address of the CentOS VM.  

Use the `ifconfig` command to quickly view the IP address. 

![Screen Shot 2020-05-30 at 1.37.00 PM](/assets/images/screen-shot-2020-05-30-at-1.37.00-pm.png)

As seen in the screenshot above, the IP address is `192.168.0.182`. I will now attempt to connect to this IP from my host machine (MacBook Pro).

Before attempting access from a web browser, I will repeat the `curl` command on the Mac terminal: `curl https://192.168.0.182 -k`. If all goes well, the output should be same as what we see below and in the previous `curl` output from the CentOS VM.

![Screen Shot 2020-05-30 at 1.39.13 PM](/assets/images/screen-shot-2020-05-30-at-1.39.13-pm.png)

Great! Looks like there is proper connectivity. I will fire up Google Chrome and attempt to access QRadar CE. 

![Screen Shot 2020-05-30 at 1.41.33 PM](/assets/images/screen-shot-2020-05-30-at-1.41.33-pm.png)

Chrome will display a **Your connection is not private** warning. We can ignore this for now and click on **Advanced > Proceed to 192.168.0.182 (unsafe)**

![Screen Shot 2020-05-30 at 1.53.30 PM](/assets/images/screen-shot-2020-05-30-at-1.53.30-pm.png)

There you go! Welcome to QRadar CE. Log in with the username `admin` and password which was set on the console during the installation step. 

![Screen Shot 2020-05-30 at 1.55.12 PM](/assets/images/screen-shot-2020-05-30-at-1.55.12-pm.png)

You will be greeted with the **QRadar Community Edition - License Agreement**. Read and click on **Accept** to continue. 

![Screen Shot 2020-05-30 at 1.56.27 PM](/assets/images/screen-shot-2020-05-30-at-1.56.27-pm.png)

This is the **Dashboard** view of QRadar CE. However, I noticed that the **System Time** (displayed on the top-right) is not tuned to my timezone. 

To change the **System Time**, click on **Admin** to open the Admin menu.

![Screen Shot 2020-05-30 at 2.03.40 PM](/assets/images/screen-shot-2020-05-30-at-2.03.40-pm.png)

Next, click on **System and License Management**. 

![Screen Shot 2020-05-30 at 2.07.38 PM](/assets/images/screen-shot-2020-05-30-at-2.07.38-pm-e1590836602154.png)

Select the **localhost (console)** item and click on the **Actions** menu item. Under **Actions**, click on **View and Manage System**. 

![Screen Shot 2020-05-30 at 3.10.49 PM](/assets/images/screen-shot-2020-05-30-at-3.10.49-pm.png)

Before we change the system time, I would like to mention that this is a critical area of QRadar CE as there are a variety of configuration options. You can view the licensing details such as EPS utilization, configure the firewall to whitelist IP addresses, and configure an email server among many other actions. 

Click on **System Time** and set the desired time and select the correct timezone. Once completed, press **Save**. 

![Screen Shot 2020-05-30 at 3.14.42 PM](/assets/images/screen-shot-2020-05-30-at-3.14.42-pm.png)

You will be notified that services will be restarted and asked for another confirmation. Press **OK**. 

![Screen Shot 2020-05-30 at 3.17.19 PM](/assets/images/screen-shot-2020-05-30-at-3.17.19-pm.png)

Once we provide confirmation, a message should appear saying **System Time is updated successfully. Services will now restart** as seen in the screenshot below. You can close the tab and refresh the QRadar CE home page in a few minutes.

![Screen Shot 2020-05-30 at 3.40.29 PM](/assets/images/screen-shot-2020-05-30-at-3.40.29-pm.png)

### Conclusion & What's Next

In this tutorial, we installed QRadar CE V7.3.3 on VirtualBox and completed basic configuration of the system time. QRadar CE offers SIEM Administrators, SOC Analysts, and enthusiasts the power to experiment and practice real-world concepts in a test environment. 

The next step is to feed some logs into our newly installed QRadar CE. It is to be noted that QRadar CE only supports a handful of parsers/DSMs (Device Support Modules) out of the box. The complete list can be viewed in the [QRadar CE V7.3.3 Official Documentation](http://ibm.biz/ce733overview). However, more DSMs can be added for more integrations. Check out [this video](https://www.youtube.com/watch?v=4pDfMmlUKs0) for more details.

I recommend starting with a basic integration such as Linux OS. This can be easily achieved with Linux VM (such as CentOS or Ubuntu) using `syslog`. Check out [this video](https://www.youtube.com/watch?v=Dmf2iwRqATI) for more details. 

Here are some other useful resources: 

- [QRadar CE V7.3.3 Official Documentation](http://ibm.biz/ce733overview)
- [QRadar V7.3.3 Getting Started Guide](https://www.ibm.com/support/knowledgecenter/SS42VS_7.3.3/com.ibm.qradar.doc/b_qradar_gs_guide.pdf)
- [QRadar Supported DSMs](https://www.ibm.com/support/knowledgecenter/en/SS42VS_DSM/com.ibm.dsm.doc/r_supported_dsm_list.html?cp=SS42VS_7.3.3)
- [DSM Configuration Guide](https://www.ibm.com/support/knowledgecenter/SS42VS_DSM/com.ibm.dsm.doc/b_dsm_guide.pdf?origURL=SS42VS_DSM/b_dsm_guide.pdf)
- [IBM X-Force App Exchange](https://exchange.xforce.ibmcloud.com/hub?br=QRadar) 
- [Jose Bravo Videos on YouTube](https://www.youtube.com/user/jbravovideos/videos)

Please reach out if you have any questions or comments.