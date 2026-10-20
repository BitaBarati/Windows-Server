
# 01 - DNS: NetBIOS and DNS Name Resolution

## 1.1. Lab Environment

The following examples use a standalone Windows Server DNS deployment. The configuration is suitable for lab environments and can be adapted to Windows Server 2016, 2019, 2022, and 2025.

| Component | Example Value |
|---|---|
| DNS server hostname | `DNS01` |
| DNS server IPv4 address | `192.168.10.10` |
| Subnet mask | `255.255.255.0` |
| Default gateway | `192.168.10.1` |
| DNS domain | `contoso.com` |
| Client hostname | `CLIENT01` |
| Client IPv4 address | `192.168.10.20` |

Replace these example values with addresses and names appropriate for your lab.

> [!IMPORTANT]
> This section configures DNS for a standalone DNS namespace. If the server is also an Active Directory Domain Services (AD DS) domain controller, use an appropriate Active Directory-integrated DNS zone for the AD domain instead of creating an unrelated standalone zone.

## 1.2. Concept

Name resolution translates hostnames into IP addresses or identifies network services.

Windows supports several name resolution mechanisms:

- **DNS (Domain Name System):** The primary name resolution system for modern Windows networks and Active Directory.
- **NetBIOS Name Resolution:** A legacy mechanism used by older Windows networking applications.
- **LLMNR (Link-Local Multicast Name Resolution):** Resolves names on the local network link when applicable DNS resolution is unavailable.
- **Hosts File:** Provides static hostname-to-IP mappings on an individual computer.

DNS is essential for Active Directory domain controller discovery and service location, including the resolution of SRV records.

## 1.3. DNS vs. NetBIOS

| Feature | DNS | NetBIOS |
|---|---|---|
| Name structure | Hierarchical namespace | Flat naming system |
| Example | `dc01.contoso.com` | `DC01` |
| Resolution methods | DNS queries to configured servers | Broadcasts, WINS, and local NetBIOS name tables |
| IP support | IPv4 and IPv6 | Traditional NetBIOS over TCP/IP uses IPv4 |
| Active Directory | Essential for AD DS DNS-based discovery | Primarily for legacy compatibility |
| Modern usage | Recommended | Use only when required |

### 1.3.1. DNS Names

A fully qualified domain name (FQDN) identifies a host within the DNS namespace.

Example:

- **Hostname:** `DNS01`
- **DNS domain:** `contoso.com`
- **FQDN:** `DNS01.contoso.com`
- **IPv4 address:** `192.168.10.10`

Common DNS record types:

| Record Type | Purpose |
|---|---|
| `A` | Maps a hostname to an IPv4 address |
| `AAAA` | Maps a hostname to an IPv6 address |
| `CNAME` | Maps an alias to another DNS name |
| `PTR` | Maps an IP address to a DNS name through reverse lookup |
| `SRV` | Identifies a host and port providing a specific service |
| `SOA` | Stores authoritative information about a DNS zone |
| `NS` | Identifies authoritative name servers for a DNS zone |

### 1.3.2. NetBIOS Names

NetBIOS uses a flat naming system traditionally associated with legacy Windows networking.

A NetBIOS name occupies 16 bytes, including a one-byte suffix that identifies the name type. The visible computer-name portion is limited to 15 characters.

NetBIOS name resolution may use:

- **Broadcasts:** Name queries sent to devices on the local network segment.
- **WINS (Windows Internet Name Service):** A centralized service for resolving NetBIOS names to IPv4 addresses.
- **LMHOSTS:** A local file containing static NetBIOS name mappings.

WINS is a legacy technology and should generally not be introduced into new deployments.

## 1.4. Configure a Static IP Address

A DNS server should have a stable IP address so that clients can consistently reach it.

### GUI Configuration

1. Open **Server Manager**.
2. Select **Local Server**.
3. Click the Ethernet connection.
4. Open **Network Connections**.
5. Right-click the appropriate network adapter and select **Properties**.
6. Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
7. Select **Use the following IP address**.
8. Enter the following example values:
   - IP address: `192.168.10.10`
   - Subnet mask: `255.255.255.0`
   - Default gateway: `192.168.10.1`
   - Preferred DNS server: `192.168.10.10`
9. Click **OK** and close the network properties windows.

For a standalone DNS server, configure its preferred DNS server according to the intended resolution architecture. In this lab, the server will resolve its own DNS zone locally.

### PowerShell Verification

Display the current IP configuration:

```powershell
Get-NetIPConfiguration
```

Display IPv4 addresses:

```powershell
Get-NetIPAddress -AddressFamily IPv4 |
    Where-Object { $_.IPAddress -notlike "127.*" } |
    Select-Object InterfaceAlias, IPAddress, PrefixLength
```

Check the configured DNS servers:

```powershell
Get-DnsClientServerAddress -AddressFamily IPv4
```

> [!WARNING]
> Do not blindly apply the example IP address to a production server. Ensure the address is available and consistent with the network's IP addressing plan.

## 1.5. Install the DNS Server Role

