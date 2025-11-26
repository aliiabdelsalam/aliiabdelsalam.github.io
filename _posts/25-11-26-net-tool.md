---
title: Enumeration Using Net Tool
date: 2025-11-26
category: Active Directory
img_path: ../assest/images/Active Directory/net tool
---
In this series, we will start talking about Active Director  and the most common Attack  carried out against it. The advantage is that we will explain how to perform the Attack in a practical wa  and also how to Detec  it as an Incident Responde .

Initially, we will begin with the most critical part an Attacker performs: the Enumeration  process.

For those who don't know what Enumeration is, we will quickly explain it: It's an essential Attack Phas  that the attacker must carry out because it helps them gather more information about their targe , such as the Services that might be available and their versions, as well as if there are any open Sessions or sessions that can be opened, like a Null Sessio  (which we will explain later). Overall, it's a very beneficial process for the Attacker.

We have many Tool  that can help us in enumeration, such as `Nmap`, `Powerview`, `Psloogedon`, `BloodHound`, `CrackMapExec`, `SMBMap`, `enum4linux`, and `net.exe`.

These are some of the Tools we use in enumeration. We won't cover them all, but we will explain a part of them.

---

<h1><span style="color:Darkorange; font-weight:bold"> Net Tool</span></h1>

## <span style="color:red; font-weight:bold"> Offensive Part </span>

Let's start with our first Tool: <span style="font-Weight:bold">Net.exe</span> .

The net.exe  utility is a built-in Windows tool used for managing accounts, groups, services, networks, sessions, etc. It is considered a fast way to collect information  Enumeratio specific to the domain without needing high privileges or external tools.
 Most Important Enumeration Commands using net 

### <span style="font-size:20px; font-weight:bold">1) Enumerate Domain User </span> 

```powershell
net user /domain
```

This displays all User  in the Active Directory.

<img src="{{ '/assets/Images/Active Directory/net tool/1.png' | relative_url }}" alt="1">
---

### <span style="font-size:20px; font-weight:bold">2) View Full Info About a Specific User </span> 


```powershell
net user username /domain
```

This displays:

- The groups the member belongs to
- Last password set
- Last logon time
- Whether the account is Active or has Restrictions

Example:

```powershell
net user administrator /domain
```

<img src="{{ '/assets/Images/Active Directory/net tool/2.png' | relative_url }}" alt="2">

---

### <span style="font-size:20px; font-weight:bold">3) Enumerate Domain Group </span> 


```powershell
net group /domain
```

This displays all groups at the domain level.

<img src="{{ '/assets/Images/Active Directory/net tool/3.png' | relative_url }}" alt="3">

---


### <span style="font-size:20px; font-weight:bold">4) List Members of a Group </span> 

```powershell
net group "Domain Admins" /domain
```

This outputs who is inside that group.

Important examples:

```powershell
net group "Enterprise Admins" /domain
net group "Administrators" /domain
```

<img src="{{ '/assets/Images/Active Directory/net tool/4.png' | relative_url }}" alt="4">

---


### <span style="font-size:20px; font-weight:bold">5) Enumerate Local Groups on a Machine </span> 

```powershell
net localgroup
```

<img src="{{ '/assets/Images/Active Directory/net tool/5.png' | relative_url }}" alt="5">

---


### <span style="font-size:20px; font-weight:bold">6) Members of a Local Group</span> 

```powershell
net localgroup administrators
```

This displays who has Admin Acces  on the local machine  very important for Lateral Movement.

<img src="{{ '/assets/Images/Active Directory/net tool/6.png' | relative_url }}" alt="6">

---

### <span style="font-size:20px; font-weight:bold">7) Show Domain Controller</span> 

```powershell
net group "Domain Controllers" /domain
```

The Domain Controller name here is D .

<img src="{{ '/assets/Images/Active Directory/net tool/7.png' | relative_url }}" alt="7">

---


### <span style="font-size:20px; font-weight:bold">8) Find Logged-in Users on a Host</span> 

```powershell
net session
```

If you have privileges:

- You will be able to see who has an open session on the machine
- Very useful for Pivotin 

---

 
### <span style="font-size:20px; font-weight:bold">9) Enumerate Network Share</span> 

```powershell
net share
```

The first step to discover shared folder  + writable share .

<img src="{{ '/assets/Images/Active Directory/net tool/8.png' | relative_url }}" alt="8">

---


### <span style="font-size:20px; font-weight:bold">10) Show Computers in the Domain</span> 

