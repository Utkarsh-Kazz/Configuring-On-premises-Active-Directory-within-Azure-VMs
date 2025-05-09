<p>
<img src="https://i.imgur.com/pU5A58S.png" height="40%" width="70%"alt="Microsoft Active Directory Logo"/>
</p>

<h1>Configuring Active Directory (On-Premises) Within Azure</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />
<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10

<h2>High-Level Deployment and Configuration Steps</h2>

- Create Resources
- Ensure Connectivity between the client and Domain Controller
- Install Active Directory
- Create an Admin and Normal User Account in AD
- Join Client-1 to your domain (myadproject.com)
- Setup Remote Desktop for non-administrative users on Client-1
- Create additional users and attempt to log into client-1 with one of the users

<h2>Deployment and Configuration Steps</h2>
<br />
<br />
<h3 align="center">Setup Resources in Azure</h3>
<br />
<p>
  In the Azure portal, I created a Virtual Network (VNet) and subnet to house both my virtual machines. Then I deployed a Windows Server 2022 VM named "DC-1" to act as my Domain Controller, and assigned it a static private IP address.
</p>
<p>
  <img src="https://i.imgur.com/gaAzjvb.png" height="75%" width="100%" alt="resource group"/>
  <img src="https://i.imgur.com/hubTfey.png" height="75%" width="100%" alt="vm ms server"/>
</p>
<p>
   I also deployed a Windows 10 VM named "Client-1" in the same VNet, region, and resource group as the domain controller to ensure proper communication between the two machines.
</p>
<p>
  <img src="https://i.imgur.com/XyEmv8f.png" height="75%" width="100%" alt="vm windows"/>
</p>
<p>
  Set Domain Controller’s NIC Private IP address to be static:
</p>
<p>
  <img src="https://i.imgur.com/KHU9kC4.png" height="75%" width="100%" alt="static ip"/>
</p>
<p>
  <img src="https://i.imgur.com/rFpHLdQ.png" height="75%" width="100%" alt="topology"/>
</p>
<br />
<br />
<h3 align="center">Ensure Connectivity between the client and Domain Controller</h3>
<br />
<p>
  To verify network connectivity, I logged into Client-1 using Remote Desktop and continuously pinged DC-1’s private IP using the ping -t command. 
</p>
<p>
  <img src="https://i.imgur.com/bnPM9tX.png" height="75%" width="100%" alt="perpetual ping"/>
</p>
<p>
 I then logged into DC-1 and enabled ICMPv4 (ping) through the local Windows Firewall.
</p>
<p>
  <img src="https://i.imgur.com/ZpPyEkt.png" height="75%" width="100%" alt="enable ICMPv4"/>
</p>
<p>
 I confirmed back on Client-1 that the ping was successful, indicating proper connectivity.
</p>
<p>
  <img src="https://i.imgur.com/8o3OfjY.png" height="75%" width="100%" alt="ping success"/>
</p>
<br />
<br />
<h3 align="center">Installing Active Directory</h3>
<br />
<p>
Next, I installed Active Directory Domain Services on DC-1 and promoted it to a Domain Controller by creating a new forest called mydomain.com (or myadproject.com).
</p>
<p>
  <img src="https://i.imgur.com/A1V9XJ5.png" height="75%" width="100%" alt="active directory install"/>
</p>
<p>
  Promoting it as a Domain Controller
</p>
<p>
  <img src="https://i.imgur.com/zi15fw4.png" height="75%" width="100%" alt="domain controller promotion"/>
</p>
<p>
  <img src="https://i.imgur.com/DCFUVrM.png" height="75%" width="100%" alt="set new forest"/>
</p>
<p>
 After the system rebooted, I logged into DC-1 using the domain account mydomain.com\labuser.
</p>
<p>
  <img src="https://i.imgur.com/7UakWMQ.png" height="75%" width="100%" alt="fqdn login"/>
</p>
<br />
<br />
<h3 align="center">Create an Admin and Normal User Account in AD</h3>
<br />
<p>
 Inside Active Directory Users and Computers (ADUC), I created two Organizational Units (OUs): _EMPLOYEES and _ADMINS.
