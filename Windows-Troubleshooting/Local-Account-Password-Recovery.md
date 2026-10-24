# Local Account Password Recovery

## 1. Boot into Windows Recovery Environment (WinRE)

Windows Recovery Environment (WinRE) provides recovery and troubleshooting tools, including Command Prompt.

### 1.1. From Windows Settings

1. Sign in to Windows.
2. Open **Settings**.
3. Navigate to **System > Recovery**.
4. Under **Advanced startup**, click **Restart now**.
5. Confirm the restart if prompted.
6. Wait for the recovery menu to appear.

### 1.2. From the Sign-in Screen

1. Start the computer and wait for the Windows sign-in screen.
2. Click the **Power** icon.
3. Hold down **Shift**.
4. While holding **Shift**, select **Restart**.
5. Wait for Windows Recovery Environment to open.

## 2. Open Command Prompt in WinRE

1. Select **Troubleshoot**.
2. Select **Advanced options**.
3. Choose **Command Prompt**.
4. Select an administrator account if prompted.
5. Enter the account password if required.
6. Wait for Command Prompt to open.

**Note:** Depending on the device configuration, WinRE may request administrator credentials or a BitLocker recovery key.

## 3. Identify the Windows Installation Drive

Drive letters in WinRE may differ from those used during normal Windows operation. Do not assume that Windows is installed on `C:`.

### 3.1. List Available Volumes

Run:

```cmd
diskpart
list volume
exit
```

Review the volumes and identify the likely Windows partition by its size, file system, and drive letter.

### 3.2. Verify the Windows Directory

Check the likely drive letters individually:

```cmd
dir C:\Windows
dir D:\Windows
dir E:\Windows
```

The correct partition should contain the Windows installation directory and system files.

### 3.3. Navigate to System32

Replace `D:` with the actual Windows installation drive:

```cmd
D:
cd \Windows\System32
dir
```

Verify that the directory contains the expected Windows system files.

## 4. Utilman.exe Method

`utilman.exe` is the Windows Accessibility Manager executable associated with accessibility features on the sign-in screen.

This method involves modifying the accessibility executable to launch Command Prompt from the sign-in screen. Because this can bypass normal Windows authentication, the commands for replacing the executable and using the resulting command shell to reset a password are not included here.

**Warning:** Modifying Windows system files can affect sign-in security and system integrity. Do not leave altered system files in place. Use an authorized recovery procedure to restore the original configuration.

## 5. Security Notes

- Use recovery tools only on systems you own or are authorized to administer.
- Verify the Windows installation drive before working with system directories.
- Keep BitLocker recovery keys and administrator credentials secure.
- Ensure that Windows system files and accessibility features remain in their expected state after troubleshooting.
