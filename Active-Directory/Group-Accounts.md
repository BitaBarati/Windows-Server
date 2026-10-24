## Active Directory — Group Accounts

1. Group Types and Scopes

1.1 Group Types

## Active Directory supports two main group types:

* Security Group: Used to assign permissions to users, computers, and other security principals.
* Distribution Group: Used primarily for email distribution. It cannot be used to assign security permissions.

## GUI Path: Server Manager → Tools → Active Directory Users and Computers (ADUC)

1.2 Group Scopes

Group scope determines which accounts can be members of a group and where the group can be used to assign permissions.

Group Scope	Members	Permission Assignment
Local	Accounts and groups permitted by the local computer’s security model	Resources on the local computer
Domain Local (DL)	Accounts and groups from trusted domains, subject to scope restrictions	Resources within its own domain
Global (GL)	Accounts and Global groups from the same domain	Resources in any trusted domain
Universal (U)	Accounts and Global or Universal groups from any domain in the forest	Resources in any domain in the forest or trusted domains

Important: Domain Local, Global, and Universal are Active Directory domain group scopes. A Local Group is a separate concept and should not be confused with a Domain Local Group.

1.3 Local vs. Domain Local Groups

* Local Group: Exists in the local Security Accounts Manager (SAM) database of an individual computer. It manages permissions for resources on that computer.
* Domain Local Group: Exists in Active Directory and can be used to manage permissions for resources within its own domain. Its membership can include eligible accounts and groups from trusted domains.

Example:

A Domain Local group named DL_FileShare_Read can be granted Read permission on a shared folder in its domain. Eligible users from different domains can be added through the appropriate group nesting.

1.4 Group Scope Membership Rules

## Global (GL)

* Can contain user and computer accounts, and Global groups from the same domain.
* Commonly used to organize users according to department or job role.
* Can be granted permissions to resources in any trusted domain.

## Domain Local (DL)

* Can contain accounts and groups from trusted domains, subject to Active Directory membership rules.
* Commonly used to assign permissions to resources within its own domain.
* Can be used to manage access to file shares, folders, and other domain resources.

## Universal (U)

* Can contain user and computer accounts, Global groups from any domain in the forest, and Universal groups from the forest.
* Commonly used when membership must combine users or Global groups from multiple domains.
* Can be granted permissions to resources across domains.
* Membership changes can affect Global Catalog replication.

1.5 Recommended Permission Management Model

Use the AGDLP model to manage resource access:

* A — Accounts
* G — Global Groups
* DL — Domain Local Groups
* P — Permissions

Example:

1. Add users to GG_HR_Users.
2. Add GG_HR_Users to DL_HR_Folder_Read.
3. Assign Read permission on the HR shared folder to DL_HR_Folder_Read.

## For environments with multiple domains, the AGUDLP model can be used:

* A — Accounts
* G — Global Groups
* U — Universal Groups
* DL — Domain Local Groups
* P — Permissions

1.6 Changing Group Scope

Group scope can be changed in ADUC, but only when the group’s existing membership satisfies the target scope’s restrictions.

From	To	Allowed
Global	Universal	Yes, if the group is not a member of another Global group
Domain Local	Universal	Yes, if the group does not contain another Domain Local group
Universal	Global	Yes, if the group does not contain members from another domain
Universal	Domain Local	Yes

## GUI Steps:

1. Open Server Manager → Tools → Active Directory Users and Computers.
2. Navigate to the OU or container containing the group.
3. Right-click the group and select Properties.
4. Open the General tab.
5. Select the required Group scope.
6. Click Apply → OK.

## Important Notes:

* A Global group cannot contain users or groups from another domain.
* A Domain Local group can contain eligible accounts and groups from trusted domains.
* A Universal group can contain Global groups from any domain in the forest and Universal groups from the forest.
* A Domain Local group cannot be converted directly to Global. Convert it to Universal first, if the membership rules permit it.
* A scope change may be unavailable if existing group memberships violate the target scope’s rules.
* Distribution groups cannot be used to assign security permissions.

1.7 Creating and Managing Groups in ADUC

## Create a Group:

1. Open Active Directory Users and Computers.
2. Navigate to the required OU.
3. Right-click the OU and select New → Group.
4. Enter the group name.
5. Select the Group scope: Domain Local, Global, or Universal.
6. Select the Group type: Security or Distribution.
7. Click OK.

## Manage Group Membership:

1. Open the group’s Properties.
2. Select the Members tab.
3. Click Add to add users, computers, or eligible groups.
4. Use Remove to remove existing members.
5. Click Apply → OK.

1.8 PowerShell — Group Management

Run these commands in a PowerShell session with the Active Directory module available and appropriate permissions.

Import the Active Directory module:

Import-Module ActiveDirectory

## Create a Global Security Group:

## New-ADGroup `
    -Name "GG_HR_Users" `
    -SamAccountName "GG_HR_Users" `
    -GroupScope Global `
    -GroupCategory Security `
    -Path "OU=Groups,DC=contoso,DC=com"

## Create a Domain Local Security Group:

## New-ADGroup `
    -Name "DL_HR_Folder_Read" `
    -SamAccountName "DL_HR_Folder_Read" `
    -GroupScope DomainLocal `
    -GroupCategory Security `
    -Path "OU=Groups,DC=contoso,DC=com"

## Create a Universal Distribution Group:

## New-ADGroup `
    -Name "AllStaff_Email" `
    -SamAccountName "AllStaff_Email" `
    -GroupScope Universal `
    -GroupCategory Distribution `
    -Path "OU=Groups,DC=contoso,DC=com"

## Add a User to a Group:

## Add-ADGroupMember `
    -Identity "GG_HR_Users" `
    -Members "bita"

## Add a Group to Another Group:

## Add-ADGroupMember `
    -Identity "DL_HR_Folder_Read" `
    -Members "GG_HR_Users"

## Remove a Member from a Group:

## Remove-ADGroupMember `
    -Identity "GG_HR_Users" `
    -Members "bita" `
    -Confirm:$false

## Change Group Scope:

Set-ADGroup `
    -Identity "GG_HR_Users" `
    -GroupScope Universal

Use Global, DomainLocal, or Universal as appropriate. The membership restrictions for the target scope still apply.

## View Group Properties:

Get-ADGroup -Identity "GG_HR_Users" -Properties GroupScope, GroupCategory

## View Group Members:

Get-ADGroupMember -Identity "GG_HR_Users"

Verify Nested Group Membership:

Get-ADGroupMember -Identity "DL_HR_Folder_Read" -Recursive

The recursive query displays nested membership. It does not itself prove that a user has effective access to a resource.

1.9 Verification and Important Notes

* Verify the group’s Group scope and Group type in ADUC or PowerShell.
* Confirm that the intended users and groups appear under the Members tab.
* Verify resource permissions separately; membership alone does not grant access unless permissions are assigned.
* Use Global groups to organize accounts, Domain Local groups to manage resource permissions, and Universal groups when cross-domain membership is required.
* Prefer AGDLP or AGUDLP over assigning permissions individually to users.
* In multi-domain environments, consider Global Catalog replication when changing Universal group membership.
