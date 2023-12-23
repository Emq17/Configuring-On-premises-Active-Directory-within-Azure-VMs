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







  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/040befb7-83e7-4ce8-a978-a971a0463949)

- Click `Member Of` tab
- Click `Add`
- Type in **Domain Admins**
- Click `Check Names`
- Click `OK`
- Click `Apply` then `OK`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/f2a3cbcb-80b6-40fd-bb9b-6a5f93463907)

- Log off of the Domain Controller
- logon into the VM again
- Set `Username` to **mydomain.com\jane_admin**
- Set `Password` to **Password1**

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/594e49a8-7b18-485d-9a15-680091a4e1a4)

<h3>&#9318 Join Client to the Domain</h3>

- Return to Azure Portal
- Go to Client-01 Virtual Machine Overview Page
- Click on `Networking`
- Click on `Network Interface`

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/e799f02c-4ba3-40a7-b72f-4800f89b7930)

- Click `DNS servers`
- Click `Custom` option
- Input DC's Private IP Address
- Click `Save`
- Click `Overview`
- Click `Restart`
- Log into Client-01 Virtual Machine

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/8e1a5578-aa6f-47f2-9796-fceebdcdf5fb)

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/f8e1036a-9916-4237-a6c4-7408daa570c8)

- Right click the Windows Button
- Click `System`

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/4155d69b-b3d4-4b3c-a7ac-1b6a7e63c1bf)

- Click on `Rename this PC (advanced)`

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/e696c3d8-8cc5-4a8c-8355-e01026d99170)

- In the System Properties Window, Click `Change`
- Under `Member of`, Click `Domain`
- Type **mydomain.com**
- Click `OK`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/3bca3ba5-8405-44cd-8ac7-afdf748a551c)

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/f6471f2f-8404-4ab8-b4f6-415977c3fc3e)

- Enter **mydomain.com\jane_admin**
- Enter **Password1**
- Click `OK`
- A restart will begin

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/24754541-599a-46bd-bf97-248396735de3)

- Use Remote Desktop Connection (RDP) to log into Client-01 Virtual Machine
- Login using **mydomain.com\jane_admin** Account

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/d50c3b31-b7d2-4598-88e9-cf0e517f798c)

<h3>&#9319 Setup Remote Desktop for non-administrative users on Client-01</h3>

- Right Click the Windows Button
- Click `System`
- Click `Remote Desktop`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/e944ba21-769a-4f0d-afd5-9f6b312be69b)

- Click `Select users that can remotely access this PC`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/f8d04fe1-d77b-4c10-ad32-68bbb7d41f24)

- Click `Add...`
- Type in **Domain Users**
- Click `Check Names`
- Click `OK`
- Click `OK` again
   >**Note***
   >_Client-01 is now a normal, non-administrative user_

   ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/d0c950af-941f-48e0-b9b5-4664523409d7)
![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/260fa179-e8e0-4bee-9617-eb9a506b7a9c)

<h3>&#9320 Creating 10,000 users and logging into Client-01 with one of them</h3>

- Return to DC-1 as **jane_admin**
- Search on Windows `Powershell_ISE`
- Right Click and Select `Run as administrator`

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/c7603514-1a9b-4c3c-9063-53360d0f70d6)

- At the top menu, Click `New Script`
- Using this premade script, copy and paste the code into the box

<details close>
  <summary> PowerShell Script </summary>
  <p>
 # ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $fisrtName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
}
    </p>
</details close>


>**Note***
>This script will create 10,000 accounts, using the password **Password1**, at it's set path: _EMPLOYEES


![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/23db777a-55c5-4d86-b067-3cbef62e7b8f)


![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/d7074419-2b61-4bb5-84b9-55130f65ce33)

- Open `Server Manager`
- Open `Active Directory Users and Computers`
- Open `mydomain.com`
- Open `_EMPLOYEES` folder
- You can see the created users

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/a9cca5a6-0315-4c4f-bb8e-a95176b93c4f)

- Log into Client-01 with one of the randomly created user (I used **basaho.fap**)

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/4979210e-504d-432b-8910-7ea8a848355b)

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/c88666f2-157b-42cb-bc52-a6808743b50b)

- Back to DC-1
- Right Click your user's account
- Click `Properties`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/699b054a-d527-40d6-a5c4-5454c9621bc5)

- Click `Account`
- Now you can do actions from **Unlock Accounts, Reset Passwords, and More!!!**

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/1b919d51-d9de-4f72-97d9-c77031f1b2b5)

---
<h1>CONGRATS!!! YOU'RE DONE!!!</h1>
