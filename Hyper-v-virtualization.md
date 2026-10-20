# 02 - Hyper-V Virtualization

## 1. Introduction to Virtualization

### 1.1. Overview

Virtualization allows multiple virtual machines (VMs) to run on a single physical computer. Microsoft Hyper-V is a hypervisor available on supported Windows Server and Windows client editions.

Hyper-V enables administrators to create, configure, manage, and monitor virtual machines while sharing the host's physical resources.

### 1.2. Key Components

| Component | Description |
|---|---|
| Hyper-V Host | Physical server running the Hyper-V role |
| Virtual Machine (VM) | Virtual computer running an operating system |
| Virtual CPU (vCPU) | Processor resources assigned to a VM |
| Virtual Memory | RAM allocated to a VM |
| Virtual Hard Disk (VHDX) | File used as a VM's virtual storage |
| Virtual Switch | Connects VMs to each other and to physical networks |
| Checkpoint | Captures a VM's state at a point in time |
| Hyper-V Replica | Replicates VMs to another Hyper-V host |
| Failover Cluster | Provides high availability for supported workloads |

### 1.3. Lab Environment

The following examples use Windows Server 2022 or Windows Server 2025. Most core commands also apply to Windows Server 2016 and 2019, although some features and options may differ.

| Component | Example |
|---|---|
| Hyper-V host | `HV01` |
| Host operating system | Windows Server |
| VM name | `VM01` |
| VM generation | Generation 2 |
| VM memory | 4 GB |
| Virtual disk | 60 GB |
| VM operating system | Windows Server |
| VM storage | `D:\Hyper-V\VMs` |
| Virtual switch | `ExternalSwitch` |

**Important:** Run PowerShell commands as Administrator on the Hyper-V host unless otherwise specified. Replace example names, paths, and IP addresses with values appropriate for your lab.

---

## 2. Install Hyper-V

### 2.1. Prerequisites

Before installing Hyper-V, verify the following:

- The physical processor supports hardware virtualization.
- Hardware virtualization is enabled in BIOS/UEFI.
- Hardware Data Execution Prevention (DEP) is supported and enabled.
- Sufficient RAM and disk space are available.
- The operating system edition supports the Hyper-V role.
- The server has a stable network configuration.

Check the system's virtualization capabilities:

```powershell
Get-ComputerInfo |
    Select-Object HyperVRequirement*
```

Review the processor and operating system information:

```powershell
Get-ComputerInfo |
    Select-Object CsName, WindowsProductName, OsVersion
```

> [!WARNING]
> If hardware virtualization is disabled in BIOS/UEFI, Hyper-V may fail to start or the installation may not work as expected.

### 2.2. Install Hyper-V Using Server Manager

1. Open **Server Manager**.
2. Select **Manage > Add Roles and Features**.
3. Select **Role-based or feature-based installation**.
4. Select the target server.
5. Select the **Hyper-V** role.
6. Add the required management tools.
7. Select **Next** and configure virtual switches if appropriate.
8. Complete the wizard and restart the server when prompted.

### 2.3. Install Hyper-V Using PowerShell

Install the Hyper-V role and management tools:

```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

After the restart, verify the role:

```powershell
Get-WindowsFeature -Name Hyper-V
```

Check whether the Hyper-V PowerShell module is available:

```powershell
Get-Module -ListAvailable Hyper-V
```

Import the module:

```powershell
Import-Module Hyper-V
```

### 2.4. Configure Default VM Locations

Create directories for virtual machines and virtual hard disks:

```powershell
New-Item -ItemType Directory -Path "D:\Hyper-V\VMs" -Force
New-Item -ItemType Directory -Path "D:\Hyper-V\VHDs" -Force
```

Configure the default paths:

```powershell
Set-VMHost `
    -VirtualMachinePath "D:\Hyper-V\VMs" `
    -VirtualHardDiskPath "D:\Hyper-V\VHDs"
```

Verify the configuration:

```powershell
Get-VMHost |
    Select-Object VirtualMachinePath, VirtualHardDiskPath
