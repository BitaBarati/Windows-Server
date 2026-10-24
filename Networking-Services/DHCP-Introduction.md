# DHCP — Installation & Configuration

## 01. Introduction to DHCP

### 1. What Is DHCP?

Dynamic Host Configuration Protocol (DHCP) automatically assigns IP configuration to client devices.

DHCP provides:

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server
- DNS Domain Name

### 2. How DHCP Works

DHCP uses the DORA process:

1. Discover: Client searches for a DHCP server.
2. Offer: Server offers an IP address.
3. Request: Client requests the offered address.
4. Acknowledgment: Server confirms the lease.

### 3. DHCP Ports

| Protocol | Port | Purpose |
|---|---|---|
| UDP | 67 | DHCP Server |
| UDP | 68 | DHCP Client |

### 4. Verify Client Configuration

```cmd
ipconfig /all
ipconfig /release
ipconfig /renew
```

- `ipconfig /all`: Displays detailed network configuration.
- `ipconfig /release`: Releases the current DHCP lease.
- `ipconfig /renew`: Requests a new lease.

---

## 02. Install DHCP and Manage Scopes

### 1. Configure a Static IP Address

Before configuring DHCP, assign the server a static IP address.

**Example:**

| Setting | Value |
|---|---|
| Server Name | DC01 |
| IP Address | 192.168.1.10 |
| Subnet Mask | 255.255.255.0 |
| Gateway | 192.168.1.1 |
| DNS Server | 192.168.1.10 |

These are example values for a lab environment.

### 2. Install DHCP Server

**GUI:**

Server Manager → Manage → Add Roles and Features → DHCP Server

Select the role, install it, and complete the post-installation configuration.

**PowerShell:**

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
```

Installs the DHCP Server role and management tools.

### 3. Authorize the DHCP Server

In an Active Directory domain, authorize the DHCP server using the DHCP post-installation wizard or the appropriate administrative tools.

```powershell
Get-DhcpServerInDC
```

Lists DHCP servers authorized in Active Directory.

### 4. Create an IPv4 Scope

A Scope defines the IP address range available for DHCP clients.

**GUI:**

DHCP → IPv4 → New Scope

Configure the scope name, IP range, subnet mask, exclusions, lease duration, and scope options. Activate the scope when configuration is complete.

**Example:**

| Setting | Value |
|---|---|
| Scope Name | Office-LAN |
| Start IP | 192.168.1.100 |
| End IP | 192.168.1.200 |
| Subnet Mask | 255.255.255.0 |

**PowerShell:**

```powershell
Add-DhcpServerv4Scope `
  -Name "Office-LAN" `
  -StartRange 192.168.1.100 `
  -EndRange 192.168.1.200 `
  -SubnetMask 255.255.255.0 `
  -State Active
```

Creates and activates the scope.

```powershell
Get-DhcpServerv4Scope
```

Displays configured scopes.

### 5. Configure an Exclusion Range

Exclusions prevent DHCP from dynamically assigning specific addresses within a scope.

**GUI:**

IPv4 → Scope → Address Pool → New Exclusion Range

**PowerShell:**

```powershell
Add-DhcpServerv4ExclusionRange `
  -ScopeId 192.168.1.0 `
  -StartRange 192.168.1.1 `
  -EndRange 192.168.1.20
```

Excludes the specified addresses from dynamic allocation.

### 6. Create a DHCP Reservation

A Reservation associates an IP address with a client’s identifier, allowing the client to receive the reserved address through DHCP.

**GUI:**

IPv4 → Scope → Reservations → New Reservation

**PowerShell:**

```powershell
Add-DhcpServerv4Reservation `
  -ScopeId 192.168.1.0 `
  -IPAddress 192.168.1.150 `
  -ClientId "AA-BB-CC-DD-EE-FF" `
  -Description "Office-PC"
```

Replace the example client identifier with the client’s actual identifier.

### 7. View Address Leases

A lease represents an IP address currently assigned to a client.

**GUI:**

IPv4 → Scope → Address Leases

**PowerShell:**

```powershell
Get-DhcpServerv4Lease -ScopeId 192.168.1.0
```

Lists leases for the specified scope.

---

## 03. DHCP Options

### 1. Common DHCP Options

| Option | Name | Purpose |
|---|---|---|
| 003 | Router | Default gateway |
| 006 | DNS Servers | DNS server addresses |
| 015 | DNS Domain Name | DNS suffix |
| 044 | WINS/NBNS Servers | WINS server addresses |
| 046 | WINS/NBT Node Type | NetBIOS node type |
| 051 | Lease Time | Lease duration |
| 066 | TFTP Server Name | TFTP server for applicable boot scenarios |
| 067 | Bootfile Name | Boot file for applicable boot scenarios |

Options 044, 046, 066, and 067 are only needed in specific environments.

### 2. Configure Scope Options

**GUI:**

IPv4 → Scope → Scope Options → Configure Options

**Common settings:**

- 003 Router: 192.168.1.1
- 006 DNS Servers: 192.168.1.10
- 015 DNS Domain Name: example.local

Use values appropriate for your network.

### 3. Configure Server Options

**GUI:**

IPv4 → Server Options → Configure Options

Server options apply broadly to scopes unless a more specific applicable configuration overrides them.

### 4. View and Configure Options Using PowerShell

**View server-level options:**

```powershell
Get-DhcpServerv4OptionValue
```

**View scope options:**

```powershell
Get-DhcpServerv4OptionValue -ScopeId 192.168.1.0
```

**Set the default gateway:**

```powershell
Set-DhcpServerv4OptionValue `
  -ScopeId 192.168.1.0 `
  -Router 192.168.1.1
