<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- [Setup Resources in Azure](#setup)
- [Ensure Connectivity Between Client and Domain Controller](#connect)
- [Install Active Directory (AD)](#installad)
- [Create and Admin and Normal User Account in AD](#user)
- [Join Client-1 to Domain](#join)
- [Setup Remote Desktop for non-administrative users on Client-1](#rdc)


<h2>Deployment and Configuration Steps</h2>

<a name = "setup">
<h4>Setup Resources in Azure</h4>
</a>

- Create the Domain Controller Virtual Machine (VM). This VM will use the image of Windows Server 2022. Name your VM DC-1, and ensure that is uses at least 2 virtual CPUs.

  <img width="50%" height = "50%" alt="Create DC-1 VM" src="https://github.com/s-evelyn/configure-ad/assets/53543374/6907be9b-4258-40f6-a3aa-ea3f7927d450">


- Make sure to take note of the VM's resource group once the VM has been deployed as it will be needed when creating your second VM.
    
    <img width="50%" height = "50%" alt="Take note of Resource Group and vnet" src="https://github.com/s-evelyn/configure-ad/assets/53543374/322873bf-1cac-4b21-8f8c-6989f7eae841">

- DC-1 NIC should be set to static, so that it can be reliably discovered accross the server.
  - Navigate to the the DC-1 Virtual Machine and click on Networking, then click on Networking Interface:
 
      <img width="50%" height ="50%"  alt="Change NIC 1" src="https://github.com/s-evelyn/configure-ad/assets/53543374/ce9b8e62-805c-434d-b1f4-464b4f3b89dd">
      
  - Select IP configurations in the menu bar 

     <img width="50%" height ="50%" alt="Change NIC select IP config" src="https://github.com/s-evelyn/configure-ad/assets/53543374/0d980c73-128b-47a6-be60-90150a23be30">
   

  - Notice that the ipconfig is set to Dynamic, go ahead and click on ipconfig

      <img width="50%" height ="50%" alt="change nic 3" src="https://github.com/s-evelyn/configure-ad/assets/53543374/a9637780-ef16-4071-82db-01268d43f78d">

  - Click on Static and then Save the change made. Take note of the private IP address of DC-1

      <img width="50%" height ="50%" alt="change nic to static" src="https://github.com/s-evelyn/configure-ad/assets/53543374/4018653d-0e6e-436e-9e67-e8a252bf11ae">

- Create another VM in azure and call it Client-1.
  - Make sure that it has the Windows 10 22H2 imaging and uses at least 2vcpus. Ensure that is uses the same resource group, and vnet as DC-1

      <img width="50%" height ="50%" alt="Same resource group" src="https://github.com/s-evelyn/configure-ad/assets/53543374/49fad3f6-725e-47a7-b072-2b4202efebf9">

      <img width="50%" height ="50%" alt="V-net the same" src="https://github.com/s-evelyn/configure-ad/assets/53543374/97146f1a-9c2b-4dd4-b496-bda37b8f5854">

----------------------------------------------------------------------------------------------------------------------------
<a name = "connect">
<h4>Ensure Connectivity between Client-1 and Domain Controller</h4>
</a>

- Copy the Public IP of Client-1 from Azure

    <img width="50%" height ="50%" alt="log on to client-1" src="https://github.com/s-evelyn/configure-ad/assets/53543374/5779eee6-3944-4060-815c-02dd770f5cbe">

- Use this to log on to Client-1 via Remote Desktop Connection.

    <img width="50%" height ="50%" alt="log in to remote desktop" src="https://github.com/s-evelyn/configure-ad/assets/53543374/4a772679-fd8b-468a-8c81-ae17aa7f40e5">

- Navigate to command line and ping the private IP address of DC-1. Note that the response times out on each attempt.

    <img width="50%" height ="50%" alt="ping private ip of DC-1" src="https://github.com/s-evelyn/configure-ad/assets/53543374/31a842c7-60b6-407d-8be2-d4118ea30b2a">

- To ensure connectivity, log in to DC-1 via remote desktop, and navigate to the Windows Defender Firewall with Advanced Security.

    <img width="50%" height ="50%" alt="Windows defender firewall" src="https://github.com/s-evelyn/configure-ad/assets/53543374/f774b12d-1947-466c-8296-12e2eed9e205">

- Click on Inbound Rules and find the two ICMP Echos Requests, and click on Enable rule

    <img width="50%" height ="50%" alt="Enable ICMP echos" src="https://github.com/s-evelyn/configure-ad/assets/53543374/3cb7aa1c-6db1-4573-891c-f9f2e433c10c">

- Navigate back to client-1 and perform another ping the private IP address of DC-1. Note that ping succeeded.

    <img width="50%" height ="50%" alt="ping after icmp activation" src="https://github.com/s-evelyn/configure-ad/assets/53543374/1db79797-ad72-42fb-b747-83d424602637">

----------------------------------------------------------------------------------------------------------------------------

<a name = "installad" >
<h4>Install Active Directory</h4>
</a>

- Navigate to DC-1 and open Windows Servers Manager.
- Click on Add roles and features.

    <img width="50%" height ="50%" alt="Install DC Add roles" src="https://github.com/s-evelyn/configure-ad/assets/53543374/6ced0d1e-0f54-40ba-b076-3b64de6784b1">

- Click Next until you get to Server Roles. Click on Active Directory Domain Server.
  
    <img width="50%" height ="50%" alt="Click Active Directory Domain" src="https://github.com/s-evelyn/configure-ad/assets/53543374/f7781151-e389-44a9-a39e-221ae82d75ba">

- Click on Add Features.

    <img width="50%" height ="50%" alt="Add features" src="https://github.com/s-evelyn/configure-ad/assets/53543374/931fbe7a-a0df-4307-9f77-0e4d41054750">

-  Click next until the process is complete. Upon completion you will notice a yellow triangle, click on it and then Click "Promote this server to a domain controller".

    <img width="50%" height ="50%" alt="Promote to domain controller" src="https://github.com/s-evelyn/configure-ad/assets/53543374/23998a3d-c66e-4c2d-b6ea-563789d1d75f">

- Add a new forest. You can name it whatever you would like just remember. For the purporse of this tutorial we will use mydomain.com. Then click next.

    <img width="50%" height ="50%" alt="add a new forest" src="https://github.com/s-evelyn/configure-ad/assets/53543374/c5e7cdb0-adbe-4c51-bf9a-1f63befcf77a">

- Type in the Directory Services restore mode password. 

    <img width="50%" height ="50%" alt="direcroty services restore password" src="https://github.com/s-evelyn/configure-ad/assets/53543374/6633611e-8be4-4a09-9735-a73c24ba4bb2">

- Continue to click next until you get to install, and click install. Once you are finished the VM will restart. Go to azure and refresh the DC-1

    <img width="50%" height ="50%" alt="refresh DC-1" src="https://github.com/s-evelyn/configure-ad/assets/53543374/ecb1d196-30ea-4d97-b177-9451589c99d8">

- Login into DC-1 this time as mydomain\labuser.

    <img width="50%" height ="50%" alt="log in as domain user" src="https://github.com/s-evelyn/configure-ad/assets/53543374/e36a301b-0597-45de-b14c-07cba86aad83">


----------------------------------------------------------------------------------------------------------------------------
<a name = "user">
<h4>**Create an Admin and Normal User Account in DC-1**</h4>
</a>

- In DC-1, open the Windows Server Manager, click on tools and then click on Active Directory Users and Computers (ADUC)

    <img width="50%" height ="50%" alt="Select ADUC" src="https://github.com/s-evelyn/configure-ad/assets/53543374/873573f0-d991-485a-bc61-4b5970fda105">

- Create an Organizational Unit (OU) by right clicking on mydomain.com -> New -> Organizational Unit

    <img width="50%" height ="50%" alt="organizational unit creation" src="https://github.com/s-evelyn/configure-ad/assets/53543374/6d84f959-d45e-4325-b8bd-a340a7aaf779">

- Create a new OU called Employees, then click ok.

    <img width="50%" height ="50%" alt="Add Employees" src="https://github.com/s-evelyn/configure-ad/assets/53543374/e5cb780d-875f-44ab-bd35-229f62412854">

- Create a new OU called ADMINS.
- You are now going to create a new user who has administrative priveleges.
- In ADUC right click on the ADMINS OU that was just created, select New -> Select Users.

    <img width="50%" height ="50%" alt="New user" src="https://github.com/s-evelyn/configure-ad/assets/53543374/13fc3e07-532b-4f44-8778-5fb188cc03b3">

- Fill in with the user's information, then click next.

    <img width="50%" height ="50%" alt="jane doe create 1" src="https://github.com/s-evelyn/configure-ad/assets/53543374/8ad9d8b7-1ddb-41a1-8b41-4942ff29f038">

- Fill out the password section and for the purpose of this tutorial deselect the "User must change password at next login" and click next.

    <img width="50%" height ="50%" alt="jane doe create 2" src="https://github.com/s-evelyn/configure-ad/assets/53543374/46c498df-21d4-416c-86d3-a7bb23a9543a">


- Once the user has been created, right click on the user and select properties.

    <img width="50%" height ="50%" alt="Add Jane Doe to Security Group" src="https://github.com/s-evelyn/configure-ad/assets/53543374/5dc45b82-125e-4e36-8d41-450a58f22e56">

- Click on Member Of and Add.
  
    <img width="50%" height ="50%" alt="Add Jane Doe to Security Group 2" src="https://github.com/s-evelyn/configure-ad/assets/53543374/4123316a-9cc7-4bdb-a6e3-2aa356111abc">

- Type in domain admin and click ok.

    <img width="50%" height ="50%" alt="add jane doe to security group 3" src="https://github.com/s-evelyn/configure-ad/assets/53543374/9be20ef9-9344-4a97-8b3d-736ab6af8b9b">

- Logout/close the remote desktop to DC-1 and log backin as your admin user, i.e. domain/jane.doe.

    <img width="50%" height ="50%" alt="Sign in as jane doe" src="https://github.com/s-evelyn/configure-ad/assets/53543374/043c9666-22bb-41ce-9752-3efe9b3a0cdc">
 
------------------------------------------------------------------------------------------------------------------------

<a name = "join">
<h4>Join Client-1 to the Domain</h4>
</a>


- Go to the Azure Portal and navigate to the Client-1 VM.
- Click on Networking and then on the network interface

    <img width="50%" height ="50%" alt="client 1 dns change" src="https://github.com/s-evelyn/configure-ad/assets/53543374/89d5cdb1-0753-4f64-8979-e68c2e1f8ba0">

- Click on DNS servers tab

    <img width="50%" height ="50%" alt="change dns" src="https://github.com/s-evelyn/configure-ad/assets/53543374/9a22d652-56b9-45a4-a32f-587c01cddbcf">

- Select Custom, and then type in DC-1's private IP address.
  
    <img width="50%" height ="50%" alt="change dns 2" src="https://github.com/s-evelyn/configure-ad/assets/53543374/abf2ade2-9e8a-4220-b95d-16abcc5f7b1b">


- Restart Client-1 from the Azure Portal.
  
    <img width="50%" height ="50%" alt="Restart Client 1" src="https://github.com/s-evelyn/configure-ad/assets/53543374/6bf03add-e39d-4191-a50a-f1c4b70caab3">

- Login in to the Client-1 via remote desktop.
- Navigate to System Propertise and then About.
- Click Rename PC (Advanced).

    <img width="50%" height ="50%" alt="Rename PC attach to my domain" src="https://github.com/s-evelyn/configure-ad/assets/53543374/4b68f694-8a56-4f92-9ebb-e83e3149eab0">

-  Click Change.

    <img width="50%" height ="50%" alt="rename pc 2" src="https://github.com/s-evelyn/configure-ad/assets/53543374/d30c2933-f017-43d0-b525-41d3829017ae">

- Click Domain, and then type in the name of your domain then click ok.
  
    <img width="50%" height ="50%" alt="rename pc 3" src="https://github.com/s-evelyn/configure-ad/assets/53543374/23b17835-7d15-4aca-b7da-bbe8c46e8c9d">

- Type in the login information for the admin user you created.

    <img width="50%" height ="50%" alt="login as jane doe admin" src="https://github.com/s-evelyn/configure-ad/assets/53543374/3670a8ba-1124-4b94-872e-90af945d8df8">

- The computer is now going to let you know that you need to restart for the change to take place. And then prompt you to restart.

    <img width="50%" height ="50%" alt="restart computer" src="https://github.com/s-evelyn/configure-ad/assets/53543374/30cd7d36-ae0d-44de-be02-2331b73fc2ce">

    <img width="50%" height ="50%" alt="restart computer 2" src="https://github.com/s-evelyn/configure-ad/assets/53543374/332adafa-c99d-4314-8490-e45a157d0038">

- Open the DC-1 and then go to ADUC. Click on mydomain.com and then Computers. If you see Client-1 there this verifies that it has been added to the domain.

    <img width="50%" height ="50%" alt="See Client-1 is attached" src="https://github.com/s-evelyn/configure-ad/assets/53543374/bc8e2c38-ceaa-4daf-a493-86c90fc4e108">


--------------------------------------------------------------------------------------------------------------------------

<a name = "rdc">
<h4>Setup Remote Desktop for non-administrative users on Client-1</h4>
</a>

- Login to Client-1 as the administrative user you created.
- Open System Properities and click on Remote Desktop.

    <img width="50%" height ="50%" alt="users who can connect to mydomain" src="https://github.com/s-evelyn/configure-ad/assets/53543374/283c4067-23e3-4acb-9c20-923f183e0df1">

- Click Add.

   <img width="50%" height ="50%" alt="add remote desktop users" src="https://github.com/s-evelyn/configure-ad/assets/53543374/6c9135b6-8864-46e3-8aad-c0788e338820">
    
- Type in domain users, and then click ok.
  
   <img width="50%" height ="50%" alt="add domain users" src="https://github.com/s-evelyn/configure-ad/assets/53543374/851254a4-83ca-4f0a-a399-c86c94189b23">


- Note that this is normally done with Group Policy, which would allow you to do this to many systems at once.

- Open PowerShell ISE and run as an administrator:
- Copy the following script into powershell and run the code

- This code will create multiple users that are managed by this domain controller. Take note of the password that is being assigned to the users. Password1

  <img width="50%" height ="50%" alt="Add Powershell ISE" src="https://github.com/s-evelyn/configure-ad/assets/53543374/abac47aa-fd2b-4d16-b0b3-b69c36509ff2">

- Open ADUC, and navigate to the users and obvserve the users that have been generated by this code.

  <img width="50%" height ="50%" alt="observe users created" src="https://github.com/s-evelyn/configure-ad/assets/53543374/7e585154-3688-4ea7-b17f-8c42ec951260">

- To affirm that any domain user can access Client-1, choose one of the created domain users and log into Client-1

  <img width="338" alt="User login" src="https://github.com/s-evelyn/configure-ad/assets/53543374/7239b6d8-7c59-459a-8513-cbb2a386231a">

____________________________________________________________________________________________________________________________________________

**Congratulations you have successfully configured active directory on a azure virtual machine!**

