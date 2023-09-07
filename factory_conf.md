The commands we used to reset to factory default settings before starting

**Important Note**: 


Linux server login creds: 
- username: group5
- password: group5 

dhcpd address range inside: 10.0.0.100 - 10.0.0.200 



1. **Firewall (Cisco ASA 5510)**:

   ```shell
    G5-ASA-5510> enable
    G5-ASA-5510# conf t
    G5-ASA-5510(config)# configure factory-default
    G5-ASA-5510(config)# wr mem
   ```

2. **Switch (Cisco Catalyst 3550 SERIES)**:

   ```shell
    Switch> enable
    Switch# conf t
    Switch(config)# configure factory-default
    Switch(config)# no spanning-tree vlan 1
    Switch(config)# wr mem
   ```

2. **Access Point (Cisco Aeronet 3502 series)**:

   ```shell
    ap> en
    ap# write erase
    ap# reload
   ```