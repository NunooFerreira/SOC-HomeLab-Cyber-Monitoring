# Active Directory Attack Playbook (Kerberos-based SPN Extraction and Credential Compromise)

## Overview

- **Target:** Active Directory Domain Controller (within the soc.lab domain)
  - **IP Address:** 192.168.10.10
  - **Domain:** SOC.LAB

- **Attacker:** Kali
  - **IP Address:** 192.168.20.30

## Objective

The objective of this attack was to enumerate and extract Service Principal Names (SPNs) from the Active Directory environment, then crack the Kerberos TGS hashes to gain unauthorized access to domain accounts. Prior attacks had already revealed user credentials and access to the Active Directory domain, facilitating this attack.

## Attack Execution Timeline

1. **Kerberos Port Check**
   - Verified that the Kerberos service (port 88) is open on the Domain Controller:
     ```bash
     nc -vz 192.168.10.10 88
     ```
   - ![hash](https://github.com/user-attachments/assets/25122eab-7eb9-4de7-a62d-2ceeda7e50c9)

2. **DNS Configuration Update**
   - Updated the default DNS IP to point directly to the Domain Controller.
   - ![hash](https://github.com/user-attachments/assets/25122eab-7eb9-4de7-a62d-2ceeda7e50c9)

3. **SPN Extraction**
   - Ran the `GetUserSPNs.py` script from the mpacket toolkit to request SPNs and attempt Kerberos TGS hash extraction:
     ```bash
     python3 GetUserSPNs.py SOC.LAB/labadmin:Root1234! -request
     ```
   - ![hash](https://github.com/user-attachments/assets/25122eab-7eb9-4de7-a62d-2ceeda7e50c9)

4. **Hash Cracking**
   - Cracked the extracted Kerberos TGS hash using hashcat with the rockyou.txt wordlist, saving the results to `cracked.txt`:
     ```bash
     hashcat -m 13100 spn_hash.txt /usr/share/wordlists/rockyou.txt --force -o cracked.txt
     ```
   - ![hash](https://github.com/user-attachments/assets/25122eab-7eb9-4de7-a62d-2ceeda7e50c9)

## Monitoring Windows Logs

### Sysmon Logs Analysis

- **Observation:**
  - Multiple Sysmon log entries with **Event ID 3** were observed.
  - All connections originated from IP `192.168.10.10`, targeting `192.168.20.30`.
  - Different ephemeral ports were utilized, indicating automated scanning or communication.
- ![multipleSysmonlogs](https://github.com/user-attachments/assets/01ab753c-f8f0-42d1-8882-62f2a652937a)

### Windows Security Logs

- **Event ID 4769 – Kerberos Service Ticket Request**
  - Numerous requests for Kerberos Service Tickets were recorded from the same source IP.
- ![kerberos_tickets](https://github.com/user-attachments/assets/9275278e-067c-466c-a2bd-3fc551e18f81)

### Detailed User Log Analysis

- **User Activity:**
  - Analysis of individual logs showed multiple ticket requests for user accounts such as *Jordan* and *Candace*.
  - **User Jordan:**  
    ![Jordan](https://github.com/user-attachments/assets/4aab463d-1130-48ea-affc-d23800dd389f)
    
  - **User Candace:**  
    ![candace_rojas](https://github.com/user-attachments/assets/0fa652ef-089b-4bde-97b4-280412b502bf)

- **Logoff Event:**
  - Shortly after the last Kerberos Service Ticket Request, a **Logoff** event (Event ID 4634) was recorded for user `labadmin`.
  - ![logoff](https://github.com/user-attachments/assets/11d039c9-4803-432e-ae34-a427b6db934a)


## **Firewall Rules on pfSense to Mitigate this:**
1- Create an Alias for Internal Systems:

 - ![aliases](https://github.com/user-attachments/assets/103ebae0-d0c5-4883-b196-d6356225c08c)


2- Block Unauthorized Kerberos Requests:
 - ![firstrule](https://github.com/user-attachments/assets/116965fc-3f84-4b72-b76b-414d10214746)


- This rule blocks any Kerberos traffic (port 88) from devices that are not domain controllers, which prevents unauthorized users from interacting with the Kerberos service and reduces the risk of Kerberoasting.

3- Limit External and Internal LDAP Queries:
- ![rule3](https://github.com/user-attachments/assets/d31cc115-efc6-4a93-b269-133962b478db)


This prevents attackers from using LDAP enumeration tools (like GetUserSPNs.py), a well knowed script, to extract Service Principal Names. LDAP is only needed for trusted systems—blocking it for regular users reduces exposure, like we've just seen.

Note: In this experiment, CrowdSec was disabled to allow this action to proceed.

