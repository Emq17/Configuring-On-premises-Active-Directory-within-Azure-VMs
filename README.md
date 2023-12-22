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

<h2>High-Level Deployment and Configuration Steps</h2>

- Set up 2 Virtual Machines within Azure:
  - **Domain Controller Virtual Machine (Windows Server 2022)**
  - Set the `NIC Private IP Address` to **Static**
  - **Client Virtual Machine (Windows 10)**
  - Set the `Resource group` and `Vnet` as DC's
  - Set the `DNS` Server to use DC's `Private IP Address`
- Be logged into both Virtual Machines using `Remote Desktop Connection (RDP)`
- Enabling Inbound Rules for `Core Networking Diagnostics` within DC's Firewall to ensure connectivity between the Client and DC
- Installing `Active Directory Domain Services` within DC's Virtual Machine
- Create an Admin and User Account in Active Directory
- Link Client's VM to a Domain, log in using the original Admin Account
- Set Remote Desktop Connection for non-administrative users on Client VM
- Create additional users and attempt to log in to the Client VM as one of those users

<h2>Deployment and Configuration Steps</h2>

<h3>&#9312 Creating the Domain Controller Virtual Machine</h3>

>**Note***
>_A domain controller is a type of server that processes requests for authentication from users within a computer domain._

- Create an Azure Virtual Machine
- Set `Virtual Machine Name` to **DC-1**
- `Resource group` will automatically be given
- Set `Region` to **(US) WEST 3**
- Set `Image` to **Windows Server 2022 Datacenter: Azure Edition**
- Set `Size` to **Standard_E2s_v3 - 2 vcpus, 16 GiB memory**
- Set `Username` to **labuser**
- Set `Password` to **osticketPassword1**
- Click licensing checkbox
- Click `Review + create`
- Once Validation is approved, click `create`

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/e7282226-c698-4213-8e9d-52c5ff0ec9eb)

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/7fb2b96b-7a2e-46c9-9134-a5956d7744a2)


<h3>&#9313 Creating a Client Virtual Machine</h3>

- Follow the same steps as before with the following exceptions:
- Set `Resource group` to **DC's Resource Group**
- Set `Virtual Machine Name` to **Client-01**
- Set `Username` to **Clientuser**
- Set `Image` to **Windows 10 Pro, version 22H2 - x64 Gen2**
- Click `Next` until you reach `Networking`

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/5e230bd3-1578-44b3-9bf7-e74a5ebc399c)

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/99d44a4c-d673-4612-adb5-0f65442a8d9f)

- Set `Virtual Network` to **DC-1-vnet**
- Click `Review + check`
- Once Validation passes, click `Create`


<h3>&#9314 Assign Domain Controller's Private IP Address to STATIC</h3>

>**Note***
>_Later in the Tutorial, the DC's NIC Private IP will vary so setting it to static will allow users to login using the domain name without worry_

- Go t DC-1 Virtual Machine Overview Page
- Click on `Networking` then `Network Interface`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/a86cf0ab-fc56-456b-bae5-df14a5ce3d14)

- Click on `IP Configuration`
- Click on `ipconfig` (You can see the Private IP Address is Dynamic)

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/ac108e4f-1f5a-427d-b4c2-4afa2dcefeec)

- Change `Assignment` to **Static**
- Click `Save`

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/20d55691-3648-4a28-94c5-b36139811882)

<h3>&#9315 Ensure Connectivity between the Client and Domain Controller</h3>

>**Note***
>_Refer back to ["Let's Create Resource Groups and Deploy a Virtual Machine Together!"](https://github.com/CarlosAlvarado0718/Virtual-Machine) in order to Remote Desktop Connect into your Virtual Machines._

- Login to the Client's Virtual Machine

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/588362ad-74be-4d32-9f00-825a73d8eb08)

- Login to the Domain Controller's Virtual Machine

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/e4e96c1b-3bc1-4488-aa8d-f312aefc2bc5)

