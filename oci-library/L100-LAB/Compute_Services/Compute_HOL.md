# Compute Service

## Table of Contents

[Overview](#overview)

[Prerequisites](#Prerequisites)

[Practice 1: Generate SSH Keys](#practice-1-generate-ssh-keys)

[Practice 2: Creating a Web Server on a Compute Instance](#practice-2-creating-a-web-server-on-a-compute-instance)

**Note:** *Some of the UIs might look a little different than the screen shots included in the instructions, but you can still use the instructions to complete the hands-on labs.*

## Overview

Oracle Cloud Infrastructure Compute lets you provision and manage compute hosts, known as instances. You can launch instances as needed to meet your compute and application requirements. After you launch an instance, you can access it securely from your computer, restart it, attach and detach volumes, and terminate it when you're done with it. Any changes made to the instance's local drives are lost when you terminate it. Any saved changes to volumes attached to the instance are retained.

Be sure to review [Best Practices for Your Compute Instance](https://docs.cloud.oracle.com/iaas/Content/Compute/References/bestpracticescompute.htm) for important information about working with your Oracle Cloud Infrastructure Compute instance.

## Prerequisites

- Oracle Cloud Infrastructure account credentials (User, Password, and Tenant)
- To sign in to the Console, you need the following:
  - Tenant, User name and Password
  - URL for the Console: [https://console.us-ashburn-1.oraclecloud.com/](https://console.us-ashburn-1.oraclecloud.com/)
  - Oracle Cloud Infrastructure supports the latest versions of Google Chrome, Firefox and Internet Explorer 11

## Practice 1: Generate SSH Keys

Generate SSH keys to be used later while launching an instance.
See the [tutorial](https://github.com/sebdemanche/learning-library/blob/master/common/labs/generate-ssh-key/generate-ssh-keys.md)

## Practice 2: Creating a Web Server on a Compute Instance

Oracle Cloud Infrastructure  offers both Bare Metal and Virtual Machine instances:

- **Bare Metal**  - A bare metal compute instance gives you dedicated physical server access for highest performance and strong isolation.
- **Virtual Machine**  - A Virtual Machine (VM) is an independent computing environment that runs on top of physical bare metal hardware. The virtualization makes it possible to run multiple VMs that are isolated from each other. VMs are ideal for running applications that do not require the performance and resources (CPU, memory, network bandwidth, storage) of an entire physical machine.

An Oracle Cloud Infrastructure VM compute instance runs on the same hardware as a Bare Metal instance, leveraging the same cloud-optimized hardware, firmware, software stack, and networking infrastructure.

1. Navigate to the **Compute** tab and click **Create Instance**. We will launch a VM instance for this lab.

2. The Create Compute Instance wizard will launch. Set the name of the server to *Web-Server*. Click on the *Show Shape, Networking, Storage Options* link to expand that area of the page.
    ![Create step 1](media/Create1.png)

3. Most of the defaults are perfect for our purposes. However, you will need to scroll down to the Configure Networking area of the page and select the *Assign a public IP address* option.
    ![Create step 2](media/Create2.png)

    ***NOTE:*** *You need a public IP address so that you can SSH into the running instance later in this lab.*

4. Scroll down to the SSH area of the page. Choose the *id_rsa.pub* SSH key that you created earlier in this lab. Press the *Create* button to create your instance.

    Launching an instance is simple and intuitive with few options to select. The provisioning of the compute instance will complete in less than a minute and the instance state will change from provisioning to running.

5. Once the instance state changes to Running, you can SSH to the Public IP address of the instance.

    ![Create step 3](media/Create3.png)

6. To connect to the instance, you can use `Terminal` if you are using MAC or `Gitbash` if you are using Windows. On your terminal or gitbash enter the following command:

    ```
    ssh -i ~/.ssh/<ssh_key_name> opc@<public_ip_address>
    ```

7. For this lab, we are going to install an Apache HTTP Webserver and try to connect to it over the public Internet. SSH into the Linux instance and run following commands:

    **Note** *Apache HTTP Server is an open-source web server developed by the Apache Software Foundation. The Apache server hosts web content, and responds to requests for this content from web browsers such as Chrome or Firefox.*

    - Install Apache http

      ```shell
      sudo yum install httpd -y
      ```

    - Start the apache server and configure it to start after system reboots

      ```shell
      sudo apachectl start
      sudo systemctl enable httpd
      ```

    - Run a quick check on apache configurations

      ```shell
      sudo apachectl configtest
      ```

    - Create firewall rules to allow access to the ports on which the HTTP server listens.

      ```shell
      sudo firewall-cmd --permanent --zone=public --add-service=http
      sudo firewall-cmd --reload
      ```

    - Create an index file for your webserver

      ```shell
      sudo bash -c 'echo This is my Web-Server running on Oracle Cloud Infrastructure >> /var/www/html/index.html'
      ```

8. Open your browser and navigate to `http://Public-IPAddress` (the IP address of the Linux VM)

    **NOTE:** Your browser will not return anything because port 80 was not opened into the Security Lists

9. Using the menu, click on **Virtual Cloud Network** and then the VCN you created for this practice.

    ![Click on the VCN](media/vcn1.png)

10. Now click on **Security Lists** on the left navigation bar for the VCN.
    ![Click on Security Lists](media/vcn2.png)

11. Click on the **Default Security List**.

12. Here you need to open port 80. Click on **+ Another Ingress Rule** and add the following values as shown below:

    - **Source Type:** CIDR
    - **Source CIDR**: 0.0.0.0/0
    - **IP Protocol:** TCP
    - **Source Port Range:** All
    - **Destination Port Range:** 80
    - Click on **Add Ingress Rules** at the bottom.

    ![Add Ingress Rule](media/addIngress1.png)

13. Navigate to `http://<public_ip_address>` (the IP address of the Linux VM) in your browser. And now you should see the index page of the webserver we created above.

    ![Open you brwser to the public IP address](media/image13.png)

### Troubleshooting

If you are unable to see the webserver on your browser, possible scenarios include:

- VCN Security Lists is blocking traffic, Check VCN Security List for ingress rule for port 80
- Firewall on the linux instance is blocking traffic
  
  - `# sudo firewall-cmd --zone=public --list-services` (this should show http service as part of the public zone)
  - `# sudo netstat -tulnp | grep httpd` (an httpd service should be listening on the port 80, if it’s a different port, open up that port on your VCN SL)

- Your company VPN is blocking traffic
