# DCSync

DCSync is an attack that allows an adversary to simulate the behavior of a domain controller (DC) and retrieve password data via domain replication. The classic use for DCSync is as a precursor to a Golden Ticket attack, as it can be used to retrieve the KRBTGT hash.

Specifically, DCSync is a command in the open-source Mimikatz tool. It uses commands in the Directory Replication Service Remote Protocol (MS-DRSR) to simulate the behavior of a domain controller and ask other domain controllers to replicate information —  taking advantage of valid and necessary functions of Active Directory that cannot be turned off or disabled.

Generally, Administrators, Domain Admins and Enterprise Admins have the rights required to execute a DCSync attack. Specifically, the following rights are required:

![](<../../.gitbook/assets/image (90).png>)

```
impacket-secretsdump EGOTISTICAL-BANK.LOCAL/svc_loanmgr@10.10.10.175
```
