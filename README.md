# Configuring network with Cisco
A small writeup how to configure a network

**Equipement:**

* Firewall (Cisco ASA 5510)
* Switch (Cisco Catalyst 3550 SERIES)
* Access Point (Cisco Aeronet 3502 series)
* Linux Server (Ubuntu 20.04.6)
* Windows Server 2019


## Configuring Cisco equipment to its factory settings 

1. **Firewall (Cisco ASA 5510)**:
   ```shell
   ciscoasa> enable
   ciscoasa# conf t
   ciscoasa(config)# configure factory-default
   ciscoasa(config)# wr mem
   ```

2. **Switch (Cisco Catalyst 3550 SERIES)**:

   ```shell
   Switch> en
   Switch# write erase 
   Switch# reload 
   System configuration has been modified. Save? [yes/no]: no
   ```

2. **Access Point (Cisco Aeronet 3502 series)**:

   ```shell
   ap> en
   ap# write erase
   ap# reload
   ```

## Configuring Firewall (Cisco ASA 5510)

   * **Connect to the Console Port with MacOS or Linux**

      - Step 1. Use the Finder to go to Applications > Utilities > Terminal

      - Step 2. Connect the OS USB port to the ASA

      - Step 3. Enter the following commands to find the OS X USB port number:
         ```shell
         macbook:user$ cd /dev
         macbook:user$ ls -ltr /dev/*usb*
         crw-rw-rw- 1 root wheel 9, 66 Apr 1 16:46 tty.usbmodem1a21 
         macbook:user$ screen /dev/tty.usbmodem1a21 9600
         ```
   * **Connect to the Console Port with Windows**<br>
      - Ensure that you have console access to the Cisco ASA 5510 through a console cable and terminal emulation software like PuTTY or a similar tool.

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

3. **Set Hostname** (Optional):
   - You can set a hostname for the ASA if desired:

   ```shell
   ciscoasa(config)# hostname Firewall
   Firewall(config)# wr mem
   ```

4. **Configure Interfaces**:
   - Define the outside and inside interfaces.

   ```shell
   Firewall(config)# int Ethernet0/0
   Firewall(config)# nameif outside
   Firewall(config)# ip address 192.168.0.50 255.255.255.0
   Firewall(config)# security-level 0
   Firewall(config)# no shut
   Firewall(config)# int Ethernet0/1
   Firewall(config)# nameif inside
   Firewall(config)# ip address 10.0.0.1 255.255.255.0
   Firewall(config)# security-level 100
   Firewall(config)# no shut
   Firewall(config)# wr mem
   ```

   - The above configuration says that Ethernet0/0 will be the interface for outside network and Ethernet0/1 is the interface for inside network.

5. **Configure Routing(from inside to outside)**:

   ```shell
   Firewall(config)# route outside 0 0 192.168.0.1
   ```



6. **Configure DHCP & DNS inside**:
    - Address range of DHCP inside network: 10.0.0.100 - 10.0.0.200 
    - We binded DNS to OpenDNS servers (More Secure)
   ```shell
   Firewall(config)# dhcpd address 10.0.0.100-10.0.0.200 inside
   Firewall(config)# dhcpd dns 8.8.8.8 8.8.4.4
   Firewall(config)# dhcpd lease 7200                          
   Firewall(config)# dhcpd enable inside 
   Firewall(config)# wr mem
   ```

7. **Configure NAT**:
   - Configure basic NAT to allow inside devices to access the internet:

   ```shell
   Firewall(config)# object network myNatPool
   Firewall(config-network-object)# range 192.168.0.51 192.168.0.52
   Firewall(config-network-object)# exit
   Firewall(config)# object network myInsNet  
   Firewall(config-network-object)# subnet 10.0.0.0 255.255.255.0  
   Firewall(config-network-object)# nat (inside,outside) dynamic myNatPool
   Firewall(config-network-object)# exit 
   Firewall(config)# object network myLinServ
   Firewall(config-network-object)# host 192.168.0.50  
   Firewall(config-network-object)# nat (inside,outside) static 10.0.0.250
   Firewall(config-network-object)# exit
   Firewall(config)# same-security-traffic permit inter-interface
   Firewall(config)# wr mem 
   ```

