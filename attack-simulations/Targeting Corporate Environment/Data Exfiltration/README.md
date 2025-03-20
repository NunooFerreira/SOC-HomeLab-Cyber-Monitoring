# Active Directory Attack Playbook (Using RDP and Data Exfiltration)

## **Overview**

- **Target:** Active Directory Server: Windows Server 2022 Datacenter Evaluation
  - IP Address: `192.168.10.10`
  - Domain: `soclab.local`
  - Vulnerable Users: Populated by **BadBlood**
 





- **Target:** Windows Workstation (within the soc.lab domain)
  - IP Address: 192.168.10.20 
  - Domain: `soclab.local`

- **Attacker: Kali Linux 
  - IP Address: 192.168.20.30

From the last brute force attack:
- The attacker has successfully discovered user credentials.
- The attacker gains remote access through RDP (Remote Desktop Protocol).

We can see in the screenshot below that the Kali Linux machine was successfully logged in and is now inside the victim's system, with Error Code: 0x0 (Success)

![kali_hacked_windows10](https://github.com/user-attachments/assets/1d1e42ed-2628-4dcc-9d2f-7f88a6604d40)



## **Monitoring Windows Logs**

## **1. Credential Validation Attempt
First, we can see that the attacker  with the name: kali attempted to validate credentials for the labadmin user, which was logged as a successful authentication in the Windows Event Viewer:

![Succefull_validated](https://github.com/user-attachments/assets/4adf116e-0ef2-429e-938c-637fb25e51be)



Print Example: Add Event ID 4624 showing successful login here
