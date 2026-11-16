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

## **CHAPTER 4: Software Based Installation - Impero**

In my virtualised environment, I would like to use software that allows me to remotely control multiple client & server computers at once, using a GUI. The requirements for this system are:
Has a GUI / Can be accessed by a set of users within a group / Control multiple devices to perform tasks at once (e.g. run a batch file).

For this, case, I will be using software called 'Impero Education Pro' which will allow me to meet the requirements. Impero is quite an old software program, so it does need .NET FrameWork 3.5 to be installed on the devices that will be running Impero. Additionally, because of the age of the software, I will be using an Outdated operating system as the server (Windows Server 2008R2 Enterprise with GUI). As the network cannot connect to the internet, I believe that this network is safe, as malware cannot enter the network.

First, I will install .NET FrameWork 3.5 on Windows Server 2008R2. I can do this by using the Installation ISO File, which contains all the relevant packages for .NET 3.5 in the D:\Sources\Sxs\ Folder

Now that the pre-installation requirement is complete, I will first install Impero Server - which will manage the software across the entire network. The Impero server serves as the principal point of communication, allowing for Users & Computers to sync with Impero and for Impero to connect to Active Directory Domain Services.

For this installation of Impero Server, I have created a virtual hard disk, with a size of 75GB. This is to allow for Impero to have its files stored on a seperate location from the Windows Installation drive. In the event that the Windows Server is unavalible, the Hard disk containing all the relevant Impero Files can easily be recovered. 

This method also allows for easier backups to be made of the Server. In the even that the server is unrecoverable, Windows Server can be re-installed, with Impero files ready from either a backup file server or the Virtual Hard disk, allowing for minimal downtime, and a seamless recovery (Impero will recognise its log & configuration files, allowing the server to resume operations where the server was left before the disaster.


The Impero Server Installation has now concluded. I will configure it so that we can meet the first few requirements.
1. Can be accessed by a set of users within a group.

I will head back over to DC01, and create a new Group within the 'Departments' Organisational Unit called: 'Impero Administrators' and 'Impero Users'

Additionally, I have created a user called "g_Impero" setting the member of as 'Impero Administrators' and denying local logons.

I have also changed the 'IT Administrators' group to be a member of the 'Impero Administrators' group, so that IT Admins can have easy access to use the Impero Software.

Heading back to Impero Server, I will change the directory services configuration. I have specified the domain controller as 'DC02.littlenetwork.local' as the Fully Qualified Domain Controller, 'g_Impero' as the user that can contact the directory for authorisation as well as the password for that user.

Finally, I will change the group policy settings (within Impero). I have set the group 'Impero Administrators' to have full control over the network, Impero Server access as well as the full admin console for the client. And I have set the group 'Impero Users' to have limited access over the network to manage computers.

Now that the server side configuration is complete, I will install the Impero Client on the server, which has completed successfully.

I have also enabled a shared area on the Virtual Hard Disk containing the Impero Installation files and .NET Framework 3.5 files for clients.

Lastly, I have installed the Impero client on a test machine connected to the network. I tried to control that test machine from the server client, and the test passed all checks.

This concludes the Chapter on Impero's Installation! Next chapter will include Deployment of Windows!

___________________________________________________