8. **Configure Access Control List (ACL)**:

   ```shell
   Firewall(config)# access-list allow_icmp_inside extended permit icmp 10.0.0.0 255.255.255.0 192.168.0.0 255.255.255.0
   Firewall(config)# access-group allow_icmp_inside in interface inside
   Firewall(config)# wr mem
   ```

   ### Configure VLANs for Firewall (Cisco ASA 5510)
      1. Configure Subinterfaces on Cisco ASA Firewall for VLANs 
         ```shell
         Firewall(config)# interface e0/1.10
         Firewall(config-subif)# vlan 10
         Firewall(config-subif)# nameif Trusted
         Firewall(config-subif)# security-level 100
         Firewall(config-subif)# ip address 10.0.10.1 255.255.255.0
         Firewall(config-subif)# exit

         Firewall(config)# interface e0/1.20
         Firewall(config-subif)# vlan 20
         Firewall(config-subif)# nameif Guest
         Firewall(config-subif)# security-level 50
         Firewall(config-subif)# ip address 10.0.20.1 255.255.255.0
         Firewall(config-subif)# exit

         Firewall(config)# interface e0/1.30
         Firewall(config-subif)# vlan 30
         Firewall(config-subif)# nameif Admin
         Firewall(config-subif)# security-level 100
         Firewall(config-subif)# ip address 10.0.30.1 255.255.255.0
         Firewall(config-subif)# exit

         Firewall(config)# interface e0/1.40
         Firewall(config-subif)# vlan 40
         Firewall(config-subif)# nameif IOT
         Firewall(config-subif)# security-level 100
         Firewall(config-subif)# ip address 10.0.40.1 255.255.255.0
         Firewall(config-subif)# exit
         ```
      2. Configure DHCP pools on Cisco ASA Firewall for VLANs
         ```shell
         Firewall(config)# dhcpd address 10.0.10.100-10.0.10.200 Trusted
         Firewall(config)# dhcpd dns 8.8.8.8 8.8.4.4 interface Trusted 
         Firewall(config)# dhcpd enable Trusted
         Firewall(config)# dhcpd address 10.0.20.100-10.0.20.200 Guest 
         Firewall(config)# dhcpd dns 8.8.8.8 8.8.4.4 interface Guest
         Firewall(config)# dhcpd enable guest
         Firewall(config)# dhcpd address 10.0.30.100-10.0.30.200 Admin 
         Firewall(config)# dhcpd dns 8.8.8.8 8.8.4.4 interface Admin
         Firewall(config)# dhcpd enable Admin
         Firewall(config)# dhcpd address 10.0.40.100-10.0.40.200 IOT 
         Firewall(config)# dhcpd dns 8.8.8.8 8.8.4.4 interface IOT 
         Firewall(config)# dhcpd enable IOT
         ```
      3. Configure NAT on Cisco ASA Firewall for VLANs
         ```shell
         Firewall(config)# object network Trusted-NAT
         Firewall(config-network-object)# subnet 10.0.10.0 255.255.255.0 
         Firewall(config-network-object)# nat (Trusted,outside) dynamic interface
         Firewall(config-network-object)# exit
         Firewall(config)# object network Guest-NAT 
         Firewall(config-network-object)# subnet 10.0.20.0 255.255.255.0
         Firewall(config-network-object)# nat (Guest,outside) dynamic interface
         Firewall(config-network-object)# exit
         Firewall(config)# object network Admin-NAT 
         Firewall(config-network-object)# subnet 10.0.30.0 255.255.255.0 
         Firewall(config-network-object)# nat (Admin,outside) dynamic interface
         Firewall(config-network-object)# exit
         Firewall(config)# object network IOT-NAT 
         Firewall(config-network-object)# subnet 10.0.40.0 255.255.255.0 
         Firewall(config-network-object)# nat (IOT,outside) dynamic interface
         Firewall(config-network-object)# exit
         ```
      4. Make Guest network reachable from outside network
         ```shell
         Firewall(config)# object network myWebServ
         Firewall(config-network-object)# host [static ip on that VLAN] # In our case it's 10.0.20.113
         Firewall(config-network-object)# nat (Guest,outside) static interface service tcp www www 
         Firewall(config-network-object)# exit
         ```

      5. Configure ACL on Cisco ASA Firewall in general and for "myWebServ" network object 

         ```shell
         Firewall(config)# access-list inside_access_in extended permit object-group DM_INLINE_PROTOCOL_1 any any 
         Firewall(config)# access-list inside_access_in_1 extended permit object-group DM_INLINE_PROTOCOL_2 any any 
         Firewall(config)# access-list OUTSIDE_IN_ACL extended permit icmp any any echo-reply 
         Firewall(config)# access-list allow_icmp_inside extended permit icmp 10.0.0.0 255.255.255.0 192.168.0.0 255.255.255.0 
         Firewall(config)# access-list allow_icmp_inside extended permit ip any any 
         Firewall(config)# access-list outside_access_in_1 extended permit object-group TCPUDP any object Guest-NAT eq www 
         ```


