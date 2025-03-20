# Active Directory Attack Playbook (Kerberos-based SPN Extraction and Credential Compromise)

## **Overview**

- **Target:** Active Directory Domain Controller (within the soc.lab domain)
   - IP Address: 192.168.10.10
   - Domain: SOC.LAB
   - 
- **Attacker:** Kali 
   - IP Address: 192.168.20.30
  
1. Objective
The objective of this attack was to enumerate and extract Service Principal Names (SPNs) from the Active Directory environment and attempt to crack the Kerberos TGS hashes to gain unauthorized access to domain accounts.
Following the other attacks, we are already know the credentials from an User and the Active Directory Domain.


