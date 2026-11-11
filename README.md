# Virtualised-Windows-Domain-Environment

In this project, I have created a fully working Windows Domain network, through a virtualised Hyper-V Environment. 

## **Introduction**

For this project, I will be using my own server running Hyper-V. I have already created a Virtual Switch that is internal only. This way, the virtual machine clients can interact with each-other, while my own server can connect to the network and provide tools through a samba shared folder. 

___________________________________________________

## **CHAPTER 1: DC01 Installation, Domain Creation, DNS & DHCP**

To Begin, I created an instance of Windows Server 2022, configuring a static IP (IP: 192.168.0.2, Subnet: 255.255.255.0, Gateway: 192.168.0.1) and assigned the DNS server as 127.0.0.1 (the server itself).

Next, I configured the local server settings (Installing updates, Changing the Server name to ("DC01" to represent that the server belongs as the Schema master, Operations Master, PDC & Infrastructure Master of the domain, all prior to the domain's creation).

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
controller may not be neccessary. 

(Imagining that the Little Network is based in Glasgow, with a small remote office in Cardiff, a RDC would be implemented here, to allow for the replication of users and OUs, but not changes as DC04's purpose would be to only serve on-prem logons to the network based in Cardiff via a Windows VPN to another server based in the Glasgow Server Room).

___________________________________________________

## **CHAPTER 3: Configuring Active Directory, Network Shares and Group Policy**

All the DCs have successfully been connected to the domain and are functioning as expected!

Now is the exciting part I've been waiting for, which is to start the initial configuration of Active Directory Users & Computers. Firstly, I will be creating an Organisational Unit directory system. At the Top of the directory, underneath the Domain, I have created an OU callled The Little Network, followed by two Subfolders: Departments and Devices.

Inside the Departments OU, are further Sub-folders for the departments inside the Little Network, with a respective group being created. For example, I have created a Finance OU and inside that OU is a group called 'Finance.' This is to allow for an easier way to manage the Finance Department in Group Policy (e.g. to assign a network share that only the finance department and IT have access to view).

Before I start creating any users, I have already configured a 50GB Virtual Hard Disk for the file server I will be using for network shares. I created a shared area over the network, so that files are actually stored on this location >>> \\TLN-FILE-01\UserAreas\ and \\TLN-FILE-01\Departments\

Now I can head back over to Active Directory Users & Computers, where I will create a User that has Admin Privlidges across the entire network:
Username: g_admin (Global Administrator), Setting a secure password, Right-click & selecting 'Member of' I will now add the following groups to this user: Domain Admins; Enterprise Admins; Schema Admins; IT Administrators

Additionally, I have created the group IT Administrators and assigned it as a member of: Administrators; Domain Admins; Enterprise Admins and Schema Admins to make it easier to assign admin privlidges rather than add lots of individual roles as with the g_admin user.

Now that the user's roles have been taken care of, I need to ensure that the user's files are avalible across the network, regardless of which computer is logged into. I will head over to the Profile tab and edit the fields there. I am changing the Profile Path to use the share we created earlier: \\TLN-FILE-01\UserAreas\ and add the variable %username% to store the username, instead of having to manually type out (which is prone to human errors), doing the same for the Home Folder connect using drive letter H:\.

I can now repeat this process for other departments, by creating a user in the Finance Department, Assigning their primary group as: Finance, the profile area and home folder respectively. To make this even easier to create new users within the various departments, we can now right click any user and select the 'Copy' feature, which will replicate the 'Profile' and 'Member Of' configurations!

Lastly, I will configure a default domain group policy object. I am keeping this policy relatively simple - I will only edit the Default Domain Policy to configure the lockscreen and desktop wallpapers to a specific location.
Respectively, I will go into: Computer Configuration -> Policies -> Administrative Templates -> Personalization -> Force a specific default lock screen image. I have enabled this feature and set the server to point the computers to the custom wallpaper stored at \\TLN-FILE-01\Resources\Wallpapers\LockscreenDefault.png and repeated this process for the default wallpaper for the Users under the User Configurations.

All I have to do is Enable the policy and run 'gpupdate /force' via cmd on the rest of the computers, restart them for the changes to take effect, and the policy should be configured!

___________________________________________________