### 1.5.1. Install Using Server Manager

1. Open **Server Manager**.
2. Select **Manage > Add Roles and Features**.
3. Click **Next** on the Before You Begin page.
4. Select **Role-based or feature-based installation**.
5. Select the destination server.
6. On the Server Roles page, select **DNS Server**.
7. Click **Add Features** when prompted.
8. Continue through the wizard.
9. Click **Install** on the Confirmation page.
10. Wait for the installation to complete.

A restart is generally not required to install the DNS Server role.

### 1.5.2. Install Using PowerShell

Run PowerShell as Administrator:

```powershell
Install-WindowsFeature -Name DNS -IncludeManagementTools
```

Verify the installation:

```powershell
Get-WindowsFeature -Name DNS
```

Check the DNS Server service:

```powershell
Get-Service -Name DNS
```

If the service is stopped, start it:

```powershell
Start-Service -Name DNS
```

Confirm its status and startup type:

```powershell
Get-CimInstance -ClassName Win32_Service -Filter "Name='DNS'" |
    Select-Object Name, State, StartMode
```

## 1.6. Create a Forward Lookup Zone

A forward lookup zone contains DNS records used to resolve names to IP addresses and other resource information.

### 1.6.1. Create a Zone Using DNS Manager

1. Open **Server Manager > Tools > DNS**.
2. Expand the DNS server.
3. Right-click **Forward Lookup Zones**.
4. Select **New Zone**.
5. Click **Next**.
6. Select **Primary zone**.
7. If the server is not a domain controller, the option to store the zone in Active Directory will not be available.
8. Enter the zone name:

   `contoso.com`

9. Accept the default zone file name.
10. Choose the dynamic update setting appropriate for the environment.
11. Click **Finish**.

For this standalone lab, select **Do not allow dynamic updates** and create records manually. This prevents clients from dynamically modifying the zone.

### 1.6.2. Create a Zone Using PowerShell

Create a file-backed primary zone:

```powershell
Add-DnsServerPrimaryZone `
    -Name "contoso.com" `
    -ZoneFile "contoso.com.dns"
```

Verify the zone:

```powershell
Get-DnsServerZone -Name "contoso.com"
```

**Active Directory note:** For an AD DS domain, use an AD-integrated zone with an appropriate replication scope and secure dynamic updates. The standalone zone shown here is not a replacement for the DNS configuration required by Active Directory.

## 1.7. Create DNS Records

### 1.7.1. Create an A Record Using DNS Manager

1. Open **DNS Manager**.
2. Expand the DNS server.
3. Expand **Forward Lookup Zones**.
4. Select `contoso.com`.
5. Right-click the zone and select **New Host (A or AAAA)**.
6. Enter `DNS01` in the Name field.
7. Enter `192.168.10.10` in the IP address field.
8. Optionally select **Create associated pointer (PTR) record** if a suitable reverse lookup zone already exists.
9. Click **Add Host**.

The resulting FQDN will be:

`DNS01.contoso.com`

### 1.7.2. Create an A Record Using PowerShell

Create an A record for the DNS server:

```powershell
Add-DnsServerResourceRecordA `
    -ZoneName "contoso.com" `
    -Name "DNS01" `
    -IPv4Address "192.168.10.10"
```

Create a record for the client:

```powershell
Add-DnsServerResourceRecordA `
    -ZoneName "contoso.com" `
    -Name "CLIENT01" `
    -IPv4Address "192.168.10.20"
```

Verify the A records:

```powershell
Get-DnsServerResourceRecord -ZoneName "contoso.com" |
    Where-Object { $_.RecordType -eq "A" }
```

> [!IMPORTANT]
> Create records only for hosts that actually use the specified addresses. A manually created DNS record does not configure the host's network adapter or guarantee that the host is reachable.

## 1.8. Configure DNS Forwarders

A DNS forwarder sends queries that the DNS server cannot resolve locally to another DNS server.

For example, a standalone DNS server hosting `contoso.com` may forward external queries to an approved upstream DNS resolver.

### GUI Configuration

1. Open **DNS Manager**.
2. Right-click the DNS server and select **Properties**.
3. Open the **Forwarders** tab.
4. Click **Edit**.
5. Enter the IP address of the approved upstream DNS server.
6. Click **OK**, then **Apply**.

### PowerShell Configuration

Replace the example address with an approved, reachable DNS resolver:

```powershell
Set-DnsServerForwarder -IPAddress "192.168.10.1"
```

Verify the configuration:

```powershell
Get-DnsServerForwarder
```

> [!NOTE]
> The gateway address in this lab is only an example. A gateway does not necessarily provide DNS forwarding. Use an actual DNS resolver address supplied by the network administrator or lab design.

DNS forwarders are not required to resolve records stored in a server's own authoritative zones. Root hints can be used for external resolution when appropriate and when direct outbound DNS traffic is permitted.

## 1.9. Configure the Client to Use the DNS Server

The client must query the intended DNS server to resolve names in `contoso.com`.

### GUI Configuration

