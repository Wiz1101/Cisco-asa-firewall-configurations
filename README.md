# Configuring network with Cisco

Equipement:

* Firewall (Cisco ASA 5510)
* Switch (Cisco Catalyst 3550 SERIES)
* Access Point (Cisco Aeronet 3502 series)
* Linux Server (Ubuntu 20.04.6)
* Windows Server 2019


## Configuring Cisco equipment to it's factory settings 

1. **Firewall (Cisco ASA 5510)**:
   ```shell
   G5-ASA-5510> enable
   G5-ASA-5510# conf t
   G5-ASA-5510(config)# configure factory-default
   G5-ASA-5510(config)# wr mem
   ```

2. **Switch (Cisco Catalyst 3550 SERIES)**:

   ```shell
   Switch> en
   Switch# write erase 
   Switch# reload 
   ```

2. **Access Point (Cisco Aeronet 3502 series)**:

   ```shell
   ap> en
   ap# write erase
   ap# reload
   ```

## The configuration steps on Firewall (Cisco ASA 5510)

* **Connect to the Console Port with MacOS or Linux**

   - Step 1. Use the Finder to go to Applications > Utilities > Terminal

   - Step 2. Connect the OS USB port to the ASA

   - Step 3. Enter the following commands to find the OS X USB port number:
   Example:

   ```shell
   macbook:user$ cd /dev
   macbook:user$ ls -ltr /dev/*usb*
   crw-rw-rw- 1 root wheel 9, 66 Apr 1 16:46 tty.usbmodem1a21 
   macbook:user$ screen /dev/tty.usbmodem1a21 9600
   ```
* **Connect to the Console Port with Windows**<br>
   Ensure that you have console access to the Cisco ASA 5510 through a console cable and terminal emulation software like PuTTY or a similar tool.

1. **Connect to the Cisco ASA 5510**:
   - Connect one end of the console cable to the console port on the ASA 5510.

2. **Enter privileged EXEC mode by typing**:

   ```shell
   ciscoasa> enable
   ```

   - Enter global configuration mode:

   ```shell
   ciscoasa# conf t
   ```

3. **Setting Hostname** (Optional):
   - You can set a hostname for the ASA if desired:

   ```shell
   ciscoasa(config)# hostname G5-ASA-5510
   G5-ASA-5510(config)# wr mem
   ```

4. **Configuring Interfaces**:
   - Define the outside and inside interfaces.

   ```shell
   G5-ASA-5510(config)# int Ethernet0/0
   G5-ASA-5510(config)# nameif outside
   G5-ASA-5510(config)# ip address 192.168.0.50 255.255.255.0
   G5-ASA-5510(config)# security-level 0
   G5-ASA-5510(config)# no shut
   G5-ASA-5510(config)# int Ethernet0/1
   G5-ASA-5510(config)# nameif inside
   G5-ASA-5510(config)# ip address 10.0.0.1 255.255.255.0
   G5-ASA-5510(config)# security-level 100
   G5-ASA-5510(config)# no shut
   G5-ASA-5510(config)# wr mem
   ```

   - The above configuration says that Ethernet0/0 will be the interface for outside network and Ethernet0/1 is the interface for inside network.

5. **Configuring Routing(from inside to outside)**:

   ```shell
   G5-ASA-5510(config)# route outside 0 0 192.168.0.1
   ```



6. **Configuring DHCP & DNS inside**:
    - Address range of DHCP inside network: 10.0.0.100 - 10.0.0.200 
    - We binded DNS to OpenDNS servers (More Secure)
   ```shell
   G5-ASA-5510(config)# dhcpd address 10.0.0.100-10.0.0.200 inside
   G5-ASA-5510(config)# dhcpd dns 8.8.8.8 8.8.4.4
   G5-ASA-5510(config)# dhcpd lease 7200                          
   G5-ASA-5510(config)# dhcpd enable inside 
   G5-ASA-5510(config)# wr mem
   ```

7. **Configuring NAT**:
   - Configure basic NAT to allow inside devices to access the internet:

   ```shell
   G5-ASA-5510(config)# object network myNatPool
   G5-ASA-5510(config-network-object)# range 192.168.0.51 192.168.0.52
   G5-ASA-5510(config-network-object)# exit
   G5-ASA-5510(config)# object network myInsNet  
   G5-ASA-5510(config-network-object)# subnet 10.0.0.0 255.255.255.0  
   G5-ASA-5510(config-network-object)# nat (inside,outside) dynamic myNatPool
   G5-ASA-5510(config-network-object)# exit 
   G5-ASA-5510(config)# object network myLinServ
   G5-ASA-5510(config-network-object)# host 192.168.0.50  
   G5-ASA-5510(config-network-object)# nat (inside,outside) static 10.0.0.250
   G5-ASA-5510(config-network-object)# exit
   G5-ASA-5510(config)# same-security-traffic permit inter-interface
   G5-ASA-5510(config)# wr mem 
   ```

