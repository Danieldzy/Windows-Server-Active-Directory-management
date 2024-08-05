<h1>Windows Server Active Directory Management</h1>

<h2>Considerations</h2>
In a real office environment, best practice dictates that a Domain Controller (DC) should have only one internal-facing NIC for security reasons. Additionally, DCs and workstations should be assigned to different subnets to ensure network isolation. Internet access for the company should be facilitated through a proxy server and a firewall/router, maintaining the DC’s secure internal role.

In a home lab environment, however, we have assigned two NICs to the DC (one internal and one external) to demonstrate the lab. This setup should never been done for real-world use but serves to illustrate how to connect a workstation to the DC using Active Directory and allow the workstation to access the internet. Thus, the lab setup differs from the real-world configuration for educational purposes. Diagram 1 below should be the real-world scenario. Diagram 2 later is the demonstration of our home lab.

<p align="center">
Diagram 1: <br/>
<br/>
<img src="https://imgur.com/Ph2loLY.png" height="80%" width="80%" alt=""/>


<h2>Overview</h2>
The project involves two virtual machines: Windows Server 2022 (DC) with two NICs (NIC one: 10.0.2.15 DHCP to Internet. NIC two: 172.16.0.1 Static assigned) and Windows 11 (Internal network access to the Internet through DC RAS NAT configured).  Later we create new users (_USERS OU) in Active Directory. Connect Windows 11 Workstation to Windows Server 2022 Domain Controller(DC) and access to the Internet.<br/>
<br />


<h2>Utilities Used</h2>

- <b>Active Directory</b> 
- <b>PowerShell</b>

<h2>Environments Used </h2>

- <b>Windows Server 2022 (DC)</b> 
- <b>Windows 11 (Workstation)</b> 

<h2>Program walk-through:</h2>


<p align="center">
Diagram 2: <br/>
<br/>
<br/><img src="https://imgur.com/ZSEAZFf.png" height="80%" width="80%" alt=""/>

<br/>First, in VirtualBox VM settings, select network, and ensure Win Server DC VM Internal NIC(adapter) and Win11 workstation VM adapter are in the same Internal network and have the same name(intnet in this case).<br/>
<img src="https://imgur.com/XHH8omh.png" height="80%" width="80%" alt="Add Credential"/>


<br/>The services we have on our Windows Server 2022 DC server manager: <br/>1, AD DC used to create users and Domain controller.<br/> <br/> 2, DHCP with router function enabled to auto assign IP address to Win11 workstation.<br/>
<br/>3, Remote Access Services (RAS) with NAT (Network Address Translation) enabled. Facilitates the connection of multiple devices on a local network to the internet using a single public IP address. With this feature installed, our Windows 11 Workstation can communicate with the Internet through our Windows Server 2022 DC.<br/>
<img src="https://imgur.com/B8q0o09.png" height="80%" width="80%" alt="Add Credential"/>

<br />PowerShell script for create users walk-through:<br />
<img src="https://imgur.com/azpPcer.png" height="80%" width="80%" alt="Add New Host"/>
<br />$PASSWORD_FOR_USERS = "Password1"<br />
This line sets the variable $PASSWORD_FOR_USERS to the string "Password1". This means that the password "Password1" is now stored in this variable and can be used later in your script.<br />

<br />$USER_FIRST_LAST_LIST = Get-Content .\names.txt<br />
Get-Content .\names.txt: Reads the contents of the names.txt file.
$USER_FIRST_LAST_LIST: Stores the contents of the file in an array variable. <br/>

<br />$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force<br />
ConvertTo-SecureString: This cmdlet converts a plain text string into a secure string, which is encrypted in memory. This is useful for handling sensitive information like passwords securely.<br/>
-AsPlainText: This parameter specifies that the input string should be treated as plain text. Without this, PowerShell would expect the input to be already encrypted. -Force means intentionally converting plain text to a secure string, overriding any warnings. And finally, this input string has been stored as a new variable $password<br/>

<br />New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false<br />
New-ADOrganizationalUnit: This cmdlet is used to create a new Organizational Unit in Active Directory.-Name _USERS: Specifies the name of the new Organizational Unit. In this case, the OU will be named _USERS.
<br />-ProtectedFromAccidentalDeletion $false: This parameter means the OU is not protected, so it can be deleted by mistake or intentionally if needed. This is a lab environment, therefore we set it to $false for demonstration purposes. If you want the OU to be protected from accidental deletion, you would set this parameter to $true:

<br /> Now let us delve into the second part of the scripts, the for loop<br />
<img src="https://imgur.com/azpPcer.png" height="80%" width="80%" alt="Add New Host"/>

<br />foreach ($n in $USER_FIRST_LAST_LIST) {
    <br />$first = $n.Split(" ")[0].ToLower()
    <br />$last = $n.Split(" ")[1].ToLower()<br />
<br />This PowerShell code is starting to split each name in $USER_FIRST_LAST_LIST into first and last names, converting the first name to lowercase. It uses a space " " to separate first and last name.

<br />$username = "$($first.Substring(0,1)$last)".Tolower()
<br />This line of PowerShell code is used to create a username by combining the first initial of the first name with the full last name, and then converting the entire username to lowercase. If the name is John Doe, then it will become 'jdoe' and be stored in the username variable<br />
<br />Write-Host "Creating user: $username" -BackgroundColor Black -ForegroundColor Cyan<br />
Each time the user has been created when running the script, a message Creating user: xxx will be printed out.<br />


<br />Last part of the script<br />
<img src="https://imgur.com/LwoCrqn.png" height="80%" width="80%" alt="Add New Host"/>
<br />This part creates a new user and uses the variable we defined earlier to assign to the required field. <br />
<br />-PasswordNeverExpires $true: Specifies that the password for the user will never expire. In real work environment, you want to set it to false to implement more strict password policy<br />
<br />-Path "ou=_USERS,$(([ADSI]"").distinguishedName)": Sets the path in AD where the user will be created. This concatenates the organizational unit _USERS` with the distinguished name of the domain.<br />
<br />-Enabled $true: Enables the user account upon creation.<br />

<br />Next we run the script, you need to run this PowerShell inside the same directory where the name.txt is stored since it will pull all the information from name.txt. Or you will have to define another $file_path variable to target the name.txt file. <br />
<img src="https://imgur.com/JczTrHu.png" height="80%" width="80%" alt="Add New Host"/>
<br />When the scripts runs, you can see it has printed out the "Creating user:xxx" message, and going to Active Directory User and Computers GUI, you will see the new OU _USERS has been created and all the new users has been created.<br />
<img src="https://imgur.com/CLO6Ejd.png" height="80%" width="80%" alt="Add New Host"/>


<br />Now, we will connect our Windows 11 machine to our domain, establishing a simulated enterprise network management environment. Note that the Windows 11 Workstation VM cannot be Home Edition, as it does not support joining an Active Directory Domain.<br />
<br />After booting up the Win11 Workstation, go to Network Connections and set the IPv4 to DHCP, since this workstation is in the same internal network as our Windows Server DC (We set it up at the beginning), it will automatically get the first IP address from our Windows Server DC DHCP scope which is 172.16.0.100. From there, you can ping our DC and www.google.com, it will get the response.<br />
<img src="https://imgur.com/aPoK1ek.png" height="80%" width="80%" alt="Add New Host"/>
