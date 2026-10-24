
# Active Directory — Domain Deployment and Management

Active Directory — Installation & Configuration

01. Primary Domain Controller (PDC)

Domain Controller (DC): A server responsible for authenticating users, managing accounts, and enforcing domain security policies.

PDC Emulator: One of the five Flexible Single Master Operations (FSMO) roles in Active Directory. It handles tasks such as password change coordination and time synchronization within the domain.

Note: In modern Active Directory, the PDC Emulator role is different from the historical Primary Domain Controller (PDC) concept used in Windows NT.

02. Local Administrator

Before installing Active Directory, sign in to the server using an account with local Administrator privileges.

* Local Administrator: An account that belongs to a specific computer.
* Domain Administrator: An account that belongs to the domain.
* After the server is promoted to a Domain Controller, domain accounts and authentication are managed through the domain infrastructure.

03. Configure Static IP

A server that will function as a Domain Controller should use a static IP address or a DHCP reservation.

Required settings:

* IP Address
* Subnet Mask / Prefix Length
* Default Gateway
* Preferred DNS Server

Note: DNS configuration must match the network environment and installation stage. After the domain is established, the Domain Controller typically uses the internal domain DNS server.

04. Rename the Server

Before installing Active Directory, assign a suitable computer name based on the server’s role.

Example:

DC01

Path:

Server Manager → Local Server → Computer Name

After changing the computer name, restart the server if prompted.

05. Install Active Directory Domain Services (AD DS)

Follow these steps to install the AD DS role:

1. Open Server Manager.
2. Select Manage → Add Roles and Features.
3. Choose Role-based or feature-based installation.
4. Select the destination server.
5. Enable Active Directory Domain Services.
6. Click Add Features when prompted.
7. Continue through the wizard and click Install.

Note: Installing the AD DS role alone does not make the server a Domain Controller. The server must also be promoted.

06. DNS and Active Directory

DNS (Domain Name System) translates names into IP addresses and helps clients locate network services.

Active Directory relies on DNS records, especially SRV records, to locate Domain Controllers and other domain services.

Important points:

* Domain clients must use DNS servers that can resolve Active Directory domain records.
* Incorrect DNS settings can prevent users from signing in or locating a Domain Controller.
* In many deployments, the DNS Server role is installed and configured on the Domain Controller during AD DS promotion.

07. Promote Server to Domain Controller

After installing the AD DS role, follow these steps:

1. Open Server Manager and click the AD DS notification.
2. Select Promote this server to a domain controller.
3. Choose the deployment operation:
    * Add a new forest
    * Add a domain controller to an existing domain
    * Add a new domain to an existing forest
4. Enter the domain name when prompted.
5. Review the available functional level and DNS settings.
6. Set the Directory Services Restore Mode (DSRM) password.
7. Review the prerequisite checks.
8. Click Install if the prerequisite checks pass.
9. The server restarts automatically after the promotion process completes.

Note: To create the first domain in a lab environment, the usual choice is Add a new forest.

08. Verify Installation

After the server restarts, verify the installation by checking the following:

* Sign in using a domain account.
* Confirm that Active Directory Users and Computers is available.
* Confirm that DNS Manager is available.
* Check the status of Active Directory and DNS services.
* Verify that the required DNS records for the domain exist.
* Confirm that the SYSVOL and NETLOGON shares are accessible.

These checks help verify a successful installation. For a complete assessment, also check the health of the domain services and configuration.

09. SAM vs. NTDS.dit

SAM (Security Account Manager)

* Stores information about local Windows accounts.
* Maintains local account data and associated credential information.

NTDS.dit

* The primary Active Directory database.
* Stores information about directory objects, including users, groups, computers, and other domain objects.

After a server is promoted to a Domain Controller, domain directory information is stored in the Active Directory database.

Note: NTDS.dit is not a direct replacement for the SAM database. These databases serve different purposes.

10. DSRM — Directory Services Restore Mode

Directory Services Restore Mode (DSRM) is a special startup mode used for Active Directory maintenance and recovery in specific situations.