- On the Domain Controller's VM, type into Windows search bar **Windows Defender Firewall with Advanced Security**

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/671eac9e-4f64-41ea-8cac-3deea53b05ff)

- Click `Inbound Rules`
- Sort by `Protocol`
- Find `Core Networking Diagnostics` - `ICMP Echo Request (ICMPv4-In)`
- Select them and click `Enable Rule`

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/754d1de9-2d97-4f14-8bf7-c522358fbfd5)

- Return to DC-1 Overview Page in Azure
- Copy the `Private IP Address` underneath `Networking`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/efc871c0-1d87-40fb-819f-4f3cd4731941)

- Go into Client-1's Virtual Machine
- Type **Command Prompt** into the Windows Search Bar
- Click `Run as Administrator`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/80537f7a-f50a-40e4-9f31-b65c3597f973)

- Type **ping -t (DC-1 Private IP Address)**
   >_This will send data packets for response to DC-1 VM_
- You'll see the connectivity between the Client and DC-1 VM's
   >_Press `Ctrl+C` to stop the ping, or close the application_ 

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/07527a4b-1a4a-42e5-9f30-60e70b55b009)

<h3>&#9316 Install Active Directory Domain Services in Domain Controller Virtual Machine</h3>

- Login into DC-1's Virtual Machine
- Open `Server Manager`
- Click `Add Roles and Features`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/1d391c3b-06b2-435a-87a1-3e08077dec1f)

- Click `Next` until you reach `Select Server Roles` tab
- Checkmark `Active Directory Domain Services`
- Click `Add Features`
- Click `Next` until the Confirmation tab
- Click `Install` then `Close`

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/f2e3078b-7c31-4fa5-bf8f-ce9f3c069241)

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/90dabda9-536e-4e90-bef4-299135200218)

- Located on the Top Right Header, Click the Flag icon with a Caution Symbol
- Click `Promote this server to a domain controller`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/77a6e8a4-5717-40bf-a2e2-c08cc3fa95cb)

- Click `Add a new forest` within the Deployment Configuration Tab
- Type **mydomain.com**
- Click `Next`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/3f5ab5e7-1725-4ad0-8cd3-ccacc656d574)

- Set `Password` as **Password1**
- Click `Next` until `Install` option is enabled
- Click `Install`
    >_This action will restart the Domain Controller Virtual Machine_

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/5d77e390-e84d-4035-a817-d30c49d93b4d)

- Log into DC-1's Virtual Machine using (RDP)
- Select `More Choices`
- Click `Use a different account`
- Set the `Username` as **mydomain.com\labuser
- Set the `Password` as **osticketPassword1**
    >**Note***
    >_Don't forget the password_

![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/1988bfe0-930e-45b6-abec-42123a1fe0dc)

<h3>&#9317 Create an Admin Account in Active Directory</h3>

>**Notes***
>_We're going to make two folders_

- On the Top Right Header, click `Tools`
- Click `Active Directory Users and Computers`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/5fc81f5a-2fc7-48e5-aa04-1cced62c1a79)

- Right Click `mydomain.com`
- Hover `New`
- Click `Organizational Unit`

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/7f2b5f7c-72a9-4594-aeff-c3e3d065e0ed)

- Name one folder **_EMPLOYEES**
- Name another folder **_ADMINS**

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/ae7aa5c1-4cf9-4a5b-8b63-e544d0d5ee66)

- Right Click `_ADMINS`
- Hover `New`
- Click `User`

 ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/5c549afd-743a-485f-8e20-38501618c286)

- Set `First name` to **Jane**
- Set `Last name` to **Doe**
- Set `User logon name` to **jane_admin**
- Set `password` to **Password1**
- Uncheck `User must change password at next login`
- Checkmark `Password never expires`
- Click `Next` until the account is created

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/5b17ccda-76b7-4d40-b608-1a95e15d60e0)

  ![image](https://github.com/CarlosAlvarado0718/Configure-AD/assets/140138198/b209d86a-1cfd-499b-81aa-701c6790d89f)

- Right click on the account
- Click `Properties`

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
