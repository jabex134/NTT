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

**Step 4:** The final step of Stage 1, we complteted the network setup through the firewall GUI. 




























