# Network Tools and Technologies
My project of building a small business network

## STAGE 0
In this stage, we were given a network with a WAN-Cloud and WAN-Switch to represent our connection to the internet

![Stage 0 Network Topology drawio](https://github.com/jabex134/NTT/assets/171542727/48c60791-7864-47b2-96d9-6a6e3999cb69)

## STAGE 1

In this stage, we started to build out the network infrastructure for our small business. 
**Step 1:** We start off by configuring the LAN network on the firewall through the CLI over the console interface. On a physical firewall the LAN interface is usually preconfigured however on a virtualized firewall, we will have to manually configure it. The client requested specific IP addresses. 
Per the clients request, we will configure 10.128.0.0/24 as the LAN network.
The LAN gateway will be 10.128.0.1/24.
The LAN interface is named port2 on the FortiGate firewall. 
We used the following commands:
![image](https://github.com/jabex134/NTT/assets/171542727/b289b77a-38ae-4273-b9b0-a171eafaca86)

Then verified that the configuration is correct:

![image](https://github.com/jabex134/NTT/assets/171542727/8a76fda7-543e-4fc4-9db5-f6658d838c37)

Then we had to configure the DHCP server for the LAN interface. The firewall will perform DHCP services for devices on the LAN network. We were told to set the DHCP pool scope to 10.128.0.[100-199].

![image](https://github.com/jabex134/NTT/assets/171542727/586a0962-47a1-48f3-a4c5-e6ed061e2786)

Then verified that the configuration is correct:
![image](https://github.com/jabex134/NTT/assets/171542727/82920e19-025a-4917-9bbc-7ef0f55b08bf)

**Step 2:** Next, we added a Win10 workstation to the LAN network. We had to verify the Win10 workstation has leased a DHCP address from the LAN network with the following: 
*valid IP range: 10.128.0.[100-199]/24*
*gateway: 10.128.0.1*
*DHCP server: 10.128.0.1*, and did so by running an *ipconfig /all* in cmd prompt:
![image](https://github.com/jabex134/NTT/assets/171542727/dc4495a5-14f8-445c-9a38-f5a5f2f0609e)

Then we ping remote destinations to test LAN (10.128.0.1), WAN (8.8.8.8), and DNS (google.com) network connectivty.
The LAN access should work however the WAN and DNS failed.
![image](https://github.com/jabex134/NTT/assets/171542727/3811bb7e-1523-47e2-90fb-e9b7c98db0fc)

**Step 3:** Then we connected to the firewall GUI from the Win10 workstation. We opened a browser and went to *http://10.128.0.1/* 
http://ntt-wiki.dvg.local/lib/exe/fetch.php?w=1000&tok=21ffe8&media=gns3-ntt-lab-stage1-instructions-fw-gui1.png
and were redirected to the dashboard:
![image](https://github.com/jabex134/NTT/assets/171542727/f7e18b7c-4f3c-487a-abaa-d18b02982d9a)
We then made the following system changes:*hostname = firewall; timezone = GMT -6:00 Central Time (US & Canada); setup device as local NTP server = enabled; list on interfaces = port2, port4; idle timeout = 60; auto file system check = enabled*
We applied the changes, backupped the configuration, and rebooted the firewall.

**Step 4:** The final step of Stage 1, we completed the network setup through the firewall GUI. 
-We opened the web browser on the Win10 workstation and connected to the firewall GUI: *http://10.128.0.1/*
-Backed up the firewall Config as described in the Connect to the firewall GUI instructions.
-Configured network interfaces: Per the client's request we needed to configure:
    -10.128.0.0/24 as the LAN network
    -10.128.99.0/24 as the GUEST network
    -10.128.10.0/24 as the DMZ network
-And edited the network interfaces with the following commands: 
port1 WAN
      edit port1
          alias = WAN
          role = WAN
      #APPLY-THE-CHANGES
port2 LAN
      edit port2
          alias = LAN
          role = LAN
          create address object matching subnet = enabled
          administrative access = https, http, ping, ssh
          dns server = same as interface IP
          expand advanced
              ntp server = local
      #APPLY-THE-CHANGES
port3 GUEST
      edit port3
          alias = GUEST
          role = LAN
          ip/network mask = 10.128.99.1/24
          create address object matching subnet = enabled
          administrative access = ping, ssh
          dhcp server = enabled
          dns server = same as interface IP
          expand advanced
              ntp server = local
      #APPLY-THE-CHANGES
port4 DMZ
      edit port4
          alias = DMZ
          role = DMZ
          ip/network mask = 10.128.10.1/24
          create address object matching subnet = enabled
          administrative access = ping
      #APPLY-THE-CHANGES
-We then enabled DNS under: System > Feature Visibility
  dns database = enabled
  #APPLY-THE-CHANGES
  
-Configure the firewall system DNS
  dns servers = specify
  primary dns server = 8.8.8.8
  secondary dns server = 1.1.1.1
  #APPLY-THE-CHANGES
  
-Configure Network DNS settings under: Network > DNS Servers
LAN DNS
  Create New
  interface = LAN(port2)
  mode = recursive
  #APPLY-THE-CHANGES
GUEST DNS
  Create New
  interface = GUEST(port3)
  mode = recursive
  #APPLY-THE-CHANGES
DMZ DNS
  Create New
  interface = DMZ(port4)
  mode = recursive
  #APPLY-THE-CHANGES

-We then created service objects
LAN services
  Create New > Service Group
  name = LAN-services-group
  members = ALL_ICMP, NTP, RDP, SSH, Web Access, Windows AD
DMZ services
  Create New > Service Group
  name = DMZ-services-group
  members = ALL_ICMP, FTP, RDP, SSH, Web Access

-Then Configured Firewall rules
LAN-to-WAN policy
  Create New
  name = LAN-to-WAN
  incoming interface = LAN
  outgoing interface = WAN
  source = port2 address
  destination = all
  service = all
  NAT = enabled
  #APPLY-THE-CHANGES  
Note: NAT is enabled for outbound traffic.
DMZ-to-WAN policy
  Create New
  name = DMZ-to-WAN
  incoming interface = DMZ
  outgoing interface = WAN
  source = port4 address
  destination = all
  service = all
  NAT = enabled
  #APPLY-THE-CHANGES

LAN-to-DMZ policy
  Create New
  name = LAN-to-DMZ
  incoming interface = LAN
  outgoing interface = DMZ
  source = port2 address
  destination = port4 address
  service = DMZ-services-group
  NAT = disabled
  #APPLY-THE-CHANGES

Note: NAT is disabled for local traffic.
DMZ-to-LAN policy
  Create New
  name = DMZ-to-LAN
  incoming interface = DMZ
  outgoing interface = LAN
  source = port4 address
  destination = port2 address
  service = LAN-services-group
  NAT = disabled
  #APPLY-THE-CHANGES
Note: NAT is disabled for local traffic.
WAN-to-DMZ policy
  Create New
  name = WAN-to-DMZ
  incoming interface = WAN
  outgoing interface = DMZ
  source = all
  destination = port4 address
  service = DMZ-services-group
  NAT = disabled
  #APPLY-THE-CHANGES

Then Backed up the firewall configuration. At the end of Stage 1, our network topology looked like this:
![Stage 1 Network Topology drawio](https://github.com/jabex134/NTT/assets/171542727/feaa998c-8b08-487c-a453-c721185b93e9)



