```

**Note:** Choose storage with sufficient capacity and appropriate performance. Do not place production VM files on an unreliable or nearly full volume.

### 2.5. Verify Hyper-V Installation

Check the Hyper-V Virtual Machine Management service:

```powershell
Get-Service -Name vmms
```

Display Hyper-V host information:

```powershell
Get-VMHost
```

List existing virtual machines:

```powershell
Get-VM
```

The `vmms` service manages Hyper-V virtual machine operations.

---

## 3. Create and Manage Virtual Machines

### 3.1. Create a VM Using Hyper-V Manager

1. Open **Server Manager > Tools > Hyper-V Manager**.
2. Select the Hyper-V host.
3. Select **New > Virtual Machine**.
4. Enter the VM name, such as `VM01`.
5. Select the VM storage location.
6. Choose **Generation 2** for supported modern guest operating systems.
7. Assign startup memory.
8. Select a virtual switch, if one has already been created.
9. Create a virtual hard disk.
10. Select an operating system installation source.
11. Complete the wizard and start the VM.

### 3.2. Create a VM Using PowerShell

Create a Generation 2 VM with 4 GB of startup memory and a 60 GB virtual disk:

```powershell
New-VM `
    -Name "VM01" `
    -Generation 2 `
    -MemoryStartupBytes 4GB `
    -NewVHDPath "D:\Hyper-V\VHDs\VM01.vhdx" `
    -NewVHDSizeBytes 60GB `
    -Path "D:\Hyper-V\VMs"
```

Connect the VM to an existing virtual switch:

```powershell
Connect-VMNetworkAdapter `
    -VMName "VM01" `
    -SwitchName "ExternalSwitch"
```

Set the number of virtual processors:

```powershell
Set-VMProcessor -VMName "VM01" -Count 2
```

Configure dynamic memory:

```powershell
Set-VMMemory `
    -VMName "VM01" `
    -DynamicMemoryEnabled $true `
    -StartupBytes 4GB `
    -MinimumBytes 2GB `
    -MaximumBytes 8GB
```

Configure automatic start and stop actions:

```powershell
Set-VM `
    -Name "VM01" `
    -AutomaticStartAction StartIfRunning `
    -AutomaticStopAction ShutDown
```

Start the VM:

```powershell
Start-VM -Name "VM01"
```

Check its status:

```powershell
Get-VM -Name "VM01"
```

### 3.3. Install an Operating System

1. Obtain a compatible operating system ISO.
2. Open **VM Settings**.
3. Select **SCSI Controller > DVD Drive**.
4. Attach the ISO file.
5. Configure the VM boot order if necessary.
6. Start the VM and connect through VMConnect.
7. Complete the operating system installation.

Attach an ISO using PowerShell:

```powershell
Add-VMDvdDrive `
    -VMName "VM01" `
    -Path "D:\ISO\WindowsServer.iso"
```

Set the DVD drive as the first boot device when required:

```powershell
$dvd = Get-VMDvdDrive -VMName "VM01"

Set-VMFirmware `
    -VMName "VM01" `
    -FirstBootDevice $dvd
```

**Important:**

- The ISO path must exist on the Hyper-V host.
- Generation 2 VMs use UEFI firmware and generally require compatible boot media.
- Generation 1 may be needed for older operating systems.
- Secure Boot settings must be compatible with the guest operating system and installation media.

### 3.4. Manage VM State

Start a VM:

```powershell
Start-VM -Name "VM01"
```

Request a graceful shutdown through the guest operating system:

```powershell
Stop-VM -Name "VM01"
```

Turn off a VM immediately:

```powershell
Stop-VM -Name "VM01" -TurnOff
```

Pause a VM:

```powershell
Suspend-VM -Name "VM01"
```

Resume a paused VM:

```powershell
Resume-VM -Name "VM01"
```

View VM status:

```powershell
Get-VM -Name "VM01"
```

> [!WARNING]
> `-TurnOff` is equivalent to removing power from a physical computer and may cause data loss. Prefer a graceful shutdown whenever possible.

### 3.5. Modify VM Settings

Change the virtual processor count:

```powershell
Set-VMProcessor -VMName "VM01" -Count 4
```

Change startup memory:

```powershell
Set-VMMemory -VMName "VM01" -StartupBytes 6GB
```

View VM configuration:

```powershell
Get-VM -Name "VM01" | Format-List *
```

View virtual network adapters:

```powershell
Get-VMNetworkAdapter -VMName "VM01"
```

Changing startup memory or processor settings may require the VM to be shut down, depending on the setting and configuration.

---

## 4. Export, Import, and Checkpoint VMs

### 4.1. Export a VM

Exporting a VM creates a portable copy of its configuration and associated files.

Create an export directory:

```powershell
New-Item -ItemType Directory -Path "E:\VM-Exports" -Force
```

Export the VM:

```powershell
Export-VM `
    -Name "VM01" `
    -Path "E:\VM-Exports"