</p>
<p>
  <img src="https://i.imgur.com/cYmv0r7.png" height="75%" width="100%" alt="organizational unit"/>
  <img src="https://i.imgur.com/v02CBPI.png" height="75%" width="100%" alt="organizational unit"/>
</p>
<p>
  I then created a new user named Jane Doe with the username jane_admin and added her to the “Domain Admins” security group. Afterward, I logged out and logged back into DC-1 using the new admin credentials mydomain.com\jane_admin
</p>
<p>
  <img src="https://i.imgur.com/h546E6L.png" height="75%" width="100%" alt="admin creation"/>
</p>
<p>
  <img src="https://i.imgur.com/mnLwTgq.png" height="75%" width="100%" alt="security group"/>
</p>
<p>  
 Afterward, I logged out and logged back into DC-1 using the new admin credentials mydomain.com\jane_admin
</p>
<p>
  <img src="https://i.imgur.com/xWZ4Kol.png" height="75%" width="100%" alt="admin login"/>
</p>
<br />
<br />
<h3 align="center">Join Client-1 to your domain (myadproject.com)</h3>
<br />
<p>
From the Azure portal, I changed Client-1's DNS settings to point to DC-1’s private IP address and restarted the VM.</p>
<p>
  <img src="https://i.imgur.com/1KRsjI6.png" height="75%" width="100%" alt="client dns settings"/>
</p>
<p>
  I then logged into Client-1 as the local admin and joined the computer to the domain mydomain.com.
</p>
<p>
  <img src="https://i.imgur.com/50wszcP.png" height="75%" width="100%" alt="domain joining"/>
</p>
<p>
  After the machine restarted, I logged into DC-1 and confirmed in ADUC that Client-1 appeared under the “Computers” container, then moved it into the _CLIENTS OU after creating it.
</p>
<p>
  <img src="https://i.imgur.com/vB1n9m0.png" height="75%" width="100%" alt="active directory client verification"/>
</p>
<br />
<br />
<h3 align="center">Enable Remote Desktop Access for Domain Users</h3>
<br />
<p>
  While logged into Client-1 as jane_admin, I opened the system settings and enabled Remote Desktop. I then granted the “Domain Users” group permission to access the machine remotely, allowing standard users to log in via RDP. This could also be automated using Group Policy for larger environments.
<p>
  <img src="https://i.imgur.com/8BfpT3s.png" height="75%" width="100%" alt="remote desktop setup"/>
</p>
<br />
<br />
<h3 align="center">Bulk Create Users and Configure Lockout Policy</h3>
<br />
<p>
 Back on DC-1, I opened PowerShell ISE as an administrator and ran a script to automatically generate 10,000 user accounts inside the _EMPLOYEES OU.I tested account lockout by logging in with incorrect credentials multiple times, causing the account to be locked. I then reset the password and unlocked the account through ADUC. Finally, I modified the domain’s Group Policy settings to lock user accounts after 5 failed login attempts, with a 30-minute lockout duration. 
</p>
<p>
  <img src="https://i.imgur.com/0i8uApf.png" height="75%" width="100%" alt="create users script"/>
</p>
<p>
  <img src="https://i.imgur.com/6QOGzs6.png" height="75%" width="100%" alt="observe create users script"/>
</p>
<p>
After creating the users, I selected one account and attempted to log into Client-1 with it. The login was successful, confirming that the account was active and the domain environment was functioning as intended.
</p>
<p>
  <img src="https://i.imgur.com/ZZCfiCp.png" height="75%" width="100%" alt="employee user accounts"/>
  <img src="https://i.imgur.com/7gBpNzN.png" height="75%" width="100%" alt="employee user selection"/>
  <img src="https://i.imgur.com/cqsddjn.png" height="75%" width="100%" alt="employee user login"/>
</p>
<br />
<br />
<p>
This tutorial demonstrated how to deploy and configure a fully functional Active Directory environment using Azure Virtual Machines, simulating both cloud-based and traditional on-prem infrastructure. You built a secure, manageable environment complete with domain services, DNS settings, user provisioning, and group policy configuration.

✅ Important: Be sure to clean up your Azure environment after finishing this lab. Close all Remote Desktop sessions, delete any unused Resource Groups, and confirm that all resources have been removed to prevent unnecessary charges.
</p>

