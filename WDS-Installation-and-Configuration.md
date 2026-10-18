
# Windows Server — Windows Deployment Services (WDS)

## 01. Introduction to WDS

Windows Deployment Services (WDS) enables network-based Windows deployment using PXE (Preboot Execution Environment).

### Main Components

| Component | Purpose |
|---|---|
| WDS Server | Provides network boot and image deployment services. |
| PXE | Allows clients to boot from the network. |
| Windows ADK | Provides tools for Windows deployment and customization. |
| Windows PE (WinPE) | Provides a lightweight environment for deployment and maintenance. |
| Boot Image | Starts the client in a deployment environment. |
| Install Image | Contains the Windows operating system to be deployed. |
| Capture Image | Boots a reference computer to capture its Windows installation into a WIM file. |
| Answer File | Automates supported Windows Setup and WDS client configuration steps. |

> **Important:** WDS is deprecated starting with the Windows Server release after Windows Server 2025. Windows 11 deployment workflows that use the original Windows installation `boot.wim` and WDS mode are blocked. WDS can still PXE-boot clients using supported custom boot images.

---

## 02. Prerequisites and Deployment Environment

Before installing WDS, prepare the following environment.

### Requirements

- A supported Windows Server installation.
- Administrator privileges.
- A static IP address for the deployment server.
- An accessible NTFS volume for storing deployment images.
- DHCP service to provide IP addresses to PXE clients.
- DNS and Active Directory Domain Services (AD DS) for a domain-based WDS deployment.
- Client computers or virtual machines configured for network boot.
- Windows installation media and the appropriate Windows ADK/WinPE Add-on when required.

### Network Configuration

Example lab environment:

| Component | Example |
|---|---|
| WDS Server | `192.168.10.10` |
| DHCP Server | `192.168.10.5` |
| DNS Server | `192.168.10.10` |
| Domain | `lab.local` |
| Image Storage | `D:\RemoteInstall` |

Replace these example values with your actual lab configuration.

### Important Network Notes

- Ensure the DHCP scope contains valid IP settings, including the subnet mask, gateway, and DNS server.
- WDS and DHCP can run on the same server, but their configuration must account for both services.
- On separate subnets, configure IP helpers on the routers to forward DHCP and PXE requests.
- Do not configure DHCP options 66 and 67 by default as a universal PXE solution. They can cause problems with mixed BIOS and UEFI environments.
- Use an isolated lab network when testing image capture and deployment.

### Verification

Run the following commands on the server:

```powershell
ipconfig /all
Get-Service -Name DHCPServer -ErrorAction SilentlyContinue
Get-Service -Name DNS -ErrorAction SilentlyContinue
```

The service checks are relevant only when the corresponding roles are installed on that server.

---

## 03. Install WDS

### Method 1: Server Manager

1. Open **Server Manager**.
2. Select **Manage → Add Roles and Features**.
3. Select **Role-based or feature-based installation**.
4. Select the target server.
5. Select **Windows Deployment Services**.
6. Include the required management tools.
7. Complete the wizard and install the role.

### Method 2: PowerShell

Run PowerShell as Administrator:

```powershell
Install-WindowsFeature -Name WDS -IncludeManagementTools
```

Verify the installation:

```powershell
Get-WindowsFeature -Name WDS
```

The installation status should indicate that the feature is installed.

> Role installation does not automatically complete the WDS server configuration.

---

## 04. Configure WDS

### Configure Through the WDS Console

1. Open **Server Manager → Tools → Windows Deployment Services**.
2. Expand **Servers**.
3. Right-click the server and select **Configure Server**.
4. Choose the appropriate deployment mode:
   - **Integrated with Active Directory:** For an AD DS environment.
   - **Standalone server:** For supported standalone deployment scenarios.
5. Specify the Remote Installation Folder, for example:

   `D:\RemoteInstall`

6. Configure the PXE response policy.
7. Complete the wizard.

### Configure DHCP and PXE

If DHCP and WDS are installed on the same server:

1. Open the WDS server properties.
2. Select the **DHCP** tab.
3. Enable the options indicating that DHCP and WDS share the same server, as appropriate.
4. Configure the PXE response policy for the intended client computers.

Do not enable PXE responses to unknown clients unless that behavior is required and controlled in your lab.

### Verification

Open an elevated PowerShell window:

```powershell
Get-Service -Name WDSServer
```

Check the WDS configuration:

```cmd
wdsutil /Get-Server /Show:Config
```

If the service is stopped, inspect the WDS configuration and relevant event logs before restarting it.

---

## 05. Prepare Windows ADK and Windows PE

Windows ADK and the WinPE Add-on provide tools for creating and servicing deployment environments.

### Components

