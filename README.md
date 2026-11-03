# Virtualised-Windows-Domain-Environment

In this project, I have created a fully working Windows Domain network, through a virtualised Hyper-V Environment. 

## **Introduction**

For this project, I will be using my own server running Hyper-V. I have already created a Virtual Switch that is internal only. This way, the virtual machine clients can interact with each-other, while my own server can connect to the network and provide tools through a samba shared folder. 

___________________________________________________

## **CHAPTER 1: DC01 Installation, Domain Creation, DNS & DHCP**

To Begin, I created an instance of Windows Server 2022, configuring a static IP (IP: 192.168.0.2, Subnet: 255.255.255.0, Gateway: 192.168.0.1) and assigned the DNS server as 127.0.0.1 (the server itself).

Next, I configured the local server settings (Installing updates, Changing the Server name to ("DC01" to represent that the server belongs as the Schema master, Operations Master, PDC & Infrastructure Master of the domain, all prior to the domain's creation)

After the restart had completed to change the Server name, I ran the "Add new Roles & Features" option in the Server Manager, selected the Active Directory Domain Services (AD DS), beginning the creation process of the domain. Alongside installing AD DS, you must also install and Configure DNS (the Domain Naming System) as well as DHCP (Dynamic Host Connection Protocol) - More in this later!

I began to create the domain first, promoting DC01 to a new domain controller in a new forest! For this project, I decided to call the domain: thelittlenetwork.local then setting the Directory Services Restore Mode (DSRM) Password, the NETBIOS name, the Sysvol and NTDS folders, Passing the Prerequisites Checks & Installing a Domain!

Post Restart, I can see that the installation has been successful so far, as I no longer login as the built-in administrator, rather the Domain Administrator (LITTLENETWORK\Administrator). 

Now I will begin the work on the DHCP Configuration. As most of you will likely already know, DHCP handles requests for an IP Address from a specific range, from a server.
Launching the Configuration Wizard, I am using the Domain Admin's credentials to authorise action on the domain to commit DHCP to AD DS.

I am now in the DHCP manager, and creating a new scope, with the range of IP addresses to be 192.168.0.12 to 192.168.0.254 (Allows for 11 Static IPs for servers, and the default gateway to be reserved) with subnet mask as 255.255.255.0. I will be replicating this in the exclusions and delays tab to ensure that no machines can recieve reserved IPs, Setting the Default Gateway, and skipping the WINS server deployment.

Concluding the DHCP, DNS and AD DS deployment of DC01, I believe that we can now connect DC02 to the newly created Domain. For this project, DC02 is running Server 2022, has already been setup with a Static IP (192.168.0.3) and has the Name DC02. DC02 is not presently connected to the little network at all.

___________________________________________________

## **CHAPTER 2: The Other Domain Controllers (DC02, DC03, RDC04)**

Now that DC01 has been configured to serve as the head of the domain, with DNS & DHCP configured, we can connect other servers to promote them to domain controllers!

Starting with DC02, in Server manager, I change the local server properties from the default WORKGROUP option to domain, entering: thelittlenetwork.local as domain name, supplying the Default Domain Administrator account. DC02 Reboots and has joined the domain successfully. The server is shown in AD U&C (Users & Computers) under the Computers OU. It will later change to the Domain Controllers OU once promoted to DC.

To save the reader's time, I will not detail all the steps to join the other Domain Controllers. All the other DCs connected to the domain successfully, had DNS installed, and were promoted to Domain Controllers!

DC04 is not a standard DC, I have selected during dcpromo that the server be a Read Only DC (RDC). This is because I would like to simulate a server being deployed in a local environment, where a full domain
controller may not be neccessary 

(Imagining that the Little Network is based in Glasgow, with a small remote office in Cardiff, a RDC would be implemented here, to allow for the replication of users and OUs, but not changes as DC04's purpose would be to only serve on-prem logons to the network based in Cardiff via a Windows VPN to another server based in the Glasgow Server Room)

___________________________________________________
