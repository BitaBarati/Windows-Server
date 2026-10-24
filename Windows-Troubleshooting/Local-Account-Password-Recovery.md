# Local Account Password Recovery

## 1. Boot into Windows Recovery Environment (WinRE)

### 1.1. From Windows Settings

1. Open **Settings**.
2. Navigate to **System > Recovery**.
3. Under **Advanced startup**, click **Restart now**.
4. Wait for Windows Recovery Environment to load.

### 1.2. From the Sign-in Screen

1. On the Windows sign-in screen, click the **Power** icon.
2. Hold down **Shift**.
3. While holding **Shift**, select **Restart**.
4. Release Shift when the recovery menu appears.

### 1.3. Open Command Prompt

1. Select **Troubleshoot**.
2. Select **Advanced options**.
3. Select **Command Prompt**.
4. If prompted, select an account and provide the required credentials.

**Note:** BitLocker-protected drives may require a recovery key.

## 2. Identify the Windows Installation Drive

Drive letters in WinRE may differ from those used during normal Windows operation. Verify the correct drive before continuing.

### 2.1. Check the Current System Drive

Run:

```cmd
echo %SystemDrive%
```

This displays the system drive of the current recovery environment. It may not be the installed Windows partition.

### 2.2. List Available Volumes

Run:

```cmd
diskpart
list volume
exit
```

Review the volume list and identify likely Windows partitions by size, file system, and drive letter.

### 2.3. Check the Windows Directory

Test a candidate drive letter. For example:

```cmd
dir D:\Windows
```

If the directory is not found, try other drive letters:

```cmd
dir C:\Windows
dir E:\Windows
```

If the Windows directory and its contents appear, you have likely found the installation drive.

**Important:** Do not assume that Windows is installed on `C:` in WinRE.

## 3. Navigate to the System32 Directory

Replace `D:` with the verified Windows installation drive.

```cmd
D:
cd \Windows\System32
dir
```

Confirm that the directory contents are displayed.

## 4. Utilman.exe Procedure

### 4.1. Understand the Lab Procedure

The referenced procedure modifies the Windows accessibility component (`utilman.exe`) from the Recovery Command Prompt.

This can bypass normal sign-in protections. The executable replacement commands and instructions for launching a command shell from the sign-in screen are not included here.

For an authorized lab:

1. Use a disposable Windows virtual machine.
2. Record the Windows version and installation drive.
3. Document the original state of the accessibility component.
4. Use an approved account-recovery method.
5. Restore any system changes before returning the machine to normal use.

## 5. Local Account Commands

### 5.1. List Local Accounts

From an authenticated Windows Command Prompt, run:

```cmd
net user
```

This displays the local user accounts available in the current Windows environment.

### 5.2. View a Specific Account

Replace `TEST` with the desired local account name:

```cmd
net user TEST
```

This displays information about the specified account.

### 5.3. Change a Local Account Password

Use an authorized administrator session to change the local account password.

For example, open Computer Management:

1. Press `Win + R`.
2. Enter `compmgmt.msc`.
3. Navigate to **System Tools > Local Users and Groups > Users**.
4. Right-click the intended account.
5. Select **Set Password** and follow the prompts.

**Warning:** Resetting a password may make some user-protected data, stored credentials, or certificates inaccessible.

## 6. Verification and Troubleshooting

- **Windows directory not found:** Try another drive letter.
- **BitLocker recovery requested:** Obtain the recovery key from the authorized owner or administrator.
- **Access denied:** Confirm that the selected recovery method has the required authorization.
- **Account not listed:** Verify that you are checking the correct Windows installation.
- **Password change unsuccessful:** Verify the account name and permissions.
- **Domain account:** Domain account passwords are managed separately and may require assistance from the domain administrator.
