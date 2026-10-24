# Local Account Password Recovery

## 1. Boot into Windows Recovery Environment (WinRE)

Windows Recovery Environment (WinRE) provides recovery and troubleshooting tools, including Command Prompt.

### 1.1. From Windows Settings

1. Sign in to Windows using an account with permission to access recovery settings.
2. Open **Settings**.
3. Navigate to **System > Recovery**.
4. Under **Advanced startup**, click **Restart now**.
5. Confirm the restart if prompted.
6. Wait for the Windows Recovery Environment menu to appear.

### 1.2. From the Sign-in Screen

1. Start the computer and wait for the Windows sign-in screen.
2. Click the **Power** icon.
3. Hold down **Shift**.
4. While holding **Shift**, select **Restart**.
5. Continue holding Shift until the recovery process begins.
6. Wait for the Windows Recovery Environment menu to appear.

**Note:** The `Shift + Restart` method may be unavailable on some systems or managed devices. If Windows cannot start normally, use Windows installation or recovery media and select the repair options.

## 2. Open Command Prompt in WinRE

1. In the recovery menu, select **Troubleshoot**.
2. Select **Advanced options**.
3. Choose **Command Prompt**.
4. Select an administrator account if prompted.
5. Enter its password if required.
6. Wait for the Command Prompt window to open.

**Important:** WinRE may request an account password or a BitLocker recovery key. Access to recovery tools does not automatically grant permission to access encrypted or protected data.

## 3. Identify the Windows Installation Drive

Drive letters in WinRE may differ from those assigned during normal Windows operation. Do not assume that Windows is installed on `C:`.

### 3.1. List Available Volumes

Run:

```cmd
diskpart
list volume
exit
```

Review the listed volumes and identify the likely Windows partition by its size, file system, and drive letter.

### 3.2. Verify the Windows Directory

Check likely drive letters individually. For example:

```cmd
dir C:\Windows
dir D:\Windows
dir E:\Windows
```

The correct partition should contain the Windows installation directory and system files.

**Example:** If `D:\Windows` exists and contains Windows system directories, `D:` may be the Windows installation drive.

### 3.3. Navigate to System32

After identifying the correct drive, navigate to its System32 directory. For example, if Windows is installed on `D:`:

```cmd
D:
cd \Windows\System32
dir
```

Verify that the directory contents correspond to the expected Windows system files.

**Warning:** Do not modify, rename, or replace Windows system files unless you are following a supported recovery procedure for an authorized system.

## 4. Reset a Local Account Password Using Supported Methods

Choose the recovery method that matches the account and the access available.

### 4.1. Use the Sign-in Screen Password Reset Option

1. Return to the Windows sign-in screen.
2. Select the relevant local account.
3. If available, select **Reset password**.
4. Answer the configured security questions.
5. Follow the prompts to set a new password.

This method is available only when the account and Windows configuration support it.

### 4.2. Use a Password Reset Disk

If a password reset disk was created before the password was lost:

1. Connect the reset disk.
2. On the sign-in screen, enter an incorrect password once.
3. Select **Reset password** if it appears.
4. Follow the Password Reset Wizard.
5. Set a new password and complete the wizard.

A password reset disk is generally specific to the local account for which it was created.

### 4.3. Use Another Administrator Account

If another authorized local administrator account is available:

1. Sign in with that administrator account.
2. Open **Computer Management**.
3. Navigate to **Local Users and Groups > Users**, if supported by the Windows edition.
4. Right-click the target local account.
5. Select **Set Password**.
6. Review the warning and confirm only if authorized.

Alternatively, open Command Prompt or PowerShell as an administrator and list local accounts:

```cmd
net user
```

To change a local account password interactively, use:

```cmd
net user TEST *
```

Replace `TEST` with the actual local account name. Enter and confirm the new password when prompted. Do not share or store passwords in documentation or command history.

**Important:** An administrator-forced password reset may affect access to certain EFS-encrypted files, stored credentials, or personal certificates. Review the implications before proceeding.

## 5. Verify the Recovery

1. Return to the Windows sign-in screen.
2. Select the intended local account.
3. Enter the new password.
4. Confirm that sign-in succeeds.
5. If sign-in fails, verify the account name, keyboard layout, and selected sign-in method.

## 6. Troubleshooting Notes

- **Windows is not on C:** Use `diskpart` and `dir` to identify the correct installation drive.
- **Command Prompt requests credentials:** Use an authorized administrator account or an appropriate recovery method.
- **BitLocker recovery is requested:** Obtain the recovery key through the device owner's authorized recovery process.
- **Reset password is unavailable:** Check whether security questions, a password reset disk, or another administrator account is available.
- **The account belongs to a domain:** Contact the domain administrator; local-account recovery procedures may not apply.
- **Windows Home edition:** The Local Users and Groups management snap-in may not be available.

## 7. Security Considerations

- Use recovery tools only on devices you own or are authorized to administer.
- Prefer supported account recovery methods over modifications to Windows sign-in system files.
- Keep recovery keys and administrator credentials secure.
- After recovery, review local administrator accounts and verify that Windows security settings remain intact.