* The DSRM password is set during Domain Controller promotion.
* The DSRM account is not used for normal domain user sign-in.
* DSRM is intended for directory service maintenance and recovery scenarios.

11. SYSVOL

SYSVOL is a shared folder on Domain Controllers that contains files required to distribute domain policies and scripts.

Important contents:

* Group Policy Templates
* User logon and logoff scripts
* Files associated with domain policies

Default path:

C:\Windows\SYSVOL

Note: SYSVOL is replicated between Domain Controllers in the domain. In modern Windows Server environments, Distributed File System Replication (DFSR) is the standard replication mechanism.
## 12. Additional Domain Controller (ADC)

An Additional Domain Controller (ADC) provides redundancy and distributes directory services within an existing domain.

### Prerequisites

- Configure a static IP address on the new server.
- Configure the Preferred DNS Server to use an existing internal domain DNS server.
- Ensure network connectivity with an existing Domain Controller.
- Use an account with sufficient domain privileges.

### Installation and Configuration

1. Sign in to the new server using an account with appropriate administrative privileges.
2. Verify the server name and network configuration.
3. Open **Server Manager → Manage → Add Roles and Features**.
4. Select **Role-based or feature-based installation**.
5. Select the destination server.
6. Select **Active Directory Domain Services (AD DS)**.
7. Click **Add Features** and complete the installation.
8. In Server Manager, select **Promote this server to a domain controller**.
9. Select **Add a domain controller to an existing domain**.
10. Enter the domain name and authorized credentials.
11. Configure DNS and Global Catalog options as required.
12. Set the **Directory Services Restore Mode (DSRM)** password.
13. Run the prerequisite checks and select **Install**.
14. After the server restarts, verify DNS and Active Directory replication.

> **Note:** Installing the AD DS role alone does not make the server a Domain Controller. The promotion process is also required.

---

## 13. Active Directory Users and Computers

Use this console to manage domain users, groups, computers, and organizational units (OUs).

**GUI:** `Server Manager → Tools → Active Directory Users and Computers`

**Run command:**

```cmd
dsa.msc
```

---

## 14. Domain Controllers OU

**Path:**

`Active Directory Users and Computers → Domain → Domain Controllers`

The default Domain Controllers OU contains the computer accounts of Domain Controllers. Group Policies linked to this OU can apply settings to those servers.

---

## 15. Computer Accounts

Every domain-joined computer has a computer account in Active Directory.

Computer accounts support computer authentication, identification, and centralized management.

---

## 16. Verify Shares and AD Management Tools

To list the server's shared resources, run:

```cmd
net share
```

To open Active Directory Users and Computers, run:

```cmd
dsa.msc
```

Verify that the expected shares are present and that the management console opens successfully.

> **Note:** These checks alone do not confirm the overall health of Active Directory.

---

## 17. Access SYSVOL and NETLOGON

Open Run or File Explorer and enter:

```text
\\<DomainName>\SYSVOL
\\<DomainName>\NETLOGON
```

Replace `<DomainName>` with the DNS name of your domain.

- **SYSVOL:** Stores Group Policy files and domain-related scripts.
- **NETLOGON:** Provides access to logon scripts and related files.

---

## 18. Active Directory Backup

System State backups help recover Active Directory after server failures or accidental changes.

- Use **Windows Server Backup** or a compatible backup solution.
- Store backups securely, preferably separate from the source server.
- Regularly verify that backups can be restored.

> **Note:** A backup is only useful if the recovery process has been validated.

---

## 19. Create a Child Domain

A Child Domain is a domain created beneath an existing parent domain in the same forest.

**Example:**

```text
example.com
└── child.example.com
```

### Deployment Steps