8. **Configuring Access Control List (ACL)**:

   ```shell

   G5-ASA-5510(config)# access-list allow_icmp_inside extended permit icmp 10.0.0.0 255.255.255.0 192.168.0.0 255.255.255.0
   G5-ASA-5510(config)# access-group allow_icmp_inside in interface inside
   G5-ASA-5510(config)# wr mem
   ```




## The configuration steps on Access Point Cisco Aeronet 3502 series
**Note**: Before proceeding, make sure you have PuTTY installed on your computer.

   - To configure a Cisco Aironet access point using PuTTY, you'll need to connect to the access point via its command-line interface (CLI) over SSH or Telnet. Here are the steps to do this:
   1. **Connect to the Access Point**:
      - Ensure your computer is connected to the same network as the Cisco Aironet access point.
      - Launch PuTTY.

   2. **Open a New Session**:
      - In the PuTTY configuration window, enter the IP address or hostname of the access point in the "Host Name (or IP address)" field.
      - Choose the connection type:
      - For SSH, select "SSH."
      - For Telnet, select "Telnet."
      - Enter the port number (usually 22 for SSH or 23 for Telnet).

   3. **Initiate the Connection**:
      - Click the "Open" button to initiate the connection.

   4. **Login**:
      - Once the connection is established, you will be prompted to log in.
      - Enter the username and password for the access point. By default, the username and password are both "cisco" (without the quotes). If these credentials have been changed, use the updated login credentials.

   5. **Access the Command-Line Interface (CLI)**:
      - After successfully logging in, you will have access to the command-line interface (CLI) of the Cisco Aironet access point.

   6. **Configure the BVI1 Interface**:
      - To configure the BVI1 interface and assign an IP address and netmask, use the following commands:
      
      ```
         enable
         configure terminal
         interface BVI1
         ip address <IP_ADDRESS> <NETMASK>
         exit
      ```

      Replace `<IP_ADDRESS>` with the desired IP address (e.g., `192.168.1.2`) and `<NETMASK>` with the subnet mask (e.g., `255.255.255.0`).

   7. **Save Configuration**:
      - To save your configuration, use the following command:
      
      ```
         write memory
      ```

      This command saves your changes to the device's configuration.

   8. **Exit PuTTY**:
      - Type `exit` to log out of the access point's CLI, and then close the PuTTY session.

   9. **Verify Configuration**:
      - You can verify the configuration by trying to access the access point's web GUI using the new IP address you configured for the BVI1 interface.

      That's it! You have configured the Cisco Aironet access point's BVI1 interface using PuTTY. Be sure to follow best practices for securing your access point and changing default login credentials for improved security.

## The configuration steps on Switch (Cisco Catalyst 3550 SERIES)




## The configuration steps on Servers

1. **Configuring Linux server:**
   1.  Assigning static IP to the server
   - We have to edit "/etc/netplan/00-installer-config.yaml" file
      ```shell
      network:
         version: 2
         renderer: networkd
         ethernets:
            eth0:
               addresses:
                  - [*IP]/24
               routes:
                  - to: default
                  via: [*Gateway IP]
               nameservers:
                  addresses: [8.8.8.8, 8.8.4.4]
      ```
       ```shell
      $ sudo netplan apply
      ```
   2. Configuring HTTP and FTP server on linux:
   
      ```shell
      $ sudo apt install apache2
      $ sudo apt install vsftpd
      $ sudo ufw enable 
      $ sudo ufw allow 'Apache Full'
      $ sudo ufw allow 20,21,22,990/tcp
      $ sudo ufw allow 40000:50000/tcp
      $ systemctl status apache2
      ```
      - Also we edited /etc/vsftpd.conf file for FTP service on the ubuntu server side where we added these two rules in the end of the file:
      ```shell
      pasv_enable=Yes
      pasv_max_port=50000
      pasv_min_port=40000
      ```

2. **Configuring Windows Server:**

   1. Connect the ethernet cable to the laptop
   2. Assign static IP to Windows Server. 
      - Control Panel --> Network and Internet --> Network and Sharing Center --> Select the Ethernet and right click on Priorities --> Choose Internet Protocol version 4 --> choose properties and assign the required IP, gateway and DNS.
   3. To download browser of your choice, open Internet Explorer go to Tools which on the right top corner
   4. Choose Internet Options
   5. Go to security panel which is on top right of the new window
   6. Choose Custom Level
   7. Find File Downloads and enable it
   8. Download the browser of your choice.