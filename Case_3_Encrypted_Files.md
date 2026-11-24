# **Case 3: Encrypted Files (Inmobiliaria del Cerro)**

Nov 11, 2026 · @Facu

## **Case details**

Company: Inmobiliaria del Cerro (fictional), Mendoza, 6 employees. Windows server SRV-ARCHIVOS with the shared folder D:\Compartido.

What was reported: On Monday 11/23 at 08:30, nobody could open the files: they all ended in **`.cerro`** and a **`LEEME_RECUPERAR.txt`** file had appeared in every folder.

## **What happened?**

Sunday 11/22, between 01:40 and 02:12: an RDP brute-force attack from 203.0.113.61, with 772 failed attempts. At 02:13 the attacker managed to log in with the "recepcion" account via Remote Desktop (LogonType 10).

02:14–02:23: reconnaissance (whoami, administrators, shared folders); download of the ransomware svch0st.exe from the same IP 203.0.113.61; deletion of shadow copies, the backup catalog and Windows recovery; 02:22:58: the ransomware ran on D:\Compartido (extension .cerro).

03:41: the same account cleared the audit log (1102). The evidence was preserved thanks to the centralized log collector.

The attack was possible because RDP was exposed to the internet, with no account lockout and no MFA, the recepcion account was an administrator, and the backups were stored on the same server.

## **How was it resolved?**

Containment: isolate SRV-ARCHIVOS without shutting it down, close the RDP port published on the router, block 203.0.113.61, disable recepcion and change all credentials.

Recovery: the backups were on the same server and were deleted. Check whether any copies remain on drive E: and look for a decryptor on nomoreransom.org. Do not pay the ransom.

Close RDP and use a VPN, enable account lockout and MFA, remove administrator rights from recepcion and apply 3-2-1 backups.

Communication: notify the clients whose contracts were in D:\Compartido and assess, with legal advice, the notification required under Law 25.326 (AAIP, Argentina's data protection authority).

## **Tools used**

* Google Cloud Shell
* `jq` (with `-r`, `-c`, `select` and `@tsv`)
* `sort` and `uniq -c`
* `cat`, `ls`, `cd` and `pwd`

```
cd ~/caso-4

jq -r '.[].EventID' evidencia/eventos_security.json | sort | uniq -c

jq -r '.[] | select(.EventID == 4625) | .IpAddress' evidencia/eventos_security.json | sort | uniq -c | sort -rn

jq -c '.[] | select(.EventID == 4624 and .IpAddress == "203.0.113.61")' evidencia/eventos_security.json

jq -r '.[] | select(.EventID == 4688 and .SubjectUserName == "recepcion") | [.TimeCreated, .CommandLine] | @tsv' evidencia/eventos_security.json

jq -c '.[] | select(.EventID == 1102)' evidencia/eventos_security.json
```