```powershell
net view /domain
```

And if you want to see machines in a specific domain:

```powershell
net view /domain:DOMAINNAME
```

---


### <span style="font-size:20px; font-weight:bold">11) View Shares on a Specific Machine</span> 

```powershell
net view \\TARGET
```

---

Now we've learned how to use the Net too . It is one of the simplest enumeration tools and doesn't produce a large output, but the information we were able to obtain will be very useful for the Attack operations we will carry out on our target.

---

Now, how can we perform Detectio  for anyone using a tool like Ne  to perform Enumeration and gather information about us? Of course, as an Incident Responde , we monitor everything that happens in our environment, and we will use a tool like Wireshar  to perform this Detection.

---

## <span style="color:blue; font-weight:bold">Detection Part</span>

How can we identify that the `net` comman  is running in our environment? Firstly, all `net` commands operate using SMB + RPC over SM .

What appears in Wireshark is:

- Session Setu 
- Tree Connect to `IPC$ 
- RPC Call  such as:
    - `SamrEnumerateDomainsInSamServer`
    - `SamrLookupDomain` 
    - `SamrEnumerateUsersInDomain` 
    - `SamrQueryInformationGroup` 
    - `NetShareEnum` 

This is what we use to understand which Command is running.

<img src="{{ '/assets/Images/Active Directory/net tool/9.png' | relative_url }}" alt="9">

What is shown in the image:

- An SMB connectio  from: `192.168.3.134` → `192.168.3.131`
- Session Setu 
- Tree Connec  → `\\DC.ricin.com\IPC$`
- Then Request  containing RPC Call 

This indicates that an Enumeration proces  is taking place on the Server.

Now, we have a few things in the image that we will assume we don't know what they are, so we will quickly explain each one of them.

---

### What is SMB?

The Server Message Block SMB  protocol is a client-server network protocol that enables applications to access and share resources, such as files, printers, and serial ports, across a network. Originally developed by IBM, SMB has evolved significantly and is integral to networking in Microsoft Windows, as well as on Linux and macOS via implementations like Samba.
 How SMB work 

SMB uses a request-response model where a client sends a request to a server to access a shared resource. The communication process follows these steps:

1. Session establishmen : A client requests to establish an SMB session with the server.
2. Authenticatio : The client sends credentials, and the server authenticates the user before granting access. Modern SMB versions use robust authentication methods like Kerberos or NTLMv2.
3. Resource acces : After successful authentication, the client can perform actions (read, write, create, delete) on the shared files or use shared printers.
4. Session terminatio : When finished, the client sends a logoff message to close the session.

SMB primarily uses TCP port 44  for communication over TCP/IP networks. Older, insecure versions used TCP port 139 via NetBIOS.

---

### What is SMB Session Setup?
 Session Setup is the step where the client asks the server to create a "user session," and it usually includes Authenticatio .

- During this, the client sends authentication data: this could be NTLM  or Kerberos  (in a domain).
- After a successful Session Setup, the server returns a SESSION I , and you can perform a Tree Connec  to a Share File or IPC .
 In Wireshar : After the Negotiate Protocol Request/Response, you will find a `Session Setup Request` and `Response`. If successful, you will see `STATUS_SUCCESS` and often information about the user or a token.

<img src="{{ '/assets/Images/Active Directory/net tool/10.png' | relative_url }}" alt="10">

---

### SMB2/SMB3

These are newer versions of SMB and are better than SMBv1 in terms of security and encryption. You might want to read more about them.

---

### DCERPC
 DCE/RPC (Distributed Computing Environment / Remote Procedure Call)  is a protocol for remote procedure calls that allows a client to execute a procedure on a remote server as if it were a local call, hiding the complexity of the network communication. Developed by the Open Software Foundation, it is a core component of many systems, especially in Microsoft Windows environments where it is known as MS-RPC and is used for services like Active Directory. The protocol uses a Universal Unique Identifier (UUID) to identify services and relies on a client-server model where the client is blocked until the server responds.
 How it work 

- Client-side  A client application calls a remote procedure. The DCE/RPC library on the client machine encodes the call and its arguments into network packets and sends them to the server.
- Server-side  The server's DCE/RPC library decodes the packets and passes the request to the specific procedure. The server then executes the procedure and sends the results back to the client.
- Interface and Operation  A server's capability is defined by a UUID for the interface and an operation number for the specific function. The server uses these identifiers to route the request to the correct procedure.
- Endpoint Mapper  The server's Endpoint Mapper (EPMAP) listens for incoming calls. A client first contacts the EPMAP to find the specific endpoint for a given interface, and then connects to that endpoint for subsequent calls.
 In summar , DCERPC is a Remote Procedure Call (RPC) protocol heavily used in Windows over SM  (especially via `\\host\IPC$`).

SMB itself does not perform Enumerations; it is merely a transport mechanis . However, the entity that actually performs the Enumeration is DCERPC (Remote Procedure Call . This means:
 Calling functions on the server remotely via SMB  The SMB is just a channe  for transmitting the DCERPC messages.

It's like telling the server:

"Get me the list of users."

"Get me the groups."

"Get me the shares."

These commands are executed via DCERP .

And it is executed inside SMB on a Pipe called:

```powershell
\\IPC$
```

<img src="{{ '/assets/Images/Active Directory/net tool/11.png' | relative_url }}" alt="11">

---

### SAMR

The SAMR (Security Account Manager Remote) protoco  is a Microsoft protocol that allows for the remote management of user and group accounts and security policies on Windows systems. It is an RPC-based protocol that operates over SMB and uses port 445. This enables administrators and applications to remotely query, create, modify, and delete security principals like users and groups.
 Key features and function 

- Remote managemen : SAMR allows for the remote management of security account information, enabling actions like creating, reading, updating, and deleting user and group accounts.
- Protocol transpor : It uses the Remote Procedure Call (RPC) protocol for communication and is typically carried over the Server Message Block (SMB) protocol, which uses port 445.
- Data mode : SAMR uses the Active Directory data model for both local and remote scenarios, ensuring a consistent management experience across different account stores.
- Target object : It exposes abstract objects for server, domain, group, alias, and user, which can be managed through the protocol.
- Security implication : While a legitimate protocol, SAMR is also used in security attacks for reconnaissance, as it can be used to enumerate user and group information.
 How it's use 

- Domain controller : Domain controllers use SAMR to synchronize and manage user account information across a network.
- Local account : It is also used for managing local accounts on individual computers through their Security Account Manager (SAM) database.
- Administrator tool : Administrators can use SAMR to manage accounts and policies remotely.
- Fallback mechanis : In some cases, it can be used as a fallback when other protocols like LDAP are not available or fail.
 In brief, SAM  is a DCERPC interface responsible for managing/reading security account information in the SAM / AD (Users, Groups, aliases).

Its functions:

- Retrieves Users
- Retrieves Groups
- Translates SIDs
- Queries User Details

In Wireshark, you will find:

```powershell
SamrEnumerateDomainsInSamServer
SamrLookupDomain
SamrOpenDomain
SamrEnumerateUsersInDomain
SamrQueryInformationUser
SamrEnumerateAliasesInDomain
```

When you see SAM  → you know immediately that User/Group Enumeratio  is occurring.

<img src="{{ '/assets/Images/Active Directory/net tool/12.png' | relative_url }}" alt="12">


As shown in the image, an Enumeration occurred on the Domain, and the hacker was able to obtain Users and Groups, just as we did above.

---

### LSARPC — Local Security Authority RP 

LSARPC is a Microsoft protocol that provides remote management for the Local Security Authority. It uses RPC over SMB, communicating over the `\\PIPE\\lsarpc` named pipe, and is used for managing security-related policy settings on Windows systems. The protocol is also associated with the Local Security Authority (Translation Methods) Remote Protocol, MS-LSAT.

If you enumerate domain policies or translate SIDs, LSARP  is activated.

Its functions:

- Retrieves Security Policies
- Translates SIDs → Names
- Retrieves Trusts
- Retrieves general Security Information

In Wireshark, you will find:

```powershell
LsarOpenPolicy
LsarLookupSids
LsarQueryInformationPolicy
```

<img src="{{ '/assets/Images/Active Directory/net tool/13.png' | relative_url }}" alt="13">

---

### Quick Summary of What Happened

1. We performed Enumeratio  using the `net  tool.
2. We opened Wireshar  to monitor everything that would happen.
3. We found many SMB connection  working with a protocol like DCERP , and we learned that it is responsible for retrieving the information we requested via the `net` tool.
4. We learned that SM  is just a channe  for transmitting the DCERPC connection.
5. We also learned that SAM  is responsible for readin  this information, such as which Users are in the Domain, and so on.
6. And LSARP  is activated when SIDs are translated into Name  or when a Security Polic  is queried.

---
