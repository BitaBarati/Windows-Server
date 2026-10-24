# Active Directory — Group Policy Management & Configuration

## 01. Introduction to Group Policy Objects (GPOs)

A Group Policy Object (GPO) centrally manages user and computer settings in an Active Directory domain.

### Main Components

* **Computer Configuration:** Applies settings to computers.
* **User Configuration:** Applies settings to users.
* **GPMC:** Group Policy Management Console.
* **GPC:** Stores GPO information in Active Directory.
* **GPT:** Stores GPO files in SYSVOL.

### Open Group Policy Management

    gpmc.msc

### GPO Processing Order

1. Local
2. Site
3. Domain
4. Organizational Unit (OU)

Later applicable policies can override earlier settings when conflicts occur, subject to inheritance, enforcement, and policy-specific rules.

---

## 02. Creating and Managing GPOs

### Create a GPO

1. Open `gpmc.msc`.
2. Expand Forest → Domains → YourDomain.
3. Right-click the target domain or OU.
4. Select **Create a GPO in this domain, and Link it here**.
5. Enter a name, such as `Workstation-Policy`.
6. Right-click the GPO and select **Edit**.

### Example: Disable Control Panel

Navigate to:

    User Configuration
      → Policies
        → Administrative Templates
          → Control Panel
            → Prohibit access to Control Panel

Set the policy to **Enabled**.

### Create and Link a GPO Using PowerShell

    Import-Module GroupPolicy
    New-GPO -Name "Workstation-Policy"
    New-GPLink `
        -Name "Workstation-Policy" `
        -Target "OU=Workstations,DC=example,DC=com"

Replace the example OU distinguished name with the actual Active Directory path.

### Apply Policies

    gpupdate /force

---

## 03. Group Policy Processing

Group Policy processing determines which settings apply to users and computers.

### Important Rules

* Policies are processed in LSDOU order.
* Child OU policies normally take precedence over conflicting parent OU settings.
* Enforced links can prevent lower-level policies from overriding conflicting settings.
* Block Inheritance prevents normal inheritance from higher levels, except enforced links.
* Security filtering determines which users or computers can apply a GPO.

### Check Applied Policies

    gpresult /r

Generate an HTML report:

    gpresult /h C:\Temp\GPReport.html /f

Create `C:\Temp` first if it does not exist.

### Refresh Computer Policy

    gpupdate /target:computer /force

### Refresh User Policy

    gpupdate /target:user /force

---

## 04. WMI Filtering

WMI filtering applies a GPO only when the specified WMI query returns a matching result.

### Create a WMI Filter

1. Open GPMC.
2. Navigate to **WMI Filters**.
3. Select **New**.
4. Enter a name.
5. Add a WMI query.

Example: Match client operating systems.

    SELECT * FROM Win32_OperatingSystem
    WHERE ProductType = "1"

6. Save the filter.
7. Select the filter in the GPO’s WMI Filtering section.

### Test the Query

    Get-CimInstance Win32_OperatingSystem |
        Select-Object Caption, ProductType, Version

`ProductType = "1"` identifies client operating systems; server operating systems have different values.

---

## 05. GPResult and Group Policy Modeling

### GPResult

`gpresult` displays the Resultant Set of Policy (RSoP) for a user or computer.

    gpresult /r

Computer policies:

    gpresult /scope computer /r

User policies:

    gpresult /scope user /r

Generate a report:

    gpresult /h C:\Temp\GPReport.html /f

### Group Policy Modeling

1. Open GPMC.
2. Right-click **Group Policy Modeling**.
3. Select **Group Policy Modeling Wizard**.
4. Select the domain, user, computer, and OU.
5. Complete the wizard.
6. Review the simulated policy results.

**Note:** Group Policy Modeling simulates policy application; it does not apply the modeled settings to the actual computer.

---

## 06. GPO Backup, Restore, Import and Migration

Use the GroupPolicy PowerShell module to manage GPO backups.

### Back Up a GPO

    Backup-GPO `
        -Name "Workstation-Policy" `
        -Path "C:\GPO-Backups"

Back up all GPOs:

    Backup-GPO -All -Path "C:\GPO-Backups"

### Restore a GPO

    Restore-GPO `
        -Name "Workstation-Policy" `
        -Path "C:\GPO-Backups"

### Import Settings into Another GPO

    Import-GPO `
        -BackupGpoName "Workstation-Policy" `
        -Path "C:\GPO-Backups" `
        -TargetName "Workstation-Policy-Test" `
        -CreateIfNeeded

