#  Unconstrained Delegation in Active Directory  
### From a Penetration Testing Perspective

---

## Introduction

In modern Active Directory (AD) environments, **Unconstrained Delegation** remains a commonly abused misconfiguration. It provides an easy path for privilege escalation and lateral movement, especially in internal penetration tests where attackers assume the breach scenario.

This blog documents a full scenario from a red team perspective, demonstrating how a compromised machine configured with unconstrained delegation can lead to Domain Admin compromise.

---

## Lab Setup

- **Engagement Type:** Internal Active Directory Penetration Test  
- **Assessment Approach:** Assume Breach (Local Admin on host)  
- **Duration:** 2 Days  
- **Attacker Host:** `JOINEDDOMAIN2`

---

##  Step 1: Discover Unconstrained Delegation Machines

First, we import PowerView and PowerUp modules to enumerate AD objects:

```powershell
powershell -ep bypass
.\powerview.ps1
.\powerup.ps1
Get-DomainComputer -Unconstrained | Select-Object -ExpandProperty name
```

**Output:**

![image](https://github.com/user-attachments/assets/88c45cee-fa40-4e66-b5e9-1dcfaacf3d36)

As seen, `JOINEDDOMAIN2` is among the systems configured with unconstrained delegation.

---

##  Step 2: Confirm NTLM Compatibility (for Kerberos Ticket Handling)

Ensure the target machine allows the usage of NTLMv2, which supports the Kerberos ticket handling used in our attack.

![image](https://github.com/user-attachments/assets/e4324116-333c-4377-a941-84f759b8b1c1)


---

##  Step 3: Start TGT Monitoring with Rubeus

```powershell
.\Rubeus.exe monitor /interval:10 /filteruser:JOINEDDOMAIN2 /nowrap
```

Rubeus will monitor for incoming TGTs every 10 seconds.
![image](https://github.com/user-attachments/assets/db10458c-6780-42e9-a137-e96e6b81b10d)


---

##  Step 4: Admin RDP Access (Victim Logs In)

Enable RDP on the victim machine (JOINEDDOMAIN2):

![image](https://github.com/user-attachments/assets/47dc29a9-3b6f-4f85-88c9-4b198b55d3af)


Domain admin logs in via RDP:

![image](https://github.com/user-attachments/assets/e20c5eff-0255-4988-8d7e-2ce714995a55)


Connection from domain controller using MARVEL\Administrator:

![image](https://github.com/user-attachments/assets/a2bb378e-b1be-4e5b-b53e-b6b35c11374c)


---

##  Step 5: TGT Captured by Rubeus

Once the Domain Admin logs in, Rubeus captures the TGT:

![image](https://github.com/user-attachments/assets/211b3920-79e6-4c6f-bd6a-ab9fd8561f32)


---

##  Step 6: Inject TGT and Assume Identity

Use the captured TGT to inject and impersonate the Domain Admin:

```powershell
.\Rubeus.exe ptt /ticket:BASE64TICKET==
```

Result:

![TGT Injected](sandbox:/mnt/data/287eb3a5-0040-44d1-8a66-7c02bfbbeff1.png)

---

##  Step 7: Explore Filesystem as Domain Admin

List contents of `Administrator` and `Pparker` profiles to demonstrate proof of access:



---

##  Mitigations for Blue Teams

- Audit delegation settings:
  ```powershell
  Get-ADComputer -Filter * -Properties TrustedForDelegation | Where-Object { $_.TrustedForDelegation -eq $true }
  ```
- Replace unconstrained delegation with **constrained** or **resource-based constrained delegation**.
- Restrict high-privilege accounts from logging into non-secure hosts.
- Monitor for tools like Rubeus using EDR solutions.
![image](https://github.com/user-attachments/assets/c0e77148-a898-4ba3-b039-7fb63679b98c)

---

##  Conclusion

This scenario demonstrates how **Unconstrained Delegation**, although an old vulnerability, still leads to **Domain Admin compromise** in 2025 environments. It is a red team's low-effort, high-reward pathway when misconfigurations persist in internal networks.

**Lesson:** One RDP session to the wrong host can compromise your domain.

---

* Toolset: PowerView, PowerUp, Rubeus | Scenario: JOINEDDOMAIN2 Misconfiguration Exploitation*

