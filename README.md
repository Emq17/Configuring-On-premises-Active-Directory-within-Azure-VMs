<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Configuring-On-Premises-Active-Directory-Within-Azure-VMs</h1>
There are two different types of Active Directory essentially. There's the type that is a software that you install on a computer you own (on-premises) and then there is another called Azure Active Directory that is a serverless software as a service directly in the cloud. This walkthrough that I created demonstrates and outlines the implementation of on-premises Active Directory within Azure virtual machines. This exercise allows us to explore deeper, enhancing our understanding of its various components.<br />


<h2>Environments and Technologies</h2>

- Microsoft Azure Subscription
- Microsoft Remote Desktop (Mac)
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems</h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Deployment and Configuration</h2>

<h3>Creating the Domain Controller Virtual Machine</h3>

A domain controller is a type of server or computer that basically has Active Directory installed on it

- Create an Azure Virtual Machine
  - Resource Group: Create new --> "AD-Lab"

![Screen Shot 2023-12-22 at 2 30 09 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/4451e95c-3374-459d-8cad-6d21799d14a0)

  - Virtual Machine Name: DC-1 
  - Region: (US) West US 3
  - Image: Windows Server 2022 Datacenter: Azure Edition - x64 Gen2
  - Size: Standard_E2s_v3 - 2 vcpus, 16 GiB memory (we want to use something that at least has 2 virtual cpu's. If you have 1 it's going to be kind of slow)

![Screen Shot 2023-12-22 at 2 36 03 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/cccd9481-ac90-406d-ad26-a826cc1ce30d)
  
  - Username: labuser
  - Password: Password1234
  - Click "Review + create"
  - Open up a notepad & make sure to keep credentials just so you don't forget it

![Screen Shot 2023-12-22 at 2 42 01 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/07e7697a-10e6-46cf-91bd-5841d7d5b766)

  - Once Validation passes, click "create"

![Screen Shot 2023-12-22 at 3 13 10 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/6e4af4ca-0a53-4974-ad1a-eae8c0915e93)

<h3>Creating a Client Virtual Machine</h3>

- Follow the same steps as before but with the following:
  - Resource group: Same Resource Group DC-1 is using (AD-Lab)
  - Virtual Machine Name: Client-1
  - Region: Same region ((US) West US 3)
  - Image: Windows 10 Pro, version 22H2 - x64 Gen2
  - Size: Standard_E2s_v3 - 2 vcpus, 16 GiB memory

![Screen Shot 2023-12-22 at 3 32 32 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/248ded43-1ca1-48a8-8bc8-63e32f755493)

  - Username: labuser
  - Password: Password1234
  - Keep note of these credentials
  - Check "Licensing" box
  - Click "Next : Disks >" until you see the "Networking" tab

