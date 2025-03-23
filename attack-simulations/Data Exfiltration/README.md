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

**3. Disabling the Windows Firewall (Sysmon)**

After gaining access, I analyzed Sysmon logs to detect further suspicious activity. At 02:56:25 (UTC), I found an entry showing that the attacker executed  super suspissious the netsh command to disable the Windows Firewall across all profiles.

This action suggests that the attacker was attempting to weaken the system's network to facilitate outbound connections.

Here is the Sysmon Event ID 1 (Process Creation) log:

![firewall](https://github.com/user-attachments/assets/8f279c2e-edd7-42e6-ae79-7cc8fc1c233e)



**4. File Creation and Directory Logs (Sysmon)**

- To gain deeper insights into user activities, I turned to Sysmon logs, which provided a detailed process and deeper information, looking for Event ID 1 (Process Creation).

- With that, I found an entry that shows the attacker used PowerShell to create a compressed archive named data.zip from a directory named data. This occurred at 02:57:18 (UTC).

- Here is the Sysmon Event ID 1 (Process Creation) log:

![Datazip](https://github.com/user-attachments/assets/dfe40ac0-a367-46d9-867a-e8619a491b94)


This confirms that the attacker compressed sensitive files into a .zip archive for potential exfiltration.


**5. Suspicious Network Connection (Sysmon)**

- After that, just 1 miute right after, in the Sysmon logs, I found a suspicious network connection logged under Event ID 3 (Network Connection). This entry shows that the Windows machine (192.168.10.20) established a connection to the kali’s IP address (192.168.20.30) over port 4444, which is a non-standard port often used in reverse shells and data exfiltration activities, which may mean that the attacker tried to send the data.zip fille on this Network Connection.

Here we can see the Sysmon Event ID 3 (Network Connection) log:

![netcat](https://github.com/user-attachments/assets/35cc0c0e-55c7-4c0c-8972-4db73f1b77f2)



## **Firewall Rules on pfSense:**

1- **Create an Alias for Internal Systems:**

- ![rule1](https://github.com/user-attachments/assets/a05efab9-6c65-46b9-9ba9-350ee7470d62)

Explanation:
 - This limits RDP access to trusted internal devices (from Network: 192.168.10.0/24)


2- **Block Unauthorized Kerberos Requests:**

- ![rule2](https://github.com/user-attachments/assets/d3a12053-ea1f-483f-ad70-11f24596919b)

Explanation:
- This blocks port 4444 and other non-standard ports that kali used for reverse shells.


3- **Limit External and Internal LDAP Queries:**

- ![rule3](https://github.com/user-attachments/assets/98eeeb87-f991-49f5-8b85-fcb567f4de7b)


Explanation:
-  This prevents exfiltration via FTP/SSH/HTTP/S.


Note: In this experiment, CrowdSec was disabled to allow this action to proceed, but the rules are on the [configs directory](https://github.com/NunooFerreira/SOC-Lab-Home-Cybersecurity-Monitoring/blob/main/configs/CrowdSec_configs/custom_exfiltration_watch.yaml).