```

Verify the exported files:

```powershell
Get-ChildItem "E:\VM-Exports" -Recurse
```

**Important:** Ensure the destination has enough free space. Exporting a running VM may be supported, but plan the operation according to the workload and backup requirements.

### 4.2. Import a VM

In Hyper-V Manager:

1. Select **Import Virtual Machine**.
2. Browse to the exported VM directory.
3. Select the VM.
4. Choose the appropriate import type.
5. Complete the wizard.

PowerShell example:

```powershell
Import-VM `
    -Path "E:\VM-Exports\VM01\Virtual Machines\<VM-ID>.vmcx" `
    -Copy `
    -GenerateNewId
```

Replace `<VM-ID>` with the actual configuration filename from the export.

Common import options:

| Option | Purpose |
|---|---|
| Register in place | Register the existing VM files without copying them |
| Restore | Import while using the existing VM identity |
| Copy | Copy VM files and create a new VM identity |

> [!WARNING]
> Do not start two copies of the same server on the same network with identical identities or IP addresses unless you have deliberately prepared an isolated test environment.

### 4.3. Create a Checkpoint

Create a checkpoint before a configuration change:

```powershell
Checkpoint-VM `
    -Name "VM01" `
    -SnapshotName "Before-Update"
```

List checkpoints:

```powershell
Get-VMCheckpoint -VMName "VM01"
```

Restore a checkpoint:

```powershell
Restore-VMCheckpoint `
    -Name "Before-Update" `
    -VMName "VM01" `
    -Confirm:$false
```

Delete a checkpoint:

```powershell
Get-VMCheckpoint -VMName "VM01" |
    Where-Object Name -eq "Before-Update" |
    Remove-VMCheckpoint
```

### 4.4. Production and Standard Checkpoints

- **Production checkpoint:** Uses supported guest mechanisms to capture a consistent state without saving the VM's running memory state.
- **Standard checkpoint:** Captures the VM's state, including memory, and is useful for certain lab scenarios.

Configure the checkpoint type:

```powershell
Set-VM `
    -Name "VM01" `
    -CheckpointType Production
```

Or, for a lab where a standard checkpoint is specifically required:

```powershell
Set-VM `
    -Name "VM01" `
    -CheckpointType Standard
```

> [!IMPORTANT]
> Checkpoints are not backups. They depend on the original virtual disks and can consume significant storage. Do not keep checkpoints indefinitely on production VMs.

---

## 5. Virtual Networking

### 5.1. Virtual Switch Types

| Switch Type | VM-to-VM | VM-to-Host | External Network |
|---|---|---|---|
| External | Yes | Usually, depending on configuration | Yes |
| Internal | Yes | Yes | No direct external connectivity |
| Private | Yes | No | No |

### 5.2. Create an External Virtual Switch

An external switch connects VMs to a physical network through a host network adapter.

List available network adapters:

```powershell
Get-NetAdapter
```

Create the switch:

```powershell
New-VMSwitch `
    -Name "ExternalSwitch" `
    -NetAdapterName "Ethernet" `
    -AllowManagementOS $true
```

Replace `Ethernet` with the actual physical adapter name.

Verify the switch:

```powershell
Get-VMSwitch
```

> [!WARNING]
> Creating or changing an external switch can interrupt host network connectivity. Perform the change through a suitable management connection and confirm the correct physical adapter first.

### 5.3. Create an Internal Switch

```powershell
New-VMSwitch `
    -Name "InternalSwitch" `
    -SwitchType Internal
```

This creates a switch for VM-to-VM and VM-to-host communication. It does not automatically provide Internet access or DHCP.