- **Windows ADK:** Includes tools such as DISM and Windows System Image Manager (Windows SIM).
- **Windows PE Add-on:** Provides the WinPE files required to create customized boot media.
- **WinPE:** A lightweight bootable environment used for deployment, recovery, and image capture.

### Installation

1. Download the Windows ADK from Microsoft's official documentation.
2. Select the deployment tools required for your scenario.
3. Download and install the matching Windows PE Add-on.
4. Use the ADK version appropriate for the target operating system and deployment solution.


### Create a Working Directory

Open **Deployment and Imaging Tools Environment** as Administrator.

```cmd
copype amd64 C:\WinPE_amd64
```

This creates a working directory for 64-bit WinPE.

Create a bootable ISO:

```cmd
MakeWinPEMedia /ISO C:\WinPE_amd64 C:\WinPE_amd64.iso
```

The ISO can be used to test WinPE in a virtual machine.

> **Important:** A generic WinPE image is not automatically a complete WDS deployment solution. A custom deployment workflow may require networking support, storage drivers, scripts, and a method to access the deployment image.

---

## 06. Create and Add a Boot Image

A boot image starts the client in a preinstallation environment. It is different from the Windows operating system image installed on the client.

### Add a Boot Image

1. Open **Windows Deployment Services**.
2. Expand the server.
3. Right-click **Boot Images**.
4. Select **Add Boot Image**.
5. Browse to the supported boot image, commonly `boot.wim` for supported legacy deployment scenarios.
6. Enter an image name and description.
7. Complete the wizard.

The usual source path on supported Windows installation media is:

`<ISO_DRIVE>:\sources\boot.wim`

Replace `<ISO_DRIVE>` with the mounted ISO drive letter.

### Verification

Expand **Boot Images** in the WDS console and confirm that the image is listed.

> **Windows 11 limitation:** Do not use the Windows 11 installation-media `boot.wim` as a traditional WDS Windows Setup boot image. Windows 11 deployment requires a supported alternative workflow, such as a properly designed custom deployment environment.

---

## 07. Create a Capture Image

A Capture Image is used to boot a reference computer and capture its prepared Windows installation into a WIM file.

### Create the Capture Image

1. Open the WDS console.
2. Expand **Boot Images**.
3. Right-click a supported source boot image.
4. Select **Create Capture Image**.
5. Specify the image name and description.
6. Choose a destination for the generated WIM file.
7. Complete the wizard.
8. Add the generated capture WIM to the server's **Boot Images** collection using **Add Boot Image**.

### Important Notes

- A Capture Image is not an Install Image.
- Capture workflows depend on compatible WinPE and WDS functionality.
- A custom WinPE image may be needed for newer operating systems or specialized hardware.
- Capturing an image does not by itself guarantee that the image can be deployed successfully to different hardware.

### Verification

Confirm that the Capture Image appears in **Boot Images**.

---

## 08. Prepare and Capture a Reference Computer

A reference computer is the machine whose Windows installation will be captured.

### Prepare the Reference Computer

1. Install the intended Windows edition using a supported method.
2. Install required applications, updates, and configuration settings.
3. Remove temporary files and unnecessary user-specific data.
4. Confirm that the image does not contain credentials or other sensitive information.
5. Generalize the installation when preparing a reusable Windows image.

Run the following command on the reference computer from an elevated Command Prompt:

```cmd
%WINDIR%\System32\Sysprep\Sysprep.exe /generalize /oobe /shutdown
```

This prepares the installation for imaging and shuts down the computer.

> Sysprep should be used only when appropriate for the intended image workflow. Validate application compatibility and Windows activation requirements before generalizing the reference installation.

### Boot into the Capture Environment

1. Start the reference computer using the supported capture boot image.
2. Select network boot or the appropriate WinPE boot method.
3. Follow the capture wizard.
4. Select the Windows volume to capture.
5. Specify the image name and description.
6. Save the resulting WIM file to an accessible destination, such as a network share or supported WDS capture destination.

The exact destination options depend on the capture environment.

### Verification

Inspect the captured WIM using DISM:

```cmd
dism /Get-WimInfo /WimFile:D:\Images\Reference.wim
```

Replace the example path with the actual location of the captured image.

---

## 09. Add an Install Image

An Install Image contains a Windows operating system image that can be applied to a target computer by a compatible deployment workflow.

### Add an Image Group

1. Open the WDS console.
2. Right-click **Install Images**.
3. Select **Add Image Group**.
4. Enter a descriptive name.

### Add an Install Image

1. Right-click the appropriate image group.
2. Select **Add Install Image**.
3. Browse to the prepared WIM file.
4. Select the required image index or edition when prompted.
5. Complete the wizard.

A captured WIM may contain a prepared reference installation. An original Windows installation WIM may contain several editions.

### Verification