![Screen Shot 2023-12-22 at 3 33 31 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/384e394d-fb33-497b-9b49-60d8b3722a9f)

  - Use the same Virtual Network as our Domain Controler (DC-1-vnet) - should be automatically created when deploying the first Virtual Machine (if you don't see it your first VM is not finished deploying or you might have to refresh the page)
  - Subnet: Default 
  - Click "Review + create"
  - Once Validation passes, click "Create"

![Screen Shot 2023-12-22 at 3 35 16 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/87374171-15ff-445c-88ee-614ada8a61b4)


- Set the Domain Controller's NIC Private IP address to be static
  - Go back to your DC-1 Virtual Machine

![Screen Shot 2023-12-22 at 3 40 32 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/5455e473-dfad-4b20-b721-78154e1f02cf)

  - On the left side under "Settings" click "Networking" then click the link next to "Network Interface:"

![Screen Shot 2023-12-22 at 3 41 54 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/86e5cf82-3ee9-4295-9301-8f6258acbe24)

  - On the left under "Settings" click "IP configurations"
  - Choose "ipconfig1"
  - Change Allocation to Static instead of Dynamic (so that it doesn't change regardless if we turn our computer off/on which will allow users to login using the domain name without worry)

![Screen Shot 2023-12-22 at 3 45 55 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/dff97893-3ecd-4c51-a8ff-924e30071d3a)


<h3>Ensure connectivity between the Client and Domain Controller</h3>

Refer back to [Establishing Virtual Machines with Remote Desktop](https://github.com/Emq17/Establishing-Virtual-Machines-With-Remote-Desktop) in order to Remote Desktop Connect into your Virtual Machines.

- Login to the Client's Virtual Machine
  
![Screen Shot 2023-12-22 at 5 17 56 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/2e1b9433-c839-4c9b-8e12-5cec88883927)

- Grab DC-1's private IP address

![Screen Shot 2023-12-22 at 5 21 15 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/dc35375f-0248-4079-a1cd-fe5271c8ace1)

- Go back to your remote desktop connection for Client-1
- Go to start & open command line

![Screen Shot 2023-12-22 at 5 23 23 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/0b8df5bc-e48a-4f57-ba5f-e2192e5b3e74)

- Ping DC-1's private IP address with "ping -t" (perpetual ping meaning it will ping forever until you stop it)
- As you can see that it times out and fails because DC-1's local windows firewall is blocking ICMP traffic

![Screen Shot 2023-12-22 at 5 30 04 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/2094e59e-32c0-4d40-880a-bdf5f7856a14)

- Log into DC-1's remote desktop connection. Repeat the same steps as usual to connect.
- As you can see after connecting, we should see two separate PC's now. One for Client-1 & another for DC-1

![Screen Shot 2023-12-22 at 5 32 48 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/4890590a-ad99-4711-87b0-ff7c7101945e)

- Inside DC-1 search up "wf.msc" or type in "firewall" and choose "Windows Defender Firewall with Advanced Security"

![Screen Shot 2023-12-22 at 5 36 14 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/4ffc4caa-da4e-43f7-891f-b084a0f10bb6)

![Screen Shot 2023-12-22 at 5 36 41 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/d224132b-6a49-47f9-8b20-204f5639195b)

- Click "Inbound Rules"

![Screen Shot 2023-12-22 at 5 39 10 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/f9ca786c-38a1-4353-a1a5-a970b47f2875)

- Sort by "Protocol"

![Screen Shot 2023-12-22 at 5 41 21 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/22ad8bfc-d496-47d5-b189-f4c671f3f20b)

- Find ICMPv4 Protocols

![Screen Shot 2023-12-22 at 5 42 34 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/f452902e-347d-4b4a-a6aa-4fac199ae911)

- Find both "Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)" Private and Domain Profiles 
- Right click and hit "Enable Rule" to both of them

![Screen Shot 2023-12-22 at 5 48 25 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/225a7a1c-6e07-4641-8706-9e4cd498fe78)

- Now go back to your Client-1 remote desktop connection & observe how after we enabled the ICMP echo requests the ping succeeded and started working in the Command Prompt

![Screen Shot 2023-12-22 at 5 54 59 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/fac3bbf6-2337-4019-86c5-cf6a9a11dcca)

- Now at least we know there is connectivity between the Client and Domain Controller
- Type in "CTRL + C" to stop

![Screen Shot 2023-12-22 at 5 55 25 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/561ad303-334e-4831-b516-a8397468e2bd)

- Next we will install Active Directory Domain Services to DC-1
  - You can type in "hostname" in the Command Prompt if you ever get lost on which remote desktop connection you're in
  - Also Windows 10 won't have "Server Manager" (which is used to add and remove all the necessary server software for Active Directory)
    
![Screen Shot 2023-12-22 at 6 03 14 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/2d2dc39e-3a7b-4b19-b5ad-d6e2f2b8a9ca)

- If you don't see Server Manager you can simply click Start and find it 

![Screen Shot 2023-12-22 at 6 06 25 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/9b5c8fb9-6da2-460c-966e-a11cea27af7c)

- Next click "Add roles and features"
- This is how we install Active Directory

![Screen Shot 2023-12-22 at 6 11 06 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/d776f38c-0423-47b1-9d49-618652a36ef3)

- Click "Next >" until you get to this page and choose "Active Directory Domain Services"

![Screen Shot 2023-12-22 at 6 12 57 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/a7af0ed3-ef95-4226-891f-0ab122767c81)

- Click "Add Features"
- Click "Next >" through the dependencies then hit "Install"

![Screen Shot 2023-12-22 at 6 14 25 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/89802a2a-d3f9-40d9-9b81-4d4bdf4e12d8)

- At the top right of your screen, click the flag icon with a caution symbol. Click "Promote this server to a domain controller"
- This is actually how we finish installing Active Directory and turn the server into a domain controller

![Screen Shot 2023-12-22 at 6 22 55 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/18a4383c-4373-4e19-a0c6-a546ba6341fc)

- Choose "Add a new forest"
- This is where we name the domain whatever we'd like. It's not going to be public. To keep things generic I just chose "mydomain.com" for the Root domain name as it essentially doesn't matter for now.
- Click "Next >"

![Screen Shot 2023-12-22 at 6 26 53 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/fcd5517c-e8a2-4101-80de-4b1105a6e4d4)

- Set Password to "Password1" 

![Screen Shot 2023-12-22 at 6 33 50 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/a755e4bb-9d4f-4a13-9161-da1cbe7da5b8)

- Keep clicking "Next >" and then "Install"
- It should automatically restart once finished & as you can see the remote desktop connection became disconnected

![Screen Shot 2023-12-22 at 6 51 29 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/18ecb7db-c4fc-4198-ab3b-2ebf7adea091)

- Log back into DC-1
  - Username: mydomain.com\labuser
  - Password: Password1234

![Screen Shot 2023-12-22 at 6 49 18 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/81bd439e-57d5-40ef-b373-ec3cd0622d0e)

- Now that our Domain Controller is back online and the Domain is set up and everything, we're going to create a couple Organizational Units (while there are a lot more use cases for them just think of these as folders) inside Active directory and an Administrative User

- There are a couple ways to open up Active Directory
  - You can go to "Tools" and then click "Active Directory Users and Computers"
 
![Screen Shot 2023-12-22 at 6 58 48 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/5f952553-e70e-48a7-8411-c3756ed04302)

  - Or click "Start" and search "Active Directory"  

![Screen Shot 2023-12-22 at 6 59 44 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/3ebe5ed2-3123-4dff-b906-292cb7b4f5ad)

- Once opened, what you're looking at is pretty much the User Interface of what most people call Active Directory

![Screen Shot 2023-12-22 at 7 01 07 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/0e91dbf0-7d5d-4483-8bcd-461ab27f0647)

- Now we will create some organizational units by right clicking "mydomain.com" --> "New" --> Organizational Unit 

![Screen Shot 2023-12-22 at 7 05 32 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/1e7eaf9c-eed1-420b-9148-dc1f56820cbc)

- Create two units named as "_EMPLOYEES" and "_ADMINS" (Using this format helps it easy to filter them out)

![Screen Shot 2023-12-22 at 7 08 05 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/b29b5740-3eb7-4ff8-bc46-728ed1dd29b5)

- Hit "Refresh"

![Screen Shot 2023-12-22 at 7 10 57 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/e6dff8d3-094f-441e-9b98-7c832575356e)

- Now you can see them at the top and easier to deal with

![Screen Shot 2023-12-22 at 7 14 11 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/f674c509-1925-459c-8f98-06f19c546215)

- Next we will go ahead and set up an Admin Account
- Right click "_ADMINS" and click "New", then "User"

![Screen Shot 2023-12-22 at 7 17 44 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/fbc5e6c6-d6e4-488f-9189-18009434c69f)

- I randomly chose the name "jane doe" to continue setting up this admin account
- For "User logon name" typically organizations would have it as "a-jane" which means "admin jane" but this all depends where you work at
- I chose "jane_admin" to keep things generic as usual
- Click "Next >"

![Screen Shot 2023-12-22 at 7 19 09 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/0782a4ec-16e9-46d1-bd47-b31c7da7c787)

- Set up your password: Password1234
- Uncheck the box labeled "User must change password at next logon"
- Check the box labeled "Password never expires"
- Click "Next >" and then "Finish"

![Screen Shot 2023-12-22 at 7 26 14 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/f0856187-52ab-41f3-aa4b-2cd7875443e6)

- So now we have a user account that is created in the folder called "_ADMINS" but that name is just ambiguous. We have to still make it an actual domain admin.
- Right click the account and click "Properties"
- Click "Member Of"
- Click "Add"

![Screen Shot 2023-12-22 at 7 28 48 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/48196fe2-9efa-405f-920e-1106530eb64c)
![Screen Shot 2023-12-22 at 7 30 15 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/d9cb2b01-1278-4a7a-804f-c7e6599e79d3)

- Type in domain then click on "Check Names"

![Screen Shot 2023-12-22 at 7 31 28 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/2c1d9ae6-4015-4dcf-8aea-294bfa5e6aa8)

- Join and choose "Domain Admins" group
- Hit "OK" then "Apply" to make the changes

![Screen Shot 2023-12-22 at 7 32 50 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/eef6f211-8369-440b-8488-d91b71934116)

- If you ever get confused once again on which domain context you're using you can use the Command Prompt
- Type in "whoami" and you'll see which username you're logged in as

![Screen Shot 2023-12-22 at 7 42 47 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/c9e8b498-daed-432a-a54e-4a7db4cd025f)

- Next we will log out of DC-1 and log back in as "mydomain.com\jane_admin"
- You can also use the Command Prompt to do this by typing in "logoff"

![Screen Shot 2023-12-22 at 7 44 58 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/8ad0590d-5e3f-4f76-9ca7-132a43591308)

- Log back into the Domain Controller using the remote desktop connection
- Use "mydomain.com\jane_admin as your Username

![Screen Shot 2023-12-22 at 7 53 43 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/aae15a0c-a121-427c-8bea-73f27cd77bea)

- Once logged in, just for fun, open up Command Prompt again and type in "whoami" to see the changes

![Screen Shot 2023-12-22 at 7 56 11 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/9a926bd4-e753-4e28-9d12-cf0f5430a22c)

- This is a high level overview of the current state of things and what we are going to do next
- Now we have to go back to the Azure Portal and set Client-1's DNS settings to the Domain Controlers Private IP address
  
![Screen Shot 2023-12-22 at 7 59 15 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/cfc39344-2953-4b0b-ba31-c6e198f0350b)

![Screen Shot 2023-12-22 at 7 59 38 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/80f4fdfa-5732-472c-831f-045aa6e7dee2)

- Go back to Client-1's remote desktop connection
- Right click start and choose "System"

![Screen Shot 2023-12-22 at 8 07 10 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/9905568b-5ca3-4714-bd3b-d068ab171b5c)

- Click on "Rename this PC (advanced)" over on the right side
- Then hit "Change"

![Screen Shot 2023-12-22 at 8 09 03 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/77097504-59c4-43ca-a0dc-02b1c33b76d8)

- Remember we are using the public DNS
- If you go to your Command Prompt and type in "ipconfig /all", under "DNS Servers" you can see a public IP address and not our Domain Controllers IP address
- So when you try to join our domain that we just created called "domain.com", you will see an error message stating that the Domain Controller for the Domain cannot be contacted because it tried to reach out to the DNS Server you see in the Command Prompt and tried to ask essentially "what is the domain controller?" and there is none. 

![Screen Shot 2023-12-22 at 8 13 42 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/d2ede669-7cad-4a78-88af-60a49a6dfd2d)

- Moving forward, we need to change our IP settings in the Azure Portal. It's possible to set it inside the Virtual Machine but generally you don't want to change IP settings from within the VM.
- Find DC-1 in your Azure Portal and retrieve and copy its private IP address by going to "Networking" under settings. 

![Screen Shot 2023-12-22 at 8 49 06 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/1a69bfa3-a0a9-4cbc-8ac0-78a97945c715)

- Go back to Client-1.
- Go to "Networking" under settings again.
- Click on the Network Interface 

![Screen Shot 2023-12-22 at 8 54 11 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/b60aded1-91b6-4092-86d7-60fe052bf3a4)

- Under settings, click on "DNS servers"
- Click "Custom" instead of "Inherit from virtual network"
- Paste DC-1's Private IP address then click "Save"

![Screen Shot 2023-12-22 at 9 01 17 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/220b894a-dd06-4e33-8ef5-cd6126660fea)

- Restart Client-1 from the Portal to flush the DNS Cache

![Screen Shot 2023-12-22 at 9 05 46 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/2b04dfb4-f2d8-49c7-89de-a045eeb4ce52)

- Log back into Client-1 through Microsoft Remote Desktop with its new fresh DNS settings
- If needed, grab the public IP address from the portal
- We're going to log in it using "labuser" because it is not joined to the domain quite yet

![Screen Shot 2023-12-22 at 9 08 03 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/f0ef1ae9-ecc7-4ce4-831b-99bad0c1187d)

- When you log into Client-1 and type in "ipconfig /all" into the Command Prompt, you can see the new DNS settings and note that the DNS server is now using DC-1's private IP address

![Screen Shot 2023-12-22 at 9 15 25 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/635f20a9-1743-46d4-9c89-e3dfefc21286)

- If you ping it you can see that it there is a connection

![Screen Shot 2023-12-22 at 9 17 56 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/7f0f1685-4c1b-4fd4-8e64-945030d76bbf)

- Now as we did before, lets attempt to join Client-1 to the domain again
- Instead of getting an error message, we now get this window

![Screen Shot 2023-12-22 at 9 20 35 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/b15743e6-8dbb-41e4-89d0-6ce86b16ba40)

- Typing in "mydomain.com\jane_admin and "Password1234"

![Screen Shot 2023-12-22 at 9 22 13 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/38bee3d4-62ae-459f-8152-0d35834b69f3)

- You should see this window appear right after

![Screen Shot 2023-12-22 at 9 23 37 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/cc25d439-bef6-4e53-a529-c180f44fb310)

- Let the computer restart by following the prompt after closing the "System Properties" window. Click "Restart now"

![Screen Shot 2023-12-22 at 9 26 21 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/cdd32b09-65fe-4013-8d2b-5513069a4653)

- Because Client-1 is now a member of the domain, and "jane_admin" is now an account within the domain, we can essentially log into potentially any computers that are on that domain.
- Lets log in using "mydomain.com\jane_admin to Client-1

![Screen Shot 2023-12-22 at 10 09 11 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/c0da222f-2a71-4151-b9fc-4d45db95243d)

- Once logged in, right click "Start", click "System", to the right click "Remote Desktop", then at the bottom click "Select users that can remotely access this PC"

![Screen Shot 2023-12-22 at 10 12 48 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/15859d8d-5dc1-4f5f-b6dd-a240695c35a7)

- Click "Add"
- Type in "Domain Users" and click on "Check Names"
- So now we have added in all domain users to be able to log into this computer remotely
- Hit "OK"

![Screen Shot 2023-12-22 at 10 14 15 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/f60f60e4-37b8-4fad-97fc-72440c5818e2)

- Go back to your DC-1 remote desktop connection
- Open up Active Directory
- Expand "mydomain.com" on the left side
- Click on "Users" on the very bottom
- Click on "Domain Users"

![Screen Shot 2023-12-22 at 10 17 55 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/1be7e24d-1375-4f02-8b2b-966ed4855da8)

- Click on the "Members" tab
- So anybody within this group should theoretically be allowed to log into Client-1
- You can now log into Client-1 as a normal, non-administrative user now
- Normally you'd want to do this with "Group Policy" that allows you to change MANY systems at once but is a bit out of scope for this demonstration

![Screen Shot 2023-12-22 at 10 19 45 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/0decfe22-785c-4938-be7d-084b868248b2)

- The next thing that we're going to do is to create a bunch of additional users and attempt to log into client-1 with one of the users
  - First thing we do is to log into DC-1 as jane_admin (if you already are, make sure by opening up your command line and typing in "whoami" and "hostname"
 
![Screen Shot 2023-12-22 at 10 24 36 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/29a57373-1c5e-4dbb-afa5-9c7c8bb2ff28)

- Open "Windows Powershell ISE" as an administrator by searching it up in the Start menu (Powershell is a essentially a Windows native scripting language that you can literally do anything with)

![Screen Shot 2023-12-22 at 10 35 01 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/69f2b754-2114-4959-bdfb-2ba338aa96c0)

- Click on "Yes"

![Screen Shot 2023-12-22 at 10 35 47 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/6949c928-8ae6-4aa2-bcc6-90c16355221d)

- Create a New Script

![Screen Shot 2023-12-22 at 10 37 39 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/9019eee4-7a27-4d00-95ec-96f899f0fe6e)

- Using the link below, copy the code and paste it into the Powershell ISE new script box:

  - https://github.com/Emq17/PowerShell-ISE-Script/blob/main/code.txt

- This script will create 10,000 accounts, using the password "Password1", at it's set path: _EMPLOYEES (which we had created earlier in a previous step)

![Screen Shot 2023-12-22 at 10 56 22 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/431663a2-4f21-4e64-a07e-940a27790a1f)

- Click on the "Run Script" icon

![Screen Shot 2023-12-22 at 11 00 04 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/dc5936dd-81ca-4b9e-a2ff-47eb265be0fe)

- You can see it start to create accounts with random names in them essentially just alternating vowels and consonants

![Screen Shot 2023-12-22 at 11 01 34 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/1a148bac-6b23-43ee-b307-e1ca7127bcb7)

- As the users are being created, go back into Active Directory
- If you right click "_EMPLOYEES" and click on "Refresh" you can see the all accounts that were created be transferred into this file

![Screen Shot 2023-12-22 at 11 06 32 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/c32cfab9-72bd-4fee-ad40-f4a3024a9458)

![Screen Shot 2023-12-22 at 11 06 47 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/8d6fa6f6-d9ff-4178-846f-21bd959d8f9b)

- What we can do now is go ahead and click on any random name in the "_EMPLOYEES" file and use it to log into Client-1 with it
  - Double click on a name
  - Go to "Account"
  - Copy its User logon name

![Screen Shot 2023-12-22 at 11 13 08 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/a1316822-c204-4d59-b8c6-30985ee3b65e)

- Go back to Client-1
- Log out
- Then log back into it using "mydomain.com\bax.somu and Password1

![Screen Shot 2023-12-22 at 11 15 21 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/68b85e50-cea7-416f-ab02-d7f719efad26)

![Screen Shot 2023-12-22 at 11 15 42 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/556359f3-04a9-46c7-a5b6-8f1812e2b26d)

- This account has obviously never logged into Client-1 before but because "bax.somu" is on the domain and Client-1 is joined to the domain, Client-1 can look to the domain and see if the user is legit, which then the domain controller says yes, and it lets it log in into Client-1 as a new user.  

![Screen Shot 2023-12-22 at 11 20 20 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/ca3b5714-a617-4871-b14f-c75ec6e89dd9)

- By going to the File Explorer, "This PC", "Windows (C:), and then "Users", you can see every user that logs into this computer (including "bax.somu" up top) because a new folder is going to be created for them

![Screen Shot 2023-12-22 at 11 22 13 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/ec5e35c1-160e-45e1-889d-f3ce246c657c)

- Just to play around with things. Let me show you a demonstration on a user locking themself out by using the wrong password to log in and how to resolve it  
  - Go ahead and remote desktop connect back into DC-1
  - Choose a random user in Active Directory under "_EMPLOYEES"
  - Double click on the name of your choice
  - Click "Account"
  - Copy his account name

![Screen Shot 2023-12-22 at 11 30 16 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/609662f1-1c48-4bec-870c-9f991a395ae0)

- Logoff of Client-1 and log back in using mydomain.com\bob.saqa
- Purposely put in the wrong password multiple times 

![Screen Shot 2023-12-22 at 11 32 10 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/862226fb-252a-4a7d-b13c-b8e72538039c)

- This user should definitley be locked out by now after around ten failed attempts
- Now go back into your DC-1 remote desktop connection
- Find that same user
- Right click and choose "Properties"

![Screen Shot 2023-12-22 at 11 33 40 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/6fa29724-027a-4667-947b-58b1cb7df8e6)

- Click on the "Account" tab
- Check the box beside "Unlock account"
- Then hit "Apply" and "OK"

![Screen Shot 2023-12-22 at 11 38 23 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/12e4bf00-2ad2-4712-bb57-90d90541df0f)

- Also if a user accidentally forgets their password you can resolve this by right clicking their name again
- Click "Reset Password"

![Screen Shot 2023-12-22 at 11 40 38 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/20111fe3-52d4-4692-91b4-313458653009)

- Then you can reset it to something new.
- If its locked you can also check the box labeled "Unlock the user's account"

![Screen Shot 2023-12-22 at 11 42 54 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/0dee89d0-5c5e-4e95-84bb-7d6857a3339e)

- One more thing that you can do that you probably will not be doing in a help desk role is to disable accounts
- Right click the user once again
- Click on "Disable Account"

![Screen Shot 2023-12-22 at 11 45 13 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/70cca0c6-f39e-449a-917d-8d0c403b8c6a)

- Now log out of Client-1 and try to log back into it again using the user's account you have disabled
- You will now see that you have no access with one of these messages below

![Screen Shot 2023-12-22 at 11 47 46 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/3e11df2b-90d6-47c4-b251-361bcdceeb7d)
![Screen Shot 2023-12-22 at 11 49 09 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/01a08850-1956-408b-be56-63ccf78dff8d)

- If you go back to DC-1 then ind the user and right click choosing "Enable Account", this will resolve the issue

![Screen Shot 2023-12-22 at 11 50 56 PM](https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs/assets/147126755/c6f42cdb-75f3-4abe-8807-1100d1226621)

- This covers unlocking, disabling, re-enabling accounts, and resetting passwords because for sure you're going to be doing that a lot in your job
- It might not be directly through Active Directory like this, but more than likey you will be doing some of these tasks in your earlier career 
- Setting up Active Directory is a good skill to have as you may have to do this within your work many many times 

<h1>Congratulations on making it to the end of this comprehensive step-by-step walkthough</h1>

To save you some time, before destroying or closing anything out, please keep in mind that we will be using the same current environment in the next couple of walkthroughs as well.