### 5.4. Create a Private Switch

```powershell
New-VMSwitch `
    -Name "PrivateSwitch" `
    -SwitchType Private
```

Use this for isolated lab environments where VMs need to communicate with each other but not directly with the host or external network.

### 5.5. Connect a VM to a Switch

```powershell
Connect-VMNetworkAdapter `
    -VMName "VM01" `
    -SwitchName "InternalSwitch"
```

Check the connection:

```powershell
Get-VMNetworkAdapter -VMName "VM01"
```

### 5.6. Configure VLAN Settings

For a VM that must use VLAN 20:

```powershell
Set-VMNetworkAdapterVlan `
    -VMName "VM01" `
    -Access `
    -VlanId 20
```

Verify the VLAN configuration:

```powershell
Get-VMNetworkAdapterVlan -VMName "VM01"
```

**Important:** VLAN configuration must match the physical switch and network design. An incorrect VLAN ID may prevent the VM from communicating with the intended network.

### 5.7. Troubleshoot Virtual Networking

View virtual switches:

```powershell
Get-VMSwitch
```

View VM network adapters:

```powershell
Get-VMNetworkAdapter -VMName "VM01"
```

View host network configuration:

```powershell
Get-NetIPConfiguration
```

Test connectivity from the host:

```powershell
Test-Connection -ComputerName "192.168.10.1" -Count 4
```

If a VM cannot communicate, check the virtual switch, VM adapter connection, VLAN settings, guest IP configuration, DNS settings, firewall rules, and physical network configuration.

---

## 6. Resource Management

### 6.1. Configure Virtual CPUs

Assign two virtual processors:

```powershell
Set-VMProcessor -VMName "VM01" -Count 2
```

View processor settings:

```powershell
Get-VMProcessor -VMName "VM01"
```

Do not assign more CPU resources than the host can reasonably support for the expected workload.

### 6.2. Configure Dynamic Memory

Enable dynamic memory:

```powershell
Set-VMMemory `
    -VMName "VM01" `
    -DynamicMemoryEnabled $true `
    -StartupBytes 4GB `
    -MinimumBytes 2GB `
    -MaximumBytes 8GB
```

Verify the configuration:

```powershell
Get-VMMemory -VMName "VM01"
```

Dynamic memory allows Hyper-V to adjust memory allocation within the configured limits. Guest operating system support and integration components affect behavior.

### 6.3. Configure VM Resource Limits

Example: limit the VM to 50% of the available virtual processor resource under Hyper-V's CPU resource controls:

```powershell
Set-VMProcessor `
    -VMName "VM01" `
    -Maximum 50
```

Set a relative CPU weight:

```powershell
Set-VMProcessor `
    -VMName "VM01" `
    -RelativeWeight 100
```

**Note:** CPU limits and relative weights serve different purposes. A maximum limit caps CPU usage, while relative weight affects how CPU resources are shared when VMs compete for them.

### 6.4. Monitor VM Resources

View VM resource usage:

```powershell
Get-VM |
    Select-Object Name, State, CPUUsage, MemoryAssigned, Uptime
```

Review memory configuration:

```powershell
Get-VM | Get-VMMemory
```

Review processor configuration:

```powershell
Get-VM | Get-VMProcessor
```

Monitor the host's overall capacity using Windows performance monitoring tools and Hyper-V performance counters.

> [!IMPORTANT]
> A VM that is running slowly may be limited by storage I/O, memory pressure, network throughput, or the guest OS—not just CPU.

---

## 7. Hyper-V Virtual Disks

### 7.1. VHD and VHDX

| Format | Description |
|---|---|
| VHD | Older virtual disk format |
| VHDX | Newer format with support for larger disks and improved resilience to certain failures |

### 7.2. Create a Fixed VHDX

```powershell
New-VHD `
    -Path "D:\Hyper-V\VHDs\Data01.vhdx" `
    -SizeBytes 100GB `
    -Fixed
```

A fixed disk allocates its configured capacity on the host storage when created.

### 7.3. Create a Dynamically Expanding VHDX

```powershell
New-VHD `
    -Path "D:\Hyper-V\VHDs\Data02.vhdx" `
    -SizeBytes 100GB `
    -Dynamic
```