1. Configure a static IP address and the appropriate DNS settings on the new server.
2. Set the server name and verify connectivity with the parent domain.
3. Install the **Active Directory Domain Services (AD DS)** role.
4. Select **Promote this server to a domain controller**.
5. Select **Add a new domain to an existing forest**.
6. Under **Domain Type**, select **Child Domain**.
7. Specify the parent domain.
8. Enter the child domain name, such as `child`.
9. Provide credentials authorized to create the domain.
10. Configure DNS, set the DSRM password, and review the remaining options.
11. Run the prerequisite checks and start the installation.
12. After the restart, verify the new domain and DNS configuration.

**Required privileges:** Membership in Enterprise Admins is typically required for creating a new domain in an existing forest.

---

## 20. Create a Tree Domain

A Tree Domain is a domain with a different DNS namespace that is added to an existing forest.

**Example:**

```text
Forest: example.com
├── example.com
└── example.net
```

### Deployment Steps

1. Configure a static IP address and DNS settings appropriate for the existing forest.
2. Install the **Active Directory Domain Services (AD DS)** role.
3. Select **Promote this server to a domain controller**.
4. Select **Add a new domain to an existing forest**.
5. Under **Domain Type**, select **Tree Domain**.
6. Specify the existing forest root domain.
7. Enter the full DNS name of the new tree domain, such as `example.net`.
8. Provide credentials authorized to create the domain.
9. Configure DNS, set the DSRM password, and review the remaining options.
10. Run the prerequisite checks and start the installation.
11. After the restart, verify the forest structure and DNS configuration.

**Required privileges:** Membership in Enterprise Admins is typically required for creating a new tree domain in an existing forest.

---

## 21. View the Forest Structure

To view the domain structure:

1. Open **Active Directory Domains and Trusts**.
2. Expand the domain tree.
3. Open the domain properties to review available trust and configuration information.

A forest is the top-level logical structure in Active Directory. Domains in the same forest share a schema and a global catalog infrastructure.

---

## 22. Join a Client Computer to a Domain

### Prerequisites

- Network connectivity to a Domain Controller.
- DNS configured to use an appropriate internal domain DNS server.
- Local administrative privileges on the client.
- Credentials authorized to join the computer to the domain.

### Steps

1. Verify the client's IP and DNS settings.
2. Open the computer name and domain settings.
3. Select the option to change the computer name or domain membership.
4. Select **Domain**.
5. Enter the domain name, such as `example.com`.
6. Enter authorized domain credentials when prompted.
7. After receiving the success message, restart the computer.
8. Sign in using a domain account, for example:

   ```text
   EXAMPLE\username
   ```

9. Open Active Directory Users and Computers and verify the computer account.

> **Note:** The settings path varies by Windows version. Windows Home editions cannot directly join a traditional Active Directory Domain.

---

## 23. Demote a Domain Controller

Demotion removes the Domain Controller role from a server.

### Steps

1. Verify the health of the remaining Domain Controllers and replication.
2. Check FSMO roles, DNS, Global Catalog settings, and dependent services.
3. In Server Manager, open the AD DS section and select **More**.
4. Select **Demote this domain controller**.
5. Review the warnings and provide the required credentials.
6. Do not select the last-domain-controller option unless you intend to remove the domain.
7. Set the local Administrator password for use after demotion.
8. Complete the wizard and restart the server when prompted.
9. Remove the AD DS role if it is no longer needed.

> **Warning:** Do not perform a forced demotion without assessing its impact on replication and Active Directory metadata.

---

## 24. Last Domain Controller in the Domain

The last-domain-controller option is used when demoting the final Domain Controller in a domain.

- Do not remove the last Domain Controller unless domain removal is intended.
- Removing the final Domain Controller removes that domain from the forest.
- If it is the only domain in the forest, removing it eliminates the forest.
- Review dependencies, backups, DNS, and administrative access before proceeding.

**Required privileges:** Membership in Enterprise Admins is typically required to remove the last Domain Controller of a domain in an existing forest.

Active Directory — Domain Join & Computer Account Management

## 25. Joining a Client to a Domain

Procedure

