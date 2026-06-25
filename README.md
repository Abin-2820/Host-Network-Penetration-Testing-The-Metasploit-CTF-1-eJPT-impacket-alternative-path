# Host-Network-Penetration-Testing-metasploit-ctf1-eJPT-impacket-alternative-path
Alternative Initial Access to Windows MSSQL Lab Using Impacket

## Alternative Initial Access in eJPT Metasploit Framework CTF 1 Using Impacket

## Author
Abin Watson

---

## Disclaimer

This writeup is created strictly for educational purposes.  
All activities were performed in an authorized INE eJPT lab environment.  
No real-world systems were targeted.

---

## Overview

During the **Host & Network Penetration Testing: The Metasploit Framework CTF 1 (eJPT)** lab, most public walkthroughs relied on the Metasploit module:

exploit/windows/mssql/mssql_clr_payload

However, instead of using an exploit-based MSSQL CLR payload approach, this assessment demonstrates a credential-driven methodology using Impacket.

This alternative path better reflects real-world penetration testing scenarios where valid credentials are often obtained before exploitation.

---

## Lab Context

Platform: INE eJPT  
Challenge: Metasploit Framework CTF 1  
Target: Windows system running MSSQL  
Objective: Gain initial access and escalate privileges  

---

## Alternative Methodology (Credential-Based Access)

### Phase 1 – Service Enumeration

MSSQL service was identified during network scanning.

---

### Phase 2 – Credential Discovery

Brute-force attempts against MSSQL resulted in valid credentials:

User: sa

This shifted the attack strategy from exploitation to authenticated access.

---

### Phase 3 – Authenticated Remote Interaction

Instead of launching a Metasploit exploit module, the tool used was:

impacket-mssqlclient sa@<IP address>

This provided authenticated access to the MSSQL service.

Key reasoning:

- Leverage valid credentials
- Maintain controlled execution
- Reduce exploit dependency

---

### Phase 4 – Shell Staging

After authenticated interaction, a payload was staged and executed to obtain a reverse Meterpreter session via multi/handler.

This provided interactive access to the target system.

---

### Phase 5 – Privilege Escalation

Once inside the Meterpreter session:

getsystem

This successfully elevated privileges to SYSTEM.

---

## Comparative Analysis

| Factor | MSSQL CLR Exploit | Impacket Credential Method |
|--------|------------------|----------------------------|
| Requires Exploit | Yes | No |
| Requires Credentials | No | Yes |
| Stability | Moderate | Moderate |
| Real-World Likelihood | Medium | High |
| Detection Surface | SQL Exploit Logs | Authenticated SQL Login |

---

## Key Lessons Learned

- Enumeration results should guide methodology decisions.
- Tools not explicitly taught in coursework (like Impacket) can provide powerful alternatives.
- Always evaluate multiple paths to initial access.

---

## Conclusion

This case study highlights the importance of methodology over automation.
Rather than relying solely on predefined exploit modules, leveraging discovered credentials through tools like Impacket can provide:
- Real-world applicability
- Improved understanding of attack surface

##Author 

Abin Watson,
Date : February 2026 