### Important Differences

* **Backup-GPO:** Creates a backup of GPO settings.
* **Restore-GPO:** Restores settings to the original GPO.
* **Import-GPO:** Imports settings into a target GPO.
* **Copy-GPO:** Copies a GPO to another location or domain.

A GPO backup does not replace a full Active Directory or system-state backup.

---

## 07. GPO Delegation

GPO delegation allows administrators to manage policies without granting unnecessary domain-wide privileges.

### Configure Delegation in GPMC

1. Open GPMC.
2. Select the target GPO.
3. Open the **Delegation** tab.
4. Select **Add**.
5. Choose a user or security group.
6. Assign the required permission.

Common permissions include:

* Read
* Edit settings
* Edit settings, delete, modify security

### Delegate GPO Editing with PowerShell

    Set-GPPermission `
        -Name "Workstation-Policy" `
        -TargetName "GPO-Admins" `
        -TargetType Group `
        -PermissionLevel GpoEdit

Replace `GPO-Admins` with the actual security group name.

**Note:** Editing a GPO and linking a GPO are different permissions. Grant only the access required for the task.

---

## 08. Windows Firewall Policy

Windows Defender Firewall can be configured centrally through Group Policy.

### Configuration Path

    Computer Configuration
      → Policies
        → Windows Settings
          → Security Settings
            → Windows Defender Firewall with Advanced Security

Configure:

* Domain Profile
* Private Profile
* Public Profile
* Inbound Rules
* Outbound Rules

### Create a Firewall Rule

The following command creates a local firewall rule. Run it in an elevated PowerShell session.

    New-NetFirewallRule `
        -DisplayName "Allow TCP 8080" `
        -Direction Inbound `
        -Protocol TCP `
        -LocalPort 8080 `
        -Action Allow `
        -Profile Domain

To deploy a rule through GPO, create it under Inbound Rules or Outbound Rules in the Group Policy editor.

### Verify Firewall Profiles

    Get-NetFirewallProfile

List firewall rules:

    Get-NetFirewallRule |
        Select-Object DisplayName, Enabled, Direction, Action

**Security note:** Avoid disabling the firewall domain-wide. Allow only required traffic.

---

## 09. Software Installation Using Group Policy

Group Policy Software Installation supports deployment of Windows Installer packages (`.msi`).

### Prepare the Package

1. Create a shared folder on a file server.
2. Copy the MSI package into the folder.
3. Grant target computers the required share and NTFS permissions.
4. Use a UNC path.

Example:

    \\FILE01\Software\AppName.msi

### Configure Software Installation

Navigate to:

    Computer Configuration
      → Policies
        → Software Settings
          → Software Installation

1. Right-click **Software Installation**.
2. Select **New → Package**.
3. Enter the UNC path.
4. Select **Assigned**.

The package is assigned to computers within the GPO’s scope.

### Apply and Verify

    gpupdate /force

Computer-assigned software may install during startup processing. Restart the computer if required.

**Note:** Standard Group Policy Software Installation does not directly deploy arbitrary EXE installers.

---

## 10. AppLocker

AppLocker controls which applications and scripts users can execute.

### Configuration Path

    Computer Configuration
      → Policies
        → Windows Settings
          → Security Settings
            → Application Control Policies
              → AppLocker

### Configure AppLocker

1. Open **Configure rule enforcement**.
2. Set the required rule collections to **Audit only**.
3. Create default rules.
4. Add the required allow or deny rules.
5. Review audit events.
6. Enable enforcement after testing.

### Configure the Application Identity Service

Run in an elevated PowerShell session:

    Set-Service -Name AppIDSvc -StartupType Automatic
    Start-Service -Name AppIDSvc

Verify the service:

    Get-Service AppIDSvc

### Review Events

    Event Viewer
      → Applications and Services Logs
        → Microsoft
          → Windows
            → AppLocker

**Security note:** Test in audit mode before enforcement. Incorrect rules may block legitimate applications. AppLocker support and enforcement capabilities depend on the Windows edition and version.

---

## 11. Password Policies

Password policies control domain password requirements and account lockout behavior.

### Configuration Path

    Computer Configuration
      → Policies
        → Windows Settings
          → Security Settings
            → Account Policies
              → Password Policy

Account lockout settings are available under **Account Lockout Policy**.

### View the Default Domain Password Policy

    Get-ADDefaultDomainPasswordPolicy

### Configure the Domain Password Policy

