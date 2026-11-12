# **Case No. 1**

Nov 11, 2026 · @Facu

## **Case details**

Case number: 1

Company: Estudio Contable del Valle (fictional), Mendoza. It has a Linux server (srv-archivos) holding its clients' documentation in /srv/clientes, reachable over SSH from the internet. 

What was reported: Marcelo (IT) found a user on the server that nobody created and less free disk space than on Friday. 

**What happened?**

Confirmed compromise. An attacker got in by brute-forcing SSH, stole the clients' documentation, and left a backdoor to get back in.

* 03:02–03:19 (11/10): SSH brute force from 203.0.113.88, with 628 failed attempts out of 668 total for the week.  
* 03:19:14: successful login with the "soporte" account (an external vendor account with sudo privileges) by guessing its password.  
* 03:23–03:31: using sudo, reads /etc/shadow (hashes of all passwords), creates the user "sysbackup" with administrator privileges, installs their own SSH key on it (persistence), and compresses /srv/clientes into /tmp/.c.tgz.  
* 03:31–03:41: sends /tmp/.c.tgz via scp to 198.51.100.40 and deletes the file to cover their tracks. Client data exfiltration confirmed.  
* 04:12: logs back in with the "sysbackup" account and its SSH key from 198.51.100.40. The backdoor is already in use.

Root cause: SSH exposed to the internet with password login (PasswordAuthentication yes) and direct root login (PermitRootLogin yes); an external vendor account with sudo and a weak password; no attempt lockout or alerts.

## **How was it resolved?**

Investigation (in Google Cloud Shell):

1. Counted the failed attempts with `grep -c "Failed password"`: 668\.  
2. Grouped the attempts by IP with `grep -oE "from [0-9.]+" | sort | uniq -c | sort -rn`: 628 came from 203.0.113.88.  
3. Searched for successful logins from that IP with `grep "Accepted"`: logged in as "soporte" at 03:19.  
4. Reviewed passwd.txt and grupo\_sudo.txt: "soporte" and the new "sysbackup" have sudo.  
5. Searched the sudo commands with `grep "COMMAND"`: user creation, SSH key, and compression of /srv/clientes.  
6. Read the bash\_history of "soporte": data sent via scp to 198.51.100.40.  
7. Searched for that IP in the log: came back in with "sysbackup" at 04:12.

Containment:

* Isolate the server.  
* Block 203.0.113.88 and 198.51.100.40.  
* Lock the "sysbackup" and "soporte" accounts without deleting them (evidence) and kill their sessions.  
* Remove the attacker's key from authorized\_keys, saving a copy first.

Eradication:

* Change all passwords on the server, since the attacker has the hashes from /etc/shadow.  
* Look for other backdoors: keys in other authorized\_keys files (including root), crontab, and new users.

Prevention:

* SSH with keys only (PasswordAuthentication no) and no root (PermitRootLogin no).  
* Remove sudo from the vendor account and grant access only when needed.  
* Install fail2ban and restrict SSH to a VPN or known IPs.  
* Alerts on a high number of failed attempts.

Communication: notify the firm's clients about the leak of their documentation and assess the obligations under Argentina's Personal Data Protection Law 25.326.

## **Tools used**

`grep`, `sort`, `uniq`, `cat`, `head`. 