A dynamically expanding disk grows as data is written, up to its configured maximum size.

> [!WARNING]
> Dynamic disks can grow until the host volume runs out of space. Monitor free storage regularly.

### 7.4. Attach a Virtual Disk to a VM

```powershell
Add-VMHardDiskDrive `
    -VMName "VM01" `
    -Path "D:\Hyper-V\VHDs\Data01.vhdx"
```

Verify the disk attachment:

```powershell
Get-VMHardDiskDrive -VMName "VM01"
```

Inside the guest operating system, initialize and format the new disk if required.

### 7.5. Expand a VHDX

For a disk that supports expansion, increase its virtual capacity:

```powershell
Resize-VHD `
    -Path "D:\Hyper-V\VHDs\Data01.vhdx" `
    -SizeBytes 150GB
```

The VM's operating system may still need to extend its partition or volume to use the additional capacity.

**Important:** A virtual disk expansion is not the same as expanding the partition inside the guest OS. Back up important data before storage changes.

### 7.6. Inspect Virtual Disks

Inspect a specific virtual disk:

```powershell
Get-VHD -Path "D:\Hyper-V\VHDs\Data01.vhdx"
```

Find disks attached to a VM:

```powershell
Get-VMHardDiskDrive -VMName "VM01"
```

### 7.7. Compact a Dynamically Expanding Disk

Disk compaction can reduce the physical size of a supported dynamically expanding VHDX when reclaimable space is available.

```powershell
Optimize-VHD `
    -Path "D:\Hyper-V\VHDs\Data02.vhdx" `
    -Mode Quick
```

> [!WARNING]
> Ensure the VM and disk are in an appropriate state before maintenance. `Optimize-VHD` requires the Hyper-V module and may require the VM to be shut down or the disk to be detached. Compaction does not guarantee that a disk will become smaller.

---

## 8. VM Versions

### 8.1. View VM Configuration Versions

List VM configuration versions:

```powershell
Get-VM |
    Select-Object Name, Version, Generation, State
```

View a specific VM:

```powershell
Get-VM -Name "VM01" |
    Select-Object Name, Version, Generation
```

The VM configuration version determines which Hyper-V features and configuration format the VM supports. It is different from VM Generation 1 or Generation 2.

### 8.2. Check the Highest Supported Version

```powershell
Get-VMHostSupportedVersion
```

This helps identify the VM configuration versions supported by the host.

### 8.3. Upgrade a VM Configuration Version

First, check the current version:

```powershell
Get-VM -Name "VM01" |
    Select-Object Name, Version, State
```

After confirming compatibility, shut down the VM and upgrade its configuration version:

```powershell
Update-VMVersion -Name "VM01"
```

Verify the result:

```powershell
Get-VM -Name "VM01" |
    Select-Object Name, Version
```

**Important warnings:**

- Back up or export the VM before upgrading its configuration version.
- Upgrading may prevent the VM from running on older Hyper-V hosts.
- A VM configuration version upgrade is generally not reversible through a simple downgrade command.
- Do not upgrade until you have confirmed the target host versions and migration requirements.

---

## 9. Hyper-V Replica

### 9.1. Overview and Prerequisites

Hyper-V Replica asynchronously copies VM changes to a second Hyper-V host. It is designed for disaster recovery, not as a replacement for regular backups or a failover cluster.

Example lab:

| Component | Example |
|---|---|
| Primary host | `HV01` |
| Replica host | `HV02` |
| Primary VM | `VM01` |
| Authentication | Kerberos or certificate-based |
| Default Kerberos port | TCP 80 |
| Default certificate-based port | TCP 443 |

Before configuring replication:

- Install Hyper-V on both hosts.
- Configure name resolution and network connectivity.
- Configure the replica server to accept replication.
- Configure Windows Firewall and any network firewalls.
- Ensure the authentication method matches the environment.
- Plan storage capacity on the replica host.

### 9.2. Enable the Replica Server

On the replica host, open Hyper-V Manager.

