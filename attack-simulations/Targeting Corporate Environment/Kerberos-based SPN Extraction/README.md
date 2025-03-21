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
   - *Commentary:* The successful connection confirmed that the Kerberos service was accessible, setting the stage for further exploitation.

2. **DNS Configuration Update**
   - Updated the default DNS IP to point directly to the Domain Controller.
   - *Commentary:* This step ensured that subsequent requests would be resolved accurately within the target environment.

3. **SPN Extraction**
   - Ran the `GetUserSPNs.py` script from the mpacket toolkit to request SPNs and attempt Kerberos TGS hash extraction:
     ```bash
     python3 GetUserSPNs.py SOC.LAB/labadmin:Root1234! -request
     ```
   - *Commentary:* The output of this command, which originally showed extracted hash data, confirms that the SPNs were successfully retrieved from the Active Directory environment.

4. **Hash Cracking**
   - Cracked the extracted Kerberos TGS hash using hashcat with the rockyou.txt wordlist, and saved the results to `cracked.txt`:
     ```bash
     hashcat -m 13100 spn_hash.txt /usr/share/wordlists/rockyou.txt --force -o cracked.txt
     ```
   - *Commentary:* The cracking process revealed valid credentials from the compromised hash, illustrating the success of the Kerberoasting attack.

## Monitoring Windows Logs

### Sysmon Logs Analysis

- **Observation:**
  - Multiple Sysmon log entries with **Event ID 3** were observed.
  - All connections originated from IP `192.168.10.10`, targeting `192.168.20.30`.
  - Different ephemeral ports were utilized, which is indicative of automated scanning or communication.
- *Commentary:* These Sysmon logs suggest that the attacker was probing the network extensively, using the Domain Controller as an intermediary for further actions.

### Windows Security Logs

- **Event ID 4769 – Kerberos Service Ticket Request**
  - Numerous requests for Kerberos Service Tickets were recorded from the same source IP.
- *Commentary:* This pattern of repeated ticket requests for various user accounts is a classic sign of a **Kerberoasting attack**, where the attacker aims to collect TGS hashes to eventually crack them.

### Detailed User Log Analysis

- **User Activity:**
  - Analysis of individual logs showed multiple ticket requests for user accounts such as *Jordan* and *Candace*.
  - *Commentary:* The targeting of multiple users reinforces the likelihood of a broad attack against the domain, aimed at extracting as many credentials as possible.

- **Logoff Event:**
  - Shortly after the last Kerberos Service Ticket Request, a **Logoff** event (Event ID 4634) was recorded for user `labadmin`.
  - *Commentary:* This logoff indicates that the compromised credentials were used to complete the attack process, confirming that the initial breach provided valid access to perform these actions.

## Summary

This playbook details the step-by-step execution of a Kerberos-based attack on an Active Directory environment. By extracting SPNs and cracking Kerberos TGS hashes, the attacker was able to gain unauthorized access using compromised credentials. The monitoring of Sysmon and Windows Security logs provided clear evidence of the attack's progression, highlighting both the scanning activity and the subsequent Kerberoasting technique. This comprehensive approach not only demonstrates the technical process but also emphasizes the importance of robust logging and monitoring in detecting and mitigating such attacks.
