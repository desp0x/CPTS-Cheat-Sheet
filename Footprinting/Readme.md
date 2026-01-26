# Footprinting Cheat Sheet

## Infrastructure-based Enumeration

| Command | Description |
|------|------------|
| ```curl -s https://crt.sh/?q=<target-domain>&output=json \| jq .``` | Certificate transparency. |
| `curl -s https://crt.sh/?q=inlanefreight.com&output=json \| jq . \| grep name \| cut -d":" -f2 \| grep -v "CN=" \| cut -d'"' -f2 \| awk '{gsub(/\\n/,"\n");}1;' \| sort -u` | Unique Subdomain |
| `for i in $(cat subdomainlist);do host $i \| grep "has address" \| grep inlanefreight.com \| cut -d" " -f4 >> ip-addresses.txt;done` | Host to IP. |
| `for i in $(cat ip-addresses.txt);do shodan host $i;done` | Scan each IP address in a list using Shodan. |

---

## Host-based Enumeration

### FTP

| Command | Description |
|------|------------|
| `ftp <FQDN/IP>` | Interact with the FTP service on the target. |
| `nc -nv <FQDN/IP> 21` | Interact with the FTP service on the target. |
| `telnet <FQDN/IP> 21` | Interact with the FTP service on the target. |
| `openssl s_client -connect <FQDN/IP>:21 -starttls ftp` | Interact with the FTP service on the target using encrypted connection. |
| `wget -m --no-passive ftp://anonymous:anonymous@<target>` | Download all available files on the target FTP server. |

---

## SMB

| Command | Description |
|------|------------|
| `rpcclient -U "" <FQDN/IP>` | Interaction with the target using RPC. |
| `srvinfo`	| Server information.|
| `enumdomains`	| Enumerate all domains that are deployed in the network.|
| `querydominfo` |	Provides domain, server, and user information of deployed domains.|
| `netshareenumall` |	Enumerates all available shares.|
| `netsharegetinfo <share>` |	Provides information about a specific share.|
| `enumdomusers` | Enumerates all domain users.|
| `queryuser <RID>`	| Provides information about a specific user.|
| `for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" \| grep "User Name\|user_rid\|group_rid" && echo "";done` | Brute Forcing User RIDs |
| `smbclient -N -L //<FQDN/IP>` | Null session authentication on SMB. |
| `smbclient //<FQDN/IP>/<share>` | Connect to a specific SMB share. |
| `samrdump.py <FQDN/IP>` | Username enumeration using Impacket scripts. |
| `smbmap -H <FQDN/IP>` | Enumerating SMB shares. |
| `crackmapexec smb <FQDN/IP> --shares -u '' -p ''` | Enumerating SMB shares using null session authentication. |
| `enum4linux-ng.py <FQDN/IP> -A` | SMB enumeration using enum4linux. |

---

## NFS

| Command | Description |
|------|------------|
| `showmount -e <FQDN/IP>` | Show available NFS shares. |
| `mount -t nfs <FQDN/IP>:/<share> ./target-NFS/ -o nolock` | Mount the specific NFS share to `./target-NFS`. |
| `umount ./target-NFS` | Unmount the specific NFS share. |

---

## DNS

| Command | Description |
|------|------------|
| `dig ns <domain.tld> @<nameserver>` | NS request to the specific nameserver. |
| `dig any <domain.tld> @<nameserver>` | ANY request to the specific nameserver. |
| `dig axfr <domain.tld> @<nameserver>` | AXFR request to the specific nameserver. |
| `dnsenum --dnsserver <nameserver> --enum -p 0 -s 0 -o found_subdomains.txt -f ~/subdomains.list <domain.tld>` | Subdomain brute forcing. |
| `dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb`| Command|

---

## SMTP

| Command | Description |
|------|------------|
| `telnet <FQDN/IP> 25` | Connect to SMTP service. |
| `smtp-user-enum -M VRFY -U users.list -t 10.129.203.12 -D inlanefreight.htb` | SMTP User-Enum |

---

## IMAP / POP3

| Command | Description |
|------|------------|
| `curl -k 'imaps://<FQDN/IP>' --user <user>:<password>` | Log in to the IMAPS service using cURL. |
| `openssl s_client -connect <FQDN/IP>:imaps` | Connect to the IMAPS service. |
| `openssl s_client -connect <FQDN/IP>:pop3s` | Connect to the POP3s service. |