1. Log in to the client using a local administrator account.
2. Open Settings → System → About.
3. Select Domain or workgroup or open System Properties → Computer Name → Change.
4. Select Domain and enter the domain FQDN (e.g., corp.example.com).
5. Enter credentials authorized to join computers to the domain.
6. Restart the client after the confirmation message.
7. At the sign-in screen, select Other user and enter:

CORP\username

Or use the UPN format:

username@corp.example.com

Verification

Open Command Prompt and run:

whoami
echo %logonserver%
nltest /dsgetdc:corp.example.com

* whoami displays the current sign-in identity.
* %logonserver% displays the logon server for the current session.
* nltest locates a Domain Controller for the specified domain.

## 26. Pre-staging a Computer Account

Procedure

1. Open Server Manager → Tools → Active Directory Users and Computers.
2. Navigate to the target OU.
3. Right-click the OU → New → Computer.
4. Enter the client computer name (e.g., PC-01).
5. Select OK.
6. On the client, join the domain using the same computer name and authorized credentials.

Verification

In ADUC, locate PC-01 in the target OU and verify that the computer account exists.

## 27. Computer Account Management

Reset Account

1. Open Active Directory Users and Computers.
2. Locate the computer account.
3. Right-click the account → Reset Account.
4. Confirm the operation when prompted.

Note: Resetting the account can disrupt the existing secure channel. Do not use it as a routine troubleshooting step; follow the appropriate recovery procedure.

Disable Account

1. Open Active Directory Users and Computers.
2. Locate the computer account.
3. Right-click the account → Disable Account.
4. Confirm the operation.

To re-enable it, right-click the account and select Enable Account.

Verification

Run the following command in PowerShell on a machine with the Active Directory module and appropriate permissions:

Get-ADComputer -Identity "PC-01" -Properties Enabled |
Select-Object Name, Enabled

* Enabled = True: The account is enabled.
* Enabled = False: The account is disabled.

Active Directory — Domain Join & Account Management

## 28. Testing and Repairing the Secure Channel

Test Secure Channel

Run PowerShell as Administrator on a domain-joined client:

Test-ComputerSecureChannel

* True: The secure channel is working.
* False: The secure channel may be broken.

Repair Secure Channel

Test-ComputerSecureChannel -Repair -Credential (Get-Credential)

* Prompts for domain credentials and attempts to repair the secure channel.
* Use when troubleshooting a suspected domain trust issue.
* Note: This cmdlet is intended for domain member computers, not Domain Controllers.

⸻

## 29. Pre-staging Computer Accounts

Create a Computer Account Before Domain Join

1. Open Server Manager → Tools → Active Directory Users and Computers.
2. Select the target Organizational Unit (OU).
3. Right-click the OU → New → Computer.
4. Enter the computer name (e.g., PC-01).
5. Select OK.

Set the Default Computer Account Location

Run the following command in an elevated Command Prompt on a Domain Controller:

redircmp "OU=Workstations,DC=corp,DC=example,DC=com"

* Sets the default location for newly created computer accounts.
* Replace the example DN with the distinguished name of an existing OU.
* Does not move existing computer accounts.

Display the Current Default Location:

redircmp

⸻

## 30. User Account Management

Open User Account Properties

1. Open Server Manager → Tools → Active Directory Users and Computers.
2. Navigate to the target OU.
3. Double-click the user account.
4. Open the Account tab.

Log On To

1. Select Log On To.
2. Choose All computers or The following computers.
3. If restricted, add the permitted computer names.
4. Select OK.

Purpose: Restricts the computers from which the user can sign in using domain credentials.

Logon Hours

1. Select Logon Hours in the Account tab.
2. Select the permitted or denied time periods.
3. Apply the changes and select OK.

Purpose: Defines the permitted logon schedule for the account.

Note: This setting does not automatically sign out a user who is already logged in.

Manage User Accounts

In Active Directory Users and Computers, right-click a user account to access these actions:

* Enable/Disable Account: Enables or disables the account.
* Reset Password: Resets the user’s password.
* Properties: Opens account configuration and restrictions.
* Move: Moves the user object to another OU or container.


