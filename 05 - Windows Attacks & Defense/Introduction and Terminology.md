## What is Active Directory?

Active Directory is a directory service for Windows enterprise environments based on the protocols x.500 and LDAP, making it a distributed, hierarchical structure that allows centralized management of an organization's resources, including users, computers, groups, network devices and file shares, group policies, devices and trusts. It provides authentication, accounting and authorization functionalities and allows administrators to manage permissions and access to network resources. It is the most utilized Identity and Access management (IAM) solution.
The most apparent practice to keep AD secure is ensuring that proper patch management is in place.

## Refresher

- Domain - group of objects that share the same AD database, such as users or devices.
- Tree - one or more domains grouped.
- Forest - group of multiple trees.
- Organizational Units (OU) - AD containers containing user groups, Computers and other OUs.
- Trust - Access between resources to gain permission/access to resources in another domain.
- Domain Controller (DC) - The admin of the AD used to set up the entire Directory. They have the topmost priority and the most authority/privileges.
- AD Data Store - Contains database files and processes that store and manage directory information for users, services and applications. It contains _NTDS.DIT_, the most critical file within an AD environment.
- A regular AD user account can be used to enumerate (this can be changed but will likely damage the AD itself):
	- Domain Computers
	- Domain Users
	- Domain Group Information
	- Default Domain Policy
	- Domain Functional Levels
	- Password Policy
	- Group Policy Objects (GPOs)
	- Kerberos Delegation
	- Domain Trusts
	- Access Control Lists (ACLs)
- Lightweight Directory Access Protocol (LDAP) - Protocol that systems in the network environment use to communicate with AD.
- Authentication can be done using:
	- Username/Password - stored or transmitted as password hashes (LM, NTLM, NetNTLMv1/NetNTLMv2).
	- Kerberos tickets - acts as a trusted third party, working with a DC to authenticate clients trying to access services.
	- Authentication over LDAP - allowed via username/password, or users or computer certificates.
- Key Distribution Center (KDC) - A Kerberos service installed on a DC that creates tickets. It contains the authentication server (AS) and the ticket-granting server (TGS)
- Kerberos Tickets - Tokens that serve as proof of identity (created by the KDC)
	- TGT is proof that the client submitted valid user information to the KDC.
	- TGS is created for each service the client with a valid TGT wants to access
- KDC key is an encryption key that proves the TGT is valid.
- The group Enterprise Admins has permissions over all domains in the forest.
- Some important ports:
	- 53 - DNS
	- 88 - Kerberos
	- 135 - WMI/RPC
	- 137-139 & 445 - SMB
	- 389 & 636 - LDAP
	- 3389 - RDP
	- 5985 & 5986 - PowerShell Remoting (WinRM)

## Real-world view

There are a lot of misconfigurations that go against the least privilege principle (for commodity).
AD has limitation however:
1. **Complexity** - It is easy to get lost when looking into who is a member of a group, a member of another group and a member of yet another group. Many environments have every Domain user indirectly a member of Domain Admins.
2. **Design** - SMB stores SYSVOL, that contains GPOs. SMB allows code execution, so as long as we have a set of valid credentials, we can consistently execute code over SMB on the Domain Controllers remotely.
3. **Legacy** - Windows is not secure by default. DNS-like protocols NetBIOS and LLMNR are enabled by default and these broadcast user credentials on the wire. This [blog post](https://www.a2secure.com/en/blog/how-to-use-responder-to-capture-netntlm-and-grab-a-shell/) demonstrates the abuse of capturing credentials.