```

**Set DNS server and domain:**

```powershell
Set-DhcpServerv4OptionValue `
  -ScopeId 192.168.1.0 `
  -DnsServer 192.168.1.10 `
  -DnsDomain "example.local"
```

---

## 04. DHCP Advanced Configuration & Management

### 1. DHCP Console Structure

| Node | Purpose |
|---|---|
| Server Options | Server-wide options |
| Policies | Conditional configuration |
| Filters | Allow or deny clients by identifier |
| Address Pool | Scope range and exclusions |
| Address Leases | Assigned IP addresses |
| Reservations | Reserved client addresses |
| Scope Options | Scope-specific settings |

### 2. Server and Scope Properties

**Server Properties:**

DHCP → Right-click Server → Properties

**Scope Properties:**

DHCP → IPv4 → Scope → Right-click Scope → Properties

Review the available settings in your Windows Server version before making changes.

### 3. Activate or Deactivate a Scope

**GUI:**

IPv4 → Scope → Right-click Scope → Activate / Deactivate

A scope must be active to issue new leases.

**Verify scope status:**

```powershell
Get-DhcpServerv4Scope
```

### 4. Backup and Restore DHCP

**Backup GUI:**

DHCP → Right-click Server → Backup

**Restore GUI:**

DHCP → Right-click Server → Restore

**PowerShell backup:**

```powershell
Backup-DhcpServer -Path "C:\DHCP-Backup"
```

**PowerShell restore:**

```powershell
Restore-DhcpServer -Path "C:\DHCP-Backup"
```

Ensure the backup directory exists and is accessible. Restore operations can replace existing DHCP configuration, so use them carefully.

### 5. Check DHCP Service

```powershell
Get-Service DHCPServer
```

Displays the DHCP Server service status.

### 6. Troubleshooting

**Check the client:**

```cmd
ipconfig /all
ipconfig /release
ipconfig /renew
```

**Check the server:**

```powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 192.168.1.0
Get-Service DHCPServer
```

**Common issues:**

- DHCP service is stopped.
- Scope is inactive or the address pool is exhausted.
- DHCP authorization is missing in an Active Directory environment.
- Gateway or DNS options are incorrect.
- The client cannot reach the DHCP server.
- DHCP Relay Agent configuration is missing when clients and servers are on different subnets.

---

## 05. DHCP Failover

### 1. What Is DHCP Failover?

DHCP Failover allows two DHCP servers to share lease information for supported IPv4 scopes to improve availability.

**Modes:**

- Load Balance: Both servers serve clients.
- Hot Standby: One server primarily serves clients while the other acts as standby.

### 2. Configure Failover

**GUI:**

DHCP → IPv4 → Configure Failover

1. Select the scopes.
2. Add the partner DHCP server.
3. Select the failover mode.
4. Configure relationship settings and the shared secret if required.
5. Complete the wizard.

Ensure both servers have the required permissions, network connectivity, and authorization in Active Directory when applicable.

### 3. Verify Failover

```powershell
Get-DhcpServerv4Failover
```

Lists failover relationships on the local server.

```powershell
Get-DhcpServerv4Failover -ComputerName "DC01"
```

Lists relationships on the specified server.

Verify the relationship state and test client connectivity in a controlled environment.

---

## 06. IP Address Management (IPAM)

### 1. What Is IPAM?

IP Address Management (IPAM) helps centrally discover, monitor, and manage supported IP address infrastructure, including DHCP and DNS servers.

### 2. Install and Configure IPAM

**GUI:**

Server Manager → Manage → Add Roles and Features

Select the IPAM feature if supported by your Windows Server version.

**PowerShell:**

```powershell
Get-WindowsFeature IPAM*
```

Lists IPAM-related features.

```powershell
Install-WindowsFeature IPAM -IncludeManagementTools
```

Installs IPAM on versions that support this feature name.

### 3. Configure IPAM

**Open:**

Server Manager → IPAM

Follow the configuration wizard, configure provisioning and permissions, discover the required servers, and verify that they are manageable.

---

## 07. WSUS Scenarios

### 1. What Is WSUS?

Windows Server Update Services (WSUS) allows organizations to manage and distribute updates to supported Windows devices. WSUS is a separate service from DHCP.

### 2. Install WSUS

**GUI:**

Server Manager → Manage → Add Roles and Features → Windows Server Update Services

Install the role, configure the update content directory, and complete the required post-installation tasks.

### 3. Configure WSUS

**In the WSUS console:**

1. Configure the update source.
2. Select products and classifications.
3. Configure synchronization.
4. Synchronize updates.
5. Review and approve updates for the required computer groups.

### 4. Configure Clients Using Group Policy

Open Group Policy Management and navigate to:

Computer Configuration → Policies → Administrative Templates → Windows Components → Windows Update

Configure the applicable intranet update service policy with the WSUS server URL.

**Example:**

```text
http://WSUS01:8530
```

Use the actual server name, port, and security configuration for your environment.

### 5. Check WSUS Services

```powershell
Get-Service WsusService, W3SVC
```

Checks the WSUS and IIS services when installed.