## Configuring Switch (Cisco Catalyst 3550 SERIES)

   * **Configuring VLANs for Cisco Switch**
      1. Create Different VLANs inside of Cisco Switch VLAN database.
         ```shell
         Switch#vlan database
         Switch(vlan)# vtp server
         Switch(vlan)# vlan 10 name Trusted
         Switch(vlan)# vlan 20 name Guest
         Switch(vlan)# vlan 30 name Admin
         Switch(vlan)# vlan 40 name IOT 
         ```
      2. Assign trunk port to interface fa0/31 to communicate with the firewall
         ```shell
         Switch(config)# int fa0/31
         Switch(config-if)# switchport trunk encapsulation dot1q
         Switch(config-if)# switchport mode trunk
         Switch(config-if)# switchport trunk allowed vlan all 
         ```
      3. Assign the rest of interfaces to specific VLANs
         ```shell
         Switch(config)#int fa0/1
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 10
         Switch(config)#int fa0/2
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 10
         Switch(config)#int fa0/3
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 10
         Switch(config)#int fa0/4
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 10
         Switch(config)#int fa0/5
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 10
         .
         .
         .
         .
         Switch(config)#int fa0/16
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 40
         Switch(config)#int fa0/17
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 40
         Switch(config)#int fa0/18
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 40
         Switch(config)#int fa0/19
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 40
         Switch(config)#int fa0/20
         Switch(config-if)#switchport mode access
         Switch(config-if)#switchport access vlan 40
         ```
 

## Configuring Access Point (Cisco Aeronet 3502 series)
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
      ap# enable
      ap# configure terminal
      ap# interface BVI1
      ap# ip address <IP_ADDRESS> <NETMASK>
      ap# exit
      ```

      Replace `<IP_ADDRESS>` with the desired IP address (e.g., `192.168.1.2`) and `<NETMASK>` with the subnet mask (e.g., `255.255.255.0`).

   7. **Save Configuration**:
      - To save your configuration, use the following command:
      
      ```
      ap# write memory
      ```

      This command saves your changes to the device's configuration.

   8. **Exit PuTTY**:
      - Type `exit` to log out of the access point's CLI, and then close the PuTTY session.

   9. **Verify Configuration**:
      - You can verify the configuration by trying to access the access point's web GUI using the new IP address you configured for the BVI1 interface.

      That's it! You have configured the Cisco Aironet access point's BVI1 interface using PuTTY. Be sure to follow best practices for securing your access point and changing default login credentials for improved security.


## Configuring Servers

**Linux server:**
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

**Windows Server:**

   1. Connect the ethernet cable to the laptop
   2. Assign static IP to Windows Server. 
      - Control Panel --> Network and Internet --> Network and Sharing Center --> Select the Ethernet and right click on Priorities --> Choose Internet Protocol version 4 --> choose properties and assign the required IP, gateway and DNS.
   3. To download browser of your choice, open Internet Explorer go to Tools which on the right top corner
   4. Choose Internet Options
   5. Go to security panel which is on top right of the new window
   6. Choose Custom Level
   7. Find File Downloads and enable it
   8. Download the browser of your choice.