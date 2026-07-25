# corp.local :Active Directory Attack Lab

A self-hosted Active Directory environment built from scratch to reproduce, and then attack, the kind of misconfigurations that show up repeatedly in real internal penetration tests: Kerberoastable service accounts, AS-REP roastable users, an over-permissioned help-desk account, and unconstrained delegation sitting on a box nobody audits.

Full build log with every command, screenshot, and troubleshooting step: [`corp.local_Active_Directory_Attack_Lab_-_Ayman_Ibnousoufyane.pdf`](./corp.local_Active_Directory_Attack_Lab_-_Ayman_Ibnousoufyane.pdf)

---

## What this is

Six virtual machines on a single isolated subnet, wired together into a two-domain forest with a real trust relationship, then deliberately misconfigured and attacked from a dedicated Kali box.

| Machine | Role |
|---|---|
| **DC01** | Forest root domain controller, `corp.local` |
| **DC02** | Child domain controller, `child.corp.local` |
| **SRV-SVC** | Kerberoasting target, AS-REP roasting target, unconstrained delegation |
| **WS01 / WS02** | Domain-joined workstations (standard users, no local admin) |
| **Kali** | Attacker platform |

## Attack paths built into the lab

- **Kerberoasting** — `svc-sql` carries a registered SPN and a crackable password
- **AS-REP Roasting** — `svc-legacyapp` has Kerberos preauthentication disabled, requestable with zero credentials
- **ACL abuse** — `jdupont`, an unremarkable standard user, holds `GenericAll` over the `PrivilegedAccess` security group (BloodHound-style attack path)
- **Unconstrained delegation** — enabled on SRV-SVC's computer object
- **AD CS / ESC1** — attempted, not completed; documented honestly in the report as a troubleshooting log rather than a success

## What's actually been proven

The AS-REP roasting attack was executed live end to end from Kali: unauthenticated hash request → offline crack with hashcat → recovered plaintext password. Every other path above is configured and verified in Active Directory, but not yet exploited from the attacker box (see the report's closing chapter for the exact status of each).

## Why it looks the way it does

- **One flat `10.10.10.0/24` subnet**, isolated on its own VMware host-only network (VMnet2), no default gateway on any domain-joined machine
- **Role consolidation over more VMs** — the delegation misconfiguration and the AD CS attempt both live on SRV-SVC instead of dedicated boxes, which mirrors a common real-world pattern: one under-resourced server running several unrelated services
- **French-localized Windows Server** — several scripts had to be fixed to resolve built-in groups (Domain Admins, Enterprise Admins) by well-known SID/RID instead of by display name, since the local names differ from the English defaults

## Repository contents

```
├── corp.local_Active_Directory_Attack_Lab_-_Ayman_Ibnousoufyane.pdf   # full build log
├── scripts/                                                            # PowerShell provisioning scripts
│   ├── 01-DC01-Setup-OUs-Users-GPOs.ps1
│   ├── 02-Add-HeadOfITAdmin.ps1
│   ├── 03-Fix-DomainAdmins-Membership.ps1
│   ├── 04-DC02-Promote-ChildDomain.ps1
│   ├── 05-Add-EnterpriseAdmins.ps1
│   ├── 06-Create-Kerberoast-ASREP-Accounts.ps1
│   ├── 07-Enable-UnconstrainedDelegation-SRVSVC.ps1
│   └── 08a/08b/08c-ADCS-*.ps1                                         # prepared but not fully executed
└── README.md
```

## Tools used

`VMware Workstation Pro` · `Windows Server 2025` · `Windows 10/11` · `Kali Linux` · `Impacket` · `hashcat` · `BloodHound` (planned, not yet run) · `PowerShell` / `Active Directory module`

## Disclaimer

This lab runs entirely on an isolated, host-only virtual network with no internet-facing exposure and no connection to production systems of any kind. It exists purely for personal skill development in an offline home-lab environment. Nothing here targets or references any real organization.

## Author

**Ayman Ibnousoufyane**
[YOUR GITHUB LINK HERE] · [YOUR LINKEDIN LINK HERE]