1. Right-click the host and select **Hyper-V Settings**.
2. Select **Replication Configuration**.
3. Enable the host as a Replica server.
4. Select the required authentication method.
5. Configure the replication authorization policy.
6. Configure the storage location for replica files.
7. Apply the settings.

For Kerberos authentication, the hosts typically need suitable domain configuration and name resolution. Certificate-based authentication requires correctly configured certificates.

### 9.3. Enable Hyper-V Replica Using PowerShell

On the replica host, configure the host to accept Kerberos-authenticated replication:

```powershell
Set-VMReplicationServer `
    -ReplicationEnabled $true `
    -AllowedAuthenticationType Kerberos `
    -KerberosAuthenticationPort 80 `
    -ReplicationAllowedFromAnyServer $true `
    -DefaultStorageLocation "D:\Hyper-V\Replica"
```

Create the storage directory first:

```powershell
New-Item `
    -ItemType Directory `
    -Path "D:\Hyper-V\Replica" `
    -Force
```

**Security note:** `-ReplicationAllowedFromAnyServer $true` allows replication authorization from any primary server, subject to the configured authentication requirements. For a controlled environment, prefer an explicit allow-list:

```powershell
Set-VMReplicationServer `
    -ReplicationEnabled $true `
    -AllowedAuthenticationType Kerberos `
    -KerberosAuthenticationPort 80 `
    -ReplicationAllowedFromAnyServer $false `
    -ReplicationAllowedPrimaryServer "HV01.contoso.com" `
    -DefaultStorageLocation "D:\Hyper-V\Replica"
```

Use the correct primary-server FQDN for your environment.

### 9.4. Configure the Firewall

On the replica host, inspect the available Hyper-V Replica firewall rules:

```powershell
Get-NetFirewallRule |
    Where-Object DisplayName -Match "Hyper-V Replica"
```

Enable the applicable built-in firewall rule group if it exists on the installed Windows Server version:

```powershell
Enable-NetFirewallRule -DisplayGroup "Hyper-V Replica HTTP Listener (TCP-In)"
```

For certificate-based replication, enable the corresponding HTTPS listener rule group where available.

> [!WARNING]
> Firewall display-group names can differ by OS language and version. Confirm the exact rule group on your server before enabling it, and restrict network access to trusted hosts.

### 9.5. Enable Replication for a VM

On the primary host, configure replication for `VM01`:

```powershell
Enable-VMReplication `
    -VMName "VM01" `
    -ReplicaServerName "HV02.contoso.com" `
    -ReplicaServerPort 80 `
    -AuthenticationType Kerberos
```

Start the initial replication:

```powershell
Start-VMInitialReplication -VMName "VM01"
```

Check replication status:

```powershell
Get-VMReplication -VMName "VM01" |
    Format-List VMName, ReplicationState, ReplicationHealth
```

### 9.6. Test Failover

A test failover allows you to test a replica without performing a production failover.

On the replica host:

```powershell
Start-VMFailover -VMName "VM01" -AsTest
```

After testing, stop the test failover:

```powershell
Stop-VMFailover -VMName "VM01"
```

Follow the Hyper-V Manager workflow to confirm and clean up the test VM if required.

### 9.7. Planned and Unplanned Failover

**Planned failover** is used when the primary VM is available and can be shut down gracefully.

**Unplanned failover** is used when the primary VM is unavailable. The selected recovery point may not include the latest changes.

Do not run failover commands on a production workload without verifying the primary and replica roles, replication state, network settings, and recovery plan. Follow the appropriate Hyper-V Manager or PowerShell workflow for the intended failover type.

> [!IMPORTANT]
> Never start the primary and replica copies of the same server on the same production network simultaneously unless you have a deliberate, safe recovery procedure.

---

## 10. Live Migration

### 10.1. Overview and Prerequisites

Live Migration moves a running VM from one Hyper-V host to another with minimal interruption. It does not automatically provide high availability; that requires additional planning or a failover cluster.

Example lab:

- Source host: `HV01.contoso.com`
- Destination host: `HV02.contoso.com`
- VM: `VM01`

Both hosts should have:

- A supported Windows Server and Hyper-V configuration.
- Network connectivity and name resolution.
- Appropriate user permissions.
- Configured migration authentication.
- Compatible virtual switch names and network settings.
- Sufficient CPU, memory, and storage capacity on the destination.