1. Open **Network Connections**.
2. Right-click the client network adapter.
3. Select **Properties**.
4. Open **Internet Protocol Version 4 (TCP/IPv4)**.
5. Select **Use the following DNS server addresses**.
6. Enter the following value:
   - Preferred DNS server: `192.168.10.10`
7. Click **OK**.

Keep the client's IP address and subnet configuration consistent with the lab network.

### PowerShell Configuration

First, identify the adapter name:

```powershell
Get-NetAdapter
```

Configure the DNS server on the appropriate adapter:

```powershell
Set-DnsClientServerAddress `
    -InterfaceAlias "Ethernet" `
    -ServerAddresses "192.168.10.10"
```

Replace `Ethernet` with the actual adapter alias.

Verify the configuration:

```powershell
Get-DnsClientServerAddress `
    -InterfaceAlias "Ethernet" `
    -AddressFamily IPv4
```

**Active Directory note:** Domain-joined clients should use DNS servers that can resolve the AD domain and its service records. Do not configure public DNS servers directly on domain clients as an alternative to the internal AD DNS infrastructure.

## 1.10. Verify DNS Name Resolution

### 1.10.1. Test the DNS Server Directly

From the client, query the DNS server for the record:

```powershell
Resolve-DnsName `
    -Name "DNS01.contoso.com" `
    -Type A `
    -Server "192.168.10.10"
```

Expected result:

```text
Name:       DNS01.contoso.com
Type:       A
IPAddress:  192.168.10.10
```

The formatting may differ between Windows versions.

### 1.10.2. Test Normal Client Resolution

Resolve the DNS server's FQDN using the client's configured DNS server:

```powershell
Resolve-DnsName -Name "DNS01.contoso.com"
```

Test the client record:

```powershell
Resolve-DnsName -Name "CLIENT01.contoso.com"
```

Query the zone's SOA record:

```powershell
Resolve-DnsName `
    -Name "contoso.com" `
    -Type SOA `
    -Server "192.168.10.10"
```

The SOA record identifies key information about the DNS zone.

### 1.10.3. Test With NSLookup

Query the DNS server directly:

```cmd
nslookup DNS01.contoso.com 192.168.10.10
```

Test the DNS server's response to an unknown name:

```cmd
nslookup nonexistent.contoso.com 192.168.10.10
```

The second query should normally return a name-not-found response if the record does not exist and no applicable wildcard or other DNS behavior changes the result.

### 1.10.4. Test Network Connectivity

Test TCP connectivity to port 53:

```powershell
Test-NetConnection -ComputerName "192.168.10.10" -Port 53
```

This tests TCP port 53 only. DNS queries commonly use UDP port 53, so a successful TCP test does not prove that UDP queries work.

Test basic IP connectivity separately:

```powershell
Test-Connection -ComputerName "192.168.10.10" -Count 4
```

A failed ping alone does not prove that DNS is unavailable.

## 1.11. Troubleshooting

| Problem | Diagnostic Command | Recommended Action |
|---|---|---|
| DNS role is not installed | `Get-WindowsFeature -Name DNS` | Install the DNS Server role |
| DNS service is stopped | `Get-Service -Name DNS` | Start the service and inspect its status |
| Hostname does not resolve | `Resolve-DnsName DNS01.contoso.com` | Check the record and configured DNS server |
| Direct DNS query fails | `Resolve-DnsName DNS01.contoso.com -Server 192.168.10.10` | Check the DNS service, firewall, routing, and zone |
| Client uses the wrong DNS server | `ipconfig /all` | Correct the adapter's DNS configuration |
| Client has stale cached results | `ipconfig /displaydns` | Clear the cache with `ipconfig /flushdns` |
| External names fail to resolve | `Resolve-DnsName www.example.com -Server 192.168.10.10` | Check forwarders, root hints, and outbound DNS connectivity |
| AD domain controller cannot be located | `nltest /dsgetdc:contoso.com` | Verify AD DNS records and client DNS settings |

### Recommended Troubleshooting Sequence

1. Verify the server's IP configuration.
2. Confirm that the DNS Server role is installed and the service is running.
3. Confirm that the required zone exists.
4. Verify the relevant DNS records.
5. Query the server directly using `Resolve-DnsName -Server`.
6. Check the client's DNS server configuration.
7. Check network and firewall connectivity.
8. Investigate caching only after verifying the authoritative DNS data.

## 1.12. Important Notes

- Use a static IP address for servers that provide DNS services.
- DNS records do not configure IP addresses on the actual hosts.
- A forward lookup zone and a reverse lookup zone serve different purposes.
- A DNS forwarder is used for queries the DNS server cannot resolve locally; it is not a substitute for creating correct internal records.
- NetBIOS over TCP/IP is separate from DNS and should be enabled only when required.
- In an AD DS environment, use the DNS configuration and zone integration appropriate for the Active Directory forest and domain.
- Avoid enabling nonsecure dynamic updates in production environments without a specific, justified requirement.
- Ensure that firewall rules permit DNS traffic over UDP port 53 and TCP port 53 as required by the environment.