IMAP Commands

| Command | Description |
|------|------------|
| `1 LOGIN username password`	| User's login.|
| `1 LIST "" *	`| Lists all directories.|
| `1 CREATE "INBOX"` | Creates a mailbox with a specified name.|
|` 1 DELETE "INBOX"`| Deletes a mailbox.|
| `1 RENAME "ToRead" "Important"`	| Renames a mailbox.|
| `1 LSUB "" *`	| Returns a subset of names from the set of names that the User has declared as being active or subscribed.|
| `1 SELECT INBOX`	| Selects a mailbox so that messages in the mailbox can be accessed.|
| `1 UNSELECT INBOX`	| Exits the selected mailbox.|
| `1 FETCH <ID> all`	| Retrieves data associated with a message in the mailbox.|
| `1 CLOSE`	| Removes all messages with the Deleted flag set.|
| `1 LOGOUT`	| Closes the connection with the IMAP server.|

POP3 Commands

|Command |	Description |
|------|------------|
| `USER username`	| Identifies the user.|
| `PASS password`	| Authentication of the user using its password.|
| `STAT`	| Requests the number of saved emails from the server.|
| `LIST`	| Requests from the server the number and size of all emails.|
| `RETR id`	| Requests the server to deliver the requested email by ID.|
| `DELE id`	| Requests the server to delete the requested email by ID.|
| `CAPA`	| Requests the server to display the server capabilities.|
| `RSET	`| Requests the server to reset the transmitted information.|
| `QUIT`	| Closes the connection with the POP3 server.|

---

## SNMP

| Command | Description |
|------|------------|
| `snmpwalk -v2c -c <community string> <FQDN/IP>` | Querying OIDs using snmpwalk. |
| `onesixtyone -c community-strings.list <FQDN/IP>` | Bruteforcing community strings of the SNMP service. |
| `braa <community string>@<FQDN/IP>:.1.*` | Bruteforcing SNMP service OIDs. |

---

## MySQL

| Command | Description |
|------|------------|
| `mysql -u <user> -p<password> -h <FQDN/IP>` | Login to the MySQL server. |

---

## MSSQL

| Command | Description |
|------|------------|
| `mssqlclient.py <user>@<FQDN/IP> -windows-auth` | Log in to the MSSQL server using Windows authentication. |

---

## IPMI

| Command | Description |
|------|------------|
| `msf6 auxiliary(scanner/ipmi/ipmi_version)` | IPMI version detection. |
| `msf6 auxiliary(scanner/ipmi/ipmi_dumphashes)` | Dump IPMI hashes. |

---

## Linux Remote Management

| Command | Description |
|------|------------|
| `ssh-audit.py <FQDN/IP>` | Remote security audit against the target SSH service. |
| `ssh <user>@<FQDN/IP>` | Log in to the SSH server using the SSH client. |
| `ssh -i private.key <user>@<FQDN/IP>` | Log in to the SSH server using private key. |
| `ssh <user>@<FQDN/IP> -o PreferredAuthentications=password` | Enforce password-based authentication. |

---

## Windows Remote Management

| Command | Description |
|------|------------|
| `rdp-sec-check.pl <FQDN/IP>` | Check the security settings of the RDP service. |
| `xfreerdp /u:<user> /p:"<password>" /v:<FQDN/IP>` | Log in to the RDP server from Linux. |
| `evil-winrm -i <FQDN/IP> -u <user> -p <password>` | Log in to the WinRM server. |
| `wmiexec.py <user>:"<password>"@<FQDN/IP> "<system command>"` | Execute command using the WMI service. |

---

## Oracle TNS

| Command | Description |
|------|------------|
| `./odat.py all -s <FQDN/IP>` | Perform a variety of scans to gather information about the Oracle database services and its components. |
| `sqlplus <user>/<pass>@<FQDN/IP>/<db>` | Log in to the Oracle database. |
| `./odat.py utlfile -s <FQDN/IP> -d <db> -U <user> -P <pass> --sysdba --putFile C:\\insert\\path file.txt ./file.txt` | Upload a file with Oracle RDBMS. |