Example configuration:

    Set-ADDefaultDomainPasswordPolicy `
        -Identity "example.com" `
        -MinPasswordLength 14 `
        -ComplexityEnabled $true `
        -ReversibleEncryptionEnabled $false

Replace `example.com` with the actual domain DNS name.

### Fine-Grained Password Policies

Create a policy for accounts requiring stricter password settings:

    New-ADFineGrainedPasswordPolicy `
        -Name "Privileged-Accounts-Policy" `
        -Precedence 10 `
        -MinPasswordLength 16 `
        -ComplexityEnabled $true `
        -ReversibleEncryptionEnabled $false

Apply it to a security group:

    Add-ADFineGrainedPasswordPolicySubject `
        -Identity "Privileged-Accounts-Policy" `
        -Subjects "Privileged-Users"

Check a user’s resultant policy:

    Get-ADUserResultantPasswordPolicy -Identity "user1"

**Important:** Fine-Grained Password Policies apply to users and global security groups, not directly to OUs. Avoid reversible password encryption unless there is a documented exceptional requirement.

---

## 12. User Rights Assignment and Restricted Groups

### User Rights Assignment

User Rights Assignment defines privileges such as local logon, network access, and Remote Desktop logon.

Configuration path:

    Computer Configuration
      → Policies
        → Windows Settings
          → Security Settings
            → Local Policies
              → User Rights Assignment

Examples:

* Allow log on locally
* Allow log on through Remote Desktop Services
* Access this computer from the network
* Deny log on locally

### Restricted Groups

Restricted Groups centrally controls group membership.

Configuration path:

    Computer Configuration
      → Policies
        → Windows Settings
          → Security Settings
            → Restricted Groups

1. Right-click **Restricted Groups**.
2. Select **Add Group**.
3. Select the target group.
4. Configure either:
    * Members of this group
    * This group is a member of

**Important:** Defining Members of this group can enforce an exact membership list and remove unlisted members. Test changes before applying them broadly.

---

## 13. Loopback Processing and Slow Link Detection

### Loopback Processing

Loopback processing changes how User Configuration settings are selected on specific computers.

Configuration path:

    Computer Configuration
      → Policies
        → Administrative Templates
          → System
            → Group Policy
              → Configure user Group Policy loopback processing mode

Available modes:

* **Merge:** Combines the user’s normal policies with policies based on the computer. Computer-based user settings take precedence when they conflict.
* **Replace:** Uses user settings based on the computer instead of the user’s normal user policy.

Typical use cases include kiosks, classrooms, and Remote Desktop Session Hosts.

### Slow Link Detection

Configuration path:

    Computer Configuration
      → Policies
        → Administrative Templates
          → System
            → Group Policy
              → Configure Group Policy slow link detection

Configure the threshold according to the network environment.

### Verify the Result

    gpupdate /force
    gpresult /h C:\Temp\GPReport.html /f

**Note:** Loopback is a computer-side setting and must be configured on the computers where it is required.

---

## 14. Kerberos Policies

Kerberos is an authentication protocol used by Active Directory.

### Configuration Path

    Computer Configuration
      → Policies
        → Windows Settings
          → Security Settings
            → Account Policies
              → Kerberos Policy

Important settings include:

* Enforce user logon restrictions
* Maximum lifetime for service tickets
* Maximum lifetime for user tickets
* Maximum lifetime for user ticket renewal
* Maximum tolerance for computer clock synchronization

### View Kerberos Tickets

    klist

Purge the current logon session’s ticket cache:

    klist purge

Purging tickets can temporarily interrupt access to services until new tickets are obtained.

### Check Time Synchronization

    w32tm /query /status

**Important:** Kerberos depends on correct time synchronization. Avoid changing ticket lifetimes or encryption settings without a clear requirement.

---

## 15. PolicyPak

PolicyPak is a third-party product that extends Group Policy-based configuration and endpoint management.

### General Deployment Workflow

1. Confirm the required PolicyPak product and license.
2. Install the required server-side components and client-side extensions according to the vendor’s documentation.
3. Open the supported management console or Group Policy integration.
4. Create the required policy.
5. Link the GPO to the target domain or OU.
6. Configure security filtering and targeting.
7. Test on a limited number of computers.
8. Verify policy processing on the clients.

### Verify Standard Group Policy Processing

    gpupdate /force
    gpresult /h C:\Temp\GPReport.html /f

Review PolicyPak-specific logs and management tools for product-related errors.

