# Administrator

![OS](https://img.shields.io/badge/OS-Windows-0078D6?style=flat-square)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Retired-lightgrey?style=flat-square)
![Pwned](https://img.shields.io/badge/User%20%2B%20Root-Pwned-3fb950?style=flat-square)

| | |
|---|---|
| **Platform** | HackTheBox |
| **OS** | Windows Server (Domain Controller) |
| **Difficulty** | Medium |
| **Creator** | nirza |
| **Released** | 9 November 2024 |
| **Retired** | 19 April 2025 |
| **Domain** | `administrator.htb` |

## About the box

Administrator is an assumed-breach Active Directory machine. You start with credentials for a single low-privileged domain user and no obvious way forward: no shell, no interesting shares. The way in is the domain's own permissions. Every account on the path has a Windows ACL right over the next one, so the box comes down to reading the BloodHound graph one hop at a time until you reach a user who can DCSync the whole domain.

No CVEs and no exploits. Every step uses a misconfigured permission or a credential someone left behind, which is why it's a good model of how real AD environments get compromised.

## Attack path

```
olivia ──GenericAll──▶ michael ──control edge──▶ benjamin ──FTP──▶ Backup.psafe3
                          │                                            │
                       WinRM                                   crack master pw
                                                                       │
                                     ethan ◀──GenericWrite──── emily ◀─┘
                                       │     (targeted Kerberoast)  │
                                    DCSync                    WinRM (user flag)
                                       │
Administrator ◀──Pass-the-Hash── NT hash
```

1. **olivia → michael:** GenericAll over michael lets olivia force-reset his password with `bloodyAD`.
2. **michael → WinRM:** michael is in *Remote Management Users*, so he gets a shell, but no flag.
3. **michael → benjamin:** an outbound control edge leads to benjamin, who has FTP access.
4. **benjamin → Backup.psafe3:** a Password Safe v3 database pulled off FTP in binary mode.
5. **Crack the safe:** `pwsafe2john` + `hashcat -m 5200` opens it. It has entries for alexander, emily and emma, and only emily's works.
6. **emily → user flag:** emily is in *Remote Management Users*, and `user.txt` is on her desktop.
7. **emily → ethan:** GenericWrite over ethan allows a targeted Kerberoast, and his TGS cracks with `hashcat -m 13100`.
8. **ethan → root:** ethan has replication rights, so `secretsdump` dumps the Administrator NT hash and pass-the-hash gives the root flag.

## Techniques

`Active Directory` · `BloodHound` · `ACL abuse` · `GenericAll` · `ForceChangePassword` · `WinRM` · `FTP` · `Password Safe cracking` · `GenericWrite` · `Targeted Kerberoast` · `DCSync` · `Pass-the-Hash`

**Tools:** NetExec, BloodHound CE, bloodyAD, evil-winrm, pwsafe2john, hashcat, targetedKerberoast, impacket-secretsdump

## Writeup

📄 **[Read the full writeup](https://h3ndr1x1.github.io/HTB-writeups/Administrator/Administrator.html)** (rendered page) · [HTML source](./Administrator.html)

> Credentials and hashes in the writeup are partially redacted. Flags are not included.