Expand **Install Images** and confirm that the expected image and edition are listed.

> Adding an image to WDS does not mean that every operating system version can be installed through the traditional WDS client. Check the supported boot and deployment workflow before attempting deployment.

---

## 10. Deploy a Windows Image to Client Computers

### Traditional WDS Workflow

For supported legacy deployment scenarios:

1. Connect the client to the deployment network.
2. Configure the firmware to boot from the network.
3. Select the appropriate network boot option.
4. Load the supported WDS boot image.
5. Authenticate if required.
6. Select the appropriate Install Image.
7. Select the destination disk and complete the installation.
8. Allow Windows Setup to complete.

### BIOS and UEFI

| Firmware | Key Considerations |
|---|---|
| Legacy BIOS | Commonly uses MBR partitioning and legacy network boot. |
| UEFI | Commonly uses GPT partitioning and UEFI-compatible network boot files. |
| Secure Boot | Requires a compatible, trusted boot chain. |

Use the boot architecture and network boot configuration appropriate to the client firmware.

### Windows 11 Deployment

Traditional end-to-end deployment using the Windows 11 installation `boot.wim` in WDS mode is not supported.

For Windows 11, choose a supported deployment solution or a validated custom WinPE workflow. Microsoft Configuration Manager is one alternative for managed deployments.

---

## 11. Create and Configure an Answer File

An answer file (`unattend.xml`) automates supported Windows Setup configuration stages.

### Create an Answer File

1. Install the Windows ADK deployment tools.
2. Open **Windows System Image Manager (Windows SIM)**.
3. Select the appropriate Windows image or catalog.
4. Create a new answer file.
5. Add only the settings required for the deployment scenario.
6. Validate the file and save it as `unattend.xml`.

### Common Uses

- Language and regional settings.
- Windows Setup configuration.
- Computer naming, where supported.
- OOBE configuration.
- Supported account and domain-join settings.

### WDS Client Unattend Configuration

For supported WDS workflows:

1. Open the WDS console.
2. Right-click the server and select **Properties**.
3. Open the **Client** tab.
4. Configure the applicable client unattend file for the relevant architecture.
5. Use a separate Windows Setup answer file when the workflow requires one.

The WDS client answer file and the Windows Setup answer file serve different stages of deployment.

### Security Notes

- Never store plaintext passwords in a publicly accessible repository.
- Avoid embedding reusable credentials in answer files.
- Restrict access to deployment shares and unattended configuration files.
- Validate answer-file settings against the target Windows version.

---

## 12. Verify Deployment

After deployment, verify the client configuration.

### Check Windows Version

```cmd
winver
```

### Check IP Configuration

```cmd
ipconfig /all
```

### Check Disk and Partition Information

```powershell
Get-Disk
Get-Partition
Get-Volume
```

### Check Computer Identity

```powershell
hostname
```

### Check Domain Membership

```powershell
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, PartOfDomain
```

### Verify the Deployment

- Confirm that Windows starts successfully.
- Confirm the expected edition and configuration.
- Check network connectivity and DNS resolution.
- Confirm that required applications and drivers are installed.
- Verify domain membership when applicable.
- Check Windows activation and update status.

---

## 13. Troubleshooting and Useful Commands

### Check WDS Service

```powershell
Get-Service -Name WDSServer
```

### Start or Restart WDS

Run as Administrator:

```powershell
Start-Service -Name WDSServer
```

If the service is already running and a restart is necessary:

```powershell
Restart-Service -Name WDSServer
```

Restarting the service can interrupt active PXE or deployment sessions.

### Display WDS Configuration

```cmd
wdsutil /Get-Server /Show:Config
```

### Check Installed WDS Features

```powershell
Get-WindowsFeature -Name WDS
```

### Inspect a WIM Image

```cmd
dism /Get-WimInfo /WimFile:D:\Images\Reference.wim
```

Replace the example path with the actual image location.

### Common Problems

| Problem | Checks |
|---|---|
| Client cannot PXE boot | Check DHCP, IP helpers, firmware boot mode, and WDS service status. |
| Client receives no IP address | Check DHCP scope, VLAN, routing, and network connectivity. |
| Boot image fails to load | Check architecture compatibility, image integrity, and supported deployment scenarios. |
| Windows Setup reports a missing media driver | Check boot-image compatibility and storage/network drivers. On Windows Server 2025, traditional installation-media `boot.wim` workflows are unsupported. |
| Install Image is unavailable | Check the image group, image status, and permissions. |
| Capture fails | Check Sysprep status, disk selection, network access, destination permissions, and available storage. |
| Unattended installation fails | Validate the answer file, configuration passes, architecture, and Windows version. |
| Deployment works on BIOS but not UEFI | Check firmware settings, architecture, PXE configuration, and boot-file compatibility. |
