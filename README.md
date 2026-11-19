# AD + Entra ID Hybrid Identity Lab

A self-built hybrid identity environment: an on-premises Active Directory domain controller synced to Microsoft Entra ID via Microsoft Entra Connect. Built as a hands-on IAM portfolio project to demonstrate real provisioning, access control, and offboarding workflows.

## Goal

Stand up a working hybrid identity environment end to end and prove, with real test data, that the core IAM lifecycle (joiner → mover → leaver) actually functions and syncs correctly between on-prem Active Directory and the cloud.

## Architecture

```
DC01 (Windows Server, AD DS, Domain Controller for lab.local)
        |
        |  Microsoft Entra Connect (Password Hash Sync)
        v
Microsoft Entra ID tenant (jaylenkillian26outlook.onmicrosoft.com)
```

- **DC01** — Windows Server domain controller, running inside VirtualBox, hosting the `lab.local` Active Directory forest.
- **Microsoft Entra Connect** — installed on DC01, syncing selected OUs up to Entra ID using Password Hash Synchronization.
- **Microsoft Entra ID** — free-tier cloud tenant created via an Azure free account, serving as the cloud identity directory.

## What I Built

1. Created `DC01`, a Windows Server 2025 VM in VirtualBox, and promoted it to a domain controller for a new forest, `lab.local`.<img width="1912" height="1031" alt="Screenshot 2026-11-18 213649" src="https://github.com/user-attachments/assets/42b3a97f-6a9e-48c5-bd6f-ae19d6012b72" />

2. Built an OU structure (`Lab > Employees`, `Lab > Groups`) and provisioned 10 test user accounts via a PowerShell script (`New-ADUser`) rather than the GUI, one at a time.<img width="752" height="367" alt="Screenshot 2026-11-18 213732" src="https://github.com/user-attachments/assets/ca2c29e7-3d25-4f86-9b0e-74647415a9fd" />

3. Created two security groups (`Sales`, `IT-Support`) and assigned test users to each, based on role.<img width="757" height="330" alt="Screenshot 2026-11-18 213752" src="https://github.com/user-attachments/assets/a8178926-6b6c-485c-8205-7b66550c5595" />

4. Verified account functionality by temporarily granting "Allow log on locally" to Domain Users on the DC and signing in as a test user to confirm authentication worked end to end.<img width="1072" height="782" alt="Screenshot 2026-11-18 214049" src="https://github.com/user-attachments/assets/172a8bb0-9a87-4115-b556-a572bf90d419" /> <img width="1037" height="806" alt="Screenshot 2026-11-18 214138" src="https://github.com/user-attachments/assets/62ea4c3e-8359-4e2e-8e48-8ebd2b78fa47" /> <img width="1018" height="225" alt="Screenshot 2026-11-18 214201" src="https://github.com/user-attachments/assets/5262c31c-4f30-42de-bcee-1bda12ae27da" />



5. Created a free Microsoft Entra ID tenant and installed Microsoft Entra Connect on DC01, scoping the sync to the `Employees` and `Groups` OUs using Password Hash Synchronization.<img width="890" height="634" alt="Screenshot 2026-11-18 215952" src="https://github.com/user-attachments/assets/5dc5c701-1537-4076-910e-147ae80a868f" />

6. Confirmed successful synchronization via Synchronization Service Manager, and validated that on-prem users and groups appeared correctly in the Entra admin center.<img width="807" height="650" alt="Screenshot 2026-11-18 214608" src="https://github.com/user-attachments/assets/037d1429-30c0-45ab-a575-a7c0b3481933" />

7. Ran and documented the core IAM lifecycle against the live sync:
   - **Joiner** — created a new on-prem user and confirmed it synced to Entra ID.
   - **Mover** — moved a user between security groups on-prem and confirmed the group membership change synced to Entra ID.
   - **Leaver** — disabled (not deleted) a departing user on-prem and confirmed the account showed as disabled in Entra ID.
8. Deliberately broke sync (stopped the `ADSync` service), made a change on-prem, confirmed it did *not* appear in Entra ID, then restarted the service and forced a delta sync (`Start-ADSyncSyncCycle -PolicyType Delta`) to confirm recovery.<img width="890" height="634" alt="Screenshot 2026-11-18 215952" src="https://github.com/user-attachments/assets/4b59ecbc-ac1c-47cd-a585-1b6e4269c75f" />
<img width="1025" height="646" alt="Screenshot 2026-11-18 214920" src="https://github.com/user-attachments/assets/acb03e45-d1c8-44d2-9962-cf66c3c1688d" />
<img width="1023" height="680" alt="Screenshot 2026-11-18 214703" src="https://github.com/user-attachments/assets/c9da9fee-63a0-4652-a4de-e78c343ed735" />
<img width="1010" height="565" alt="Screenshot 2026-11-18 214635" src="https://github.com/user-attachments/assets/f2ead0df-58d7-4195-912a-f9c582542186" />
<img width="707" height="446" alt="Screenshot 2026-11-18 214901" src="https://github.com/user-attachments/assets/7858fda9-c640-472b-ad2a-09301fc93036" />



## Challenges and Troubleshooting

Real problems I ran into and resolved, documented because this is the part interviewers actually care about:

- **VM wouldn't boot ("No physical memory is available at the location required for the Windows Boot Manager")** — traced to the VM's Base Memory being misconfigured at 4 MB instead of 4096 MB (4 GB). Corrected the allocation and the VM booted normally.
- **No real IP address (APIPA / 169.254.x.x)** — occurred while working on a restrictive network (company guest Wi-Fi with client isolation, plus general unreliability of Bridged networking over Wi-Fi in VirtualBox). Resolved by moving the networking-dependent steps to a home network and using an Internal Network adapter for VM-to-VM traffic plus a second NAT adapter dedicated to outbound internet access.
**NOTE: Entra ID sign-in error (AADSTS50020)

    Got "account does not exist in tenant 'Microsoft Services'" twice, in
    two different places, with two different fixes:

    1. During Azure signup - personal MS account got routed to a dead-end
       placeholder tenant instead of creating a new one.
       Fix: signed up with a brand-new Microsoft account instead.

    2. During the Entra Connect installer - same error, even though that
       account was already confirmed Global Admin of the tenant. Personal
       accounts aren't reliably recognized by every Microsoft sign-in flow.
       Fix: created a dedicated admin account inside the tenant
       (syncadmin@<tenant>.onmicrosoft.com) and used that instead.

    Takeaway: same error, different cause each time - the fix was "stop
    using the personal account for admin," not retrying the same login.
