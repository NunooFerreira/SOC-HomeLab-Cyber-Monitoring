# Active Directory Attack Playbook (Using RDP and Data Exfiltration)

## **Overview**

- **Target:** Windows Workstation (within the soc.lab domain)
  - IP Address: 192.168.10.20 
  - Domain: `soclab.local`

- **Attacker: Kali Linux**
  - IP Address: 192.168.20.30

From the last brute force attack:
- The attacker has successfully discovered user credentials.
- The attacker gains remote access through RDP (Remote Desktop Protocol).

We can see in the screenshot below that the Kali Linux machine was successfully logged in and is now inside the victim's system.

![kali_hacked_windows10](https://github.com/user-attachments/assets/1d1e42ed-2628-4dcc-9d2f-7f88a6604d40)



## **Monitoring Windows Logs**

**1. Credential Validation Attempt:**
- We can see that the attacker with the name: kali attempted to validate credentials for the labadmin user, which was logged as a successful authentication in the Windows Event Viewer, with Error Code: 0x0 (Success):

![Succefull_validated](https://github.com/user-attachments/assets/4adf116e-0ef2-429e-938c-637fb25e51be)


**2. Kerberos Ticket Granting Ticket (TGT) Request**

- Following successful authentication, a Kerberos TGT request was logged, which means the attacker now has a TGT issued for labadmin, allowing further access to domain resources.

![kerberos](https://github.com/user-attachments/assets/c4501f99-27a2-4924-9537-f8f9c520d466)


**3. File Creation and Directory Logs (Sysmon)**

- To gain deeper insights into user activities, I turned to Sysmon logs, which provided a detailed process and deeper information, looking for Event ID 1 (Process Creation).

- With that, I found an entry that shows the attacker used PowerShell to create a compressed archive named data.zip from a directory named data. This occurred at 02:57:18 (UTC).

- Here is the Sysmon Event ID 1 (Process Creation) log:

![Datazip](https://github.com/user-attachments/assets/dfe40ac0-a367-46d9-867a-e8619a491b94)


This confirms that the attacker compressed sensitive files into a .zip archive for potential exfiltration.

