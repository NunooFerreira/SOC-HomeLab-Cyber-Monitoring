## Simulated Attack: RDP Brute-Force  

### Target  
- **Windows 10 Workstation**: `192.168.10.20` (RDP port `3389`).  

### Attack Command (Kali Linux)  
```bash
hydra -L /usr/share/wordlists/common_users.txt -P /usr/share/wordlists/rockyou.txt rdp://192.168.10.20

.\cscli.exe alerts list --ip 192.168.10.100
+----+----------------+-----------+---------+----------------------+--------+------------+
| ID |      SCOPE     |  REASON   | ACTION  |        IP            | COUNTRY | AS_NAME    |
+----+----------------+-----------+---------+----------------------+--------+------------+
| 42 | Ip:1.2.3.4     | rdp-bf    | ban     | 192.168.10.30       | US      | Kali Linux |
+----+----------------+-----------+---------+----------------------+--------+------------+
```


![Captura de ecrã 2025-03-16 041244](https://github.com/user-attachments/assets/34501442-2c73-4697-a19d-5d397f631ab2)