### 10.2. Enable Live Migration

On each host, open **Hyper-V Settings > Live Migrations**.

1. Enable incoming and outgoing live migrations.
2. Configure the permitted migration networks.
3. Choose the authentication protocol.
4. Confirm the firewall and permissions configuration.

PowerShell example on each host:

```powershell
Set-VMHost -VirtualMachineMigrationEnabled $true
```

Configure the authentication method:

```powershell
Set-VMHost `
    -VirtualMachineMigrationAuthenticationType Kerberos
```

Kerberos-based migration between domain hosts requires appropriate constrained delegation configuration. CredSSP is another option for certain scenarios, but typically requires the administrator to sign in to the source host to initiate the migration.

### 10.3. Migrate a Running VM

If the VM storage is accessible to the destination host, migrate the VM:

```powershell
Move-VM `
    -Name "VM01" `
    -DestinationHost "HV02.contoso.com"
```

To move the VM and its storage to a destination path:

```powershell
Move-VM `
    -Name "VM01" `
    -DestinationHost "HV02.contoso.com" `
    -IncludeStorage `
    -DestinationStoragePath "D:\Hyper-V\VMs\VM01"
```

Check the VM on the destination host:

```powershell
Get-VM -ComputerName "HV02.contoso.com" -Name "VM01"
```

### 10.4. Troubleshooting Live Migration

If migration fails, check:

- DNS resolution between hosts.
- Authentication and delegation settings.
- Firewall rules and migration network connectivity.
- VM switch names and network compatibility.
- Destination CPU compatibility.
- Destination memory and storage capacity.
- Permissions on the source and destination hosts.

For migrations between different processor generations, review the VM's processor compatibility settings and the supported migration requirements before changing production configurations.

**Important:** Live Migration without shared storage may require storage migration. Shared-storage designs and clustered VM migrations have different requirements.

---

## 11. Hyper-V Failover Clustering

### 11.1. Overview and Prerequisites

Failover Clustering allows supported workloads to move or restart on another cluster node when a node fails. Unlike Hyper-V Replica, a failover cluster is designed for high availability within a cluster architecture.

Example lab:

| Component | Example |
|---|---|
| Node 1 | `HV01` |
| Node 2 | `HV02` |
| Cluster name | `HVCLUSTER` |
| Cluster IP | `192.168.10.50` |
| Shared storage | Supported shared storage or an appropriate clustered storage design |

Before deployment:

- Use compatible Windows Server versions and updates on all nodes.
- Join the nodes to the appropriate Active Directory domain when required by the chosen design.
- Configure stable network connectivity and name resolution.
- Provide storage supported by the cluster design.
- Validate the configuration before creating the cluster.
- Plan quorum and witness configuration.
- Ensure adequate capacity remains when a node is unavailable.

> [!IMPORTANT]
> A failover cluster requires more than installing Hyper-V on two servers. The network, storage, cluster configuration, and guest workload must all be designed for high availability.

### 11.2. Install Failover Clustering

Run on both cluster nodes:

```powershell
Install-WindowsFeature `
    -Name Failover-Clustering `
    -IncludeManagementTools `
    -Restart
```

After the restart, confirm the feature:

```powershell
Get-WindowsFeature -Name Failover-Clustering
```

### 11.3. Validate the Cluster

Run from a management server or a cluster node:

```powershell
Test-Cluster -Node HV01, HV02
```

Review the validation report and resolve significant errors before proceeding.

> [!WARNING]
> Validation tests may affect storage or network resources depending on the test and environment. Perform validation during an appropriate maintenance window and follow the storage vendor's guidance.

### 11.4. Create the Cluster

After confirming that the nodes and network configuration are ready:

```powershell
New-Cluster `
    -Name "HVCLUSTER" `
    -Node HV01, HV02 `
    -StaticAddress 192.168.10.50 `
    -NoStorage
```

The example uses `-NoStorage` so storage is not automatically added during cluster creation. Configure the supported storage separately according to the cluster design.

Verify the cluster:

```powershell
Get-Cluster
Get-ClusterNode
Get-ClusterNetwork
```

### 11.5. Add a VM as a Cluster Role

After configuring supported cluster storage and placing the VM files in the correct location, add the VM to the cluster:

```powershell
Add-ClusterVirtualMachineRole -VMName "VM01"
```

Check clustered roles:

```powershell
Get-ClusterGroup
```

**Note:** The VM must meet the cluster's storage, networking, and configuration requirements. Do not assume that an ordinary VM on a standalone host automatically becomes highly available simply by creating a cluster.

### 11.6. Move a Clustered VM Between Nodes

Move a clustered VM role to another node:

```powershell
Move-ClusterVirtualMachineRole `
    -Name "VM01" `
    -Node "HV02"
```

Verify the current owner:

```powershell
Get-ClusterGroup -Name "VM01" |
    Select-Object Name, OwnerNode, State
```

### 11.7. Failover and Recovery Checks

Review node status:

```powershell
Get-ClusterNode
```

Review clustered VM roles:

```powershell
Get-ClusterGroup |
    Select-Object Name, OwnerNode, State
```

Before performing a planned node shutdown, confirm that clustered workloads can run on the remaining nodes and that sufficient resources are available.

**Important distinctions:**

- **Live Migration:** Moves a running VM between compatible hosts with minimal interruption.
- **Hyper-V Replica:** Maintains an asynchronous copy on a separate host for disaster recovery.
- **Failover Clustering:** Provides high availability through coordinated cluster nodes and supported storage/network architecture.

These technologies solve different problems and may be combined in larger environments, but they are not interchangeable.

---

## 12. Troubleshooting Commands and Best Practices

### 12.1. Essential Commands

View Hyper-V host information:

```powershell
Get-VMHost
```

List all VMs:

```powershell
Get-VM
```

List virtual switches:

```powershell
Get-VMSwitch
```

View VM network adapters:

```powershell
Get-VMNetworkAdapter -VMName "VM01"
```

View VM memory settings:

```powershell
Get-VMMemory -VMName "VM01"
```

View VM processor settings:

```powershell
Get-VMProcessor -VMName "VM01"
```

View virtual disks:

```powershell
Get-VMHardDiskDrive -VMName "VM01"
```

View VM checkpoints:

```powershell
Get-VMCheckpoint -VMName "VM01"
```

View replication status:

```powershell
Get-VMReplication
```

View Hyper-V service status:

```powershell
Get-Service -Name vmms
```

### 12.2. Common Issues

| Issue | Checks |
|---|---|
| VM does not start | Available memory, disk space, VM configuration, event logs |
| VM has no network access | Virtual switch, network adapter, VLAN, IP settings, firewall |
| VM runs slowly | CPU, memory pressure, storage latency, guest OS |
| VHDX cannot be expanded | VM/disk state, file permissions, available host storage |
| Live Migration fails | Authentication, DNS, firewall, compatibility, destination capacity |
| Replication fails | Replica authorization, ports, authentication, network, storage |
| Cluster role fails over incorrectly | Cluster validation, quorum, storage, networking, capacity |

### 12.3. Review Hyper-V Events

Use Event Viewer:

1. Open **Event Viewer**.
2. Navigate to **Applications and Services Logs > Microsoft > Windows**.
3. Review the Hyper-V logs, especially the VMMS and Worker channels.

List relevant event logs with PowerShell:

```powershell
Get-WinEvent -ListLog "Microsoft-Windows-Hyper-V-VMMS/*"
```

Retrieve recent VMMS events:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS/Admin" -MaxEvents 20 |
    Select-Object TimeCreated, Id, LevelDisplayName, Message
```

### 12.4. Best Practices

- Use descriptive names for hosts, VMs, switches, and virtual disks.
- Keep host operating systems updated.
- Use checkpoints temporarily, not as a backup strategy.
- Monitor free disk space and VM resource consumption.
- Use separate, well-planned networks for management, VM traffic, storage, and migration where appropriate.
- Restrict Hyper-V management access to authorized administrators.
- Test exports, backups, replication, and recovery procedures regularly.
- Validate a failover cluster before placing production workloads on it.
- Document VM configuration, IP addresses, switch mappings, and recovery steps.
- Test configuration changes in a lab before applying them to production systems.
