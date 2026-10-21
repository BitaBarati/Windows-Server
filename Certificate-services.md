
# Active Directory Certificate Services (AD CS)

## 1. Introduce CA Server and Cryptography

### 1.1. Overview

Active Directory Certificate Services (AD CS) is a Windows Server role used to issue and manage digital certificates for authentication, encryption, digital signatures, HTTPS, and IPsec.

A Certificate Authority (CA) validates certificate requests and issues certificates according to its configuration and certificate templates.

### 1.2. Key Components

| Component | Description |
|---|---|
| CA (Certificate Authority) | Issues and manages certificates |
| Root CA | Establishes the trust anchor for a PKI hierarchy |
| Subordinate CA | Receives its CA certificate from a parent CA |
| Certificate Template | Defines certificate settings and enrollment permissions |
| CRL | Certificate Revocation List; identifies revoked certificates |
| AIA | Specifies locations for CA certificates |
| CDP | Specifies locations for certificate revocation information |
| CSR | Certificate Signing Request |
| Private Key | Must remain protected and is used for cryptographic operations |
| Public Key | Distributed through the certificate |

### 1.3. Cryptography Essentials

- **Encryption:** Protects data confidentiality.
- **Hashing:** Produces a digest used for integrity verification.
- **Digital Signature:** Helps verify the signer and detect changes.
- **Certificate:** Binds an identity to a public key.
- **TLS:** Protects HTTPS connections using certificates and cryptographic negotiation.

**Important:** A certificate does not automatically encrypt all communication. The application or protocol must use it correctly.

---

## 2. Configure an Enterprise Root CA

### 2.1. Lab Environment

| Item | Value |
|---|---|
| Domain | `contoso.com` |
| Domain Controller / DNS | `DC01.contoso.com` |
| Root CA | `CA01.contoso.com` |
| Issuing CA | `CA02.contoso.com` |
| Web Server | `WEB01.contoso.com` |
| Example subnet | `192.168.10.0/24` |

Replace these values with those used in your lab.

### 2.2. Prerequisites

- A supported Windows Server installation.
- A static IP address and working DNS.
- Correct system time.
- Domain membership for an Enterprise CA.
- Appropriate administrative permissions.
- A tested backup and recovery plan.

**Warning:** Configure the server name, IP address, and domain membership before installing AD CS. Changing the server identity after CA installation can cause significant configuration problems.

### 2.3. Install the AD CS Role

Run PowerShell as Administrator on `CA01`:

```powershell
Install-WindowsFeature ADCS-Cert-Authority -IncludeManagementTools
```

### 2.4. Configure an Enterprise Root CA

```powershell
Install-AdcsCertificationAuthority `
    -CAType EnterpriseRootCA `
    -CACommonName "CONTOSO-ROOT-CA" `
    -KeyLength 4096 `
    -HashAlgorithmName SHA256 `
    -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
    -ValidityPeriod Years `
    -ValidityPeriodUnits 10 `
    -Force
```

**Important Notes**

- The example uses a 4096-bit RSA key and SHA-256.
- The CA validity period is an example policy, not a universal recommendation.
- Protect the CA private key and restrict administrative access.
- A dedicated CA server is preferable for production.
- A two-tier PKI can provide stronger separation of duties than a single-tier deployment.

### 2.5. GUI Installation

1. Open **Server Manager**.
2. Select **Add Roles and Features**.
3. Choose **Role-based or feature-based installation**.
4. Select the target server.
5. Enable **Active Directory Certificate Services**.
6. Add the required management tools.
7. Complete the wizard.
8. Select the notification flag and click **Configure Active Directory Certificate Services**.
9. Select the appropriate credentials.
10. Select **Certification Authority**.
11. Choose **Enterprise CA**.
12. Choose **Root CA**.
13. Create a new private key.
14. Configure the cryptographic provider, key length, and hash algorithm.
15. Enter the CA common name, such as `CONTOSO-ROOT-CA`.
16. Set the CA certificate validity period.
17. Select the database and log locations.
18. Review the configuration and click **Configure**.

### 2.6. Verify the CA

```powershell
Get-Service CertSvc

Get-WindowsFeature ADCS-Cert-Authority

certutil -ping

certutil -CAInfo
```

Open the Certification Authority console:

```powershell
certsrv.msc
```

Verify that the CA service is running and the CA appears in the console.

---

## 3. Configure Certificate Templates and Enrollment

### 3.1. Open Certificate Templates

```powershell
certtmpl.msc
```

Review the available certificate templates.

To publish a template:

1. Open `certsrv.msc`.
2. Right-click **Certificate Templates**.
3. Select **New > Certificate Template to Issue**.
4. Select the required template.
5. Click **OK**.

Publishing a template does not automatically grant every user or computer permission to enroll.

### 3.2. Duplicate and Configure a Web Server Template

1. Open `certtmpl.msc`.
2. Right-click the **Web Server** template.
3. Select **Duplicate Template**.
4. Set a name, such as `Contoso-WebServer`.
5. Under **Subject Name**, configure the subject information according to your enrollment design.
6. Ensure the website DNS names are correctly represented in the certificate's Subject Alternative Name (SAN).
7. Under **Extensions**, review the application policies and key usage.
8. Under **Security**, grant the intended computer accounts **Read** and **Enroll** permissions.
9. Save the template.
10. Publish it through the CA console.

**Security Note:** Grant enrollment permissions only to intended accounts. Avoid broad enrollment permissions on templates that allow requesters to supply identities.

### 3.3. Request a Computer Certificate

On a domain-joined Windows server:

```powershell
certlm.msc
```

1. Navigate to **Personal > Certificates**.
2. Right-click **Certificates**.
3. Select **All Tasks > Request New Certificate**.
4. Select **Active Directory Enrollment Policy**.
5. Select the published certificate template.
6. Supply required DNS names if the template permits it.
7. Complete enrollment.

### 3.4. Verify the Certificate

```powershell
Get-ChildItem Cert:\LocalMachine\My |
    Select-Object Subject, Issuer, NotBefore, NotAfter, Thumbprint, HasPrivateKey
```

Check the certificate's SAN, Enhanced Key Usage (EKU), validity, private key, and certification path before deployment.

---

## 4. Configure HTTPS Sites

### 4.1. Prerequisites

- IIS installed on `WEB01`.
- A certificate issued by a trusted CA.
- The website DNS name included in the certificate SAN.
- DNS resolving the website name to the web server.
- TCP port 443 permitted through relevant firewalls.

Example website: `www.contoso.com`.

### 4.2. Install IIS

On `WEB01`:

```powershell
Install-WindowsFeature Web-Server -IncludeManagementTools
```

Verify:

```powershell
Get-WindowsFeature Web-Server
```

### 4.3. Request a Web Server Certificate

On `WEB01`:

1. Open `certlm.msc`.
2. Navigate to **Personal > Certificates**.
3. Select **Request New Certificate**.
4. Choose the published web server template.
5. Ensure `www.contoso.com` is included in the certificate SAN.
6. Complete enrollment.

If the template does not support supplying the required DNS names, configure an approved template or use an appropriate enrollment method.

### 4.4. Configure the HTTPS Binding in IIS

1. Open **Server Manager > Tools > Internet Information Services (IIS) Manager**.
2. Expand the server and select **Sites**.
3. Select the website.
4. Click **Bindings**.
5. Click **Add**.
6. Set **Type** to `https`.
7. Set **Port** to `443`.
8. Enter the hostname, such as `www.contoso.com`, when using a hostname-based HTTPS binding.
9. Select the correct SSL certificate.
10. Enable **Require Server Name Indication (SNI)** when required for hostname-based HTTPS hosting.
11. Click **OK**.

### 4.5. Verify HTTPS

From a client that trusts the issuing CA:

```powershell
Resolve-DnsName www.contoso.com

Test-NetConnection www.contoso.com -Port 443

Invoke-WebRequest https://www.contoso.com
```

Ensure the client trusts the appropriate CA chain.

**Do not bypass certificate validation to conceal trust or name-mismatch problems.**

### 4.6. Common HTTPS Problems

| Problem | Checks |
|---|---|
| Name mismatch | Compare the requested hostname with the certificate SAN |
| Untrusted certificate | Verify the root and intermediate CA certificates |
| Expired certificate | Check `NotAfter` and renew the certificate |
| Missing private key | Confirm that enrollment created the private key |
| Connection refused | Check IIS bindings, firewall rules, and TCP 443 |
| Revocation check failure | Verify CRL/AIA reachability and revocation status |

---

## 5. Configure an Enterprise Subordinate CA

### 5.1. Overview

An Enterprise Subordinate CA receives its CA certificate from a parent CA. In a two-tier PKI, the root CA can remain offline while a subordinate issuing CA handles routine certificate issuance.

### 5.2. Prerequisites

- An operational parent CA.
- A domain-joined subordinate CA server.
- Appropriate permissions and an approved CA certificate request process.
- Reachable publication locations for CA certificates and revocation information.

### 5.3. Install the AD CS Role

On `CA02`:

```powershell
Install-WindowsFeature ADCS-Cert-Authority -IncludeManagementTools

New-Item -Path C:\CA -ItemType Directory -Force
```

### 5.4. Configure the Subordinate CA and Generate a Request

```powershell
Install-AdcsCertificationAuthority `
    -CAType EnterpriseSubordinateCA `
    -CACommonName "CONTOSO-ISSUING-CA01" `
    -KeyLength 4096 `
    -HashAlgorithmName SHA256 `
    -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
    -OutputCertRequestFile "C:\CA\CONTOSO-ISSUING-CA01.req" `
    -Force
```

This generates a certificate request. The subordinate CA cannot issue certificates until its CA certificate has been issued and installed successfully.

### 5.5. Submit the Request to the Parent CA

Transfer the request securely to the parent CA.

Submit the request using:

```cmd
certreq -submit C:\CA\CONTOSO-ISSUING-CA01.req
```

Select the intended parent CA if prompted. Depending on the parent's policy, the request may be issued immediately or remain pending approval.

If the request is pending, review it in the parent CA console and issue it only after the required approval.

### 5.6. Install the Issued Subordinate CA Certificate

After the parent CA issues the request, transfer the issued CA certificate securely to the subordinate server.

For a certificate issued as a `.cer` file:

```cmd
certreq -accept C:\CA\CONTOSO-ISSUING-CA01.cer
```

Use the actual path and filename of the issued certificate. This step associates the issued certificate with the pending request and its private key.

If the AD CS configuration wizard has not completed, finish the configuration using the supported installation workflow for the Windows Server version in use.

**Important:** Do not pass a `.cer` file to the `-CertFile` parameter of `Install-AdcsCertificationAuthority`. That parameter is intended for an appropriate PKCS #12 certificate file, such as `.p12` or `.pfx`, in workflows that use that format.

### 5.7. Verify the Subordinate CA

```powershell
Get-Service CertSvc

certutil -ping

certutil -CAInfo

certutil -verify C:\CA\CONTOSO-ISSUING-CA01.cer
```

Verify that:

- The CA service is running.
- The subordinate CA certificate is installed.
- The CA chain leads to the intended root CA.
- CRL and AIA locations are reachable by clients.
- Only approved certificate templates are published.

### 5.8. Root CA Security

For a production two-tier PKI:

- Keep the root CA offline except during controlled maintenance.
- Protect root CA private keys and maintain secure backups.
- Plan CRL publication and renewal before taking the root CA offline.
- Use the subordinate CA for routine certificate issuance.
- Test certificate chain validation and revocation checking from a client.

---

## 6. Configure IPsec with Certificates

### 6.1. Prerequisites

- Certificates suitable for IPsec authentication are installed on participating computers.
- The required authentication EKU and certificate trust chain are present.
- DNS, time synchronization, and certificate revocation checking work.
- Windows Defender Firewall rules are configured for the intended IPsec scenario.

**Important:** Installing AD CS alone does not enable IPsec. An IPsec policy must be configured separately.

### 6.2. Enroll Computer Certificates

1. Publish an appropriate computer certificate template on the issuing CA.
2. Grant the target computer accounts the required Read and Enroll permissions.
3. Enroll certificates on the participating computers.
4. Verify each certificate's EKU, SAN, validity, and private key.
5. Confirm that the computers trust the issuing CA chain.

Check the local machine certificate store:

```powershell
Get-ChildItem Cert:\LocalMachine\My |
    Select-Object Subject, Issuer, NotAfter, Thumbprint, HasPrivateKey
```

### 6.3. Create a Connection Security Rule

On a test computer:

1. Open **Windows Defender Firewall with Advanced Security**.
2. Select **Connection Security Rules**.
3. Click **New Rule**.
4. Choose the rule type appropriate to the scenario, such as **Isolation** or **Server-to-server**.
5. Specify the endpoint computers or address ranges.
6. Select the required authentication method.
7. Choose certificate-based authentication and the appropriate trusted CA.
8. Configure protocol and port restrictions where applicable.
9. Name the rule and complete the wizard.
10. Create a compatible policy on the peer computer or deploy the policy through Group Policy.

The available options depend on the Windows version and selected rule type.

### 6.4. Configure IPsec Through Group Policy

For a domain environment:

1. Open Group Policy Management:

   ```powershell
   gpmc.msc
   ```

2. Create or edit a GPO linked to the OU containing the target computers.
3. Configure the required connection security rules under Windows Defender Firewall with Advanced Security.
4. Ensure authentication settings are compatible on both endpoints.
5. Apply the GPO and verify the resulting policy.

### 6.5. Verify IPsec

```powershell
gpupdate /force

Get-NetIPsecRule

Get-NetIPsecMainModeSA

Get-NetIPsecQuickModeSA
```

Check the Security Associations (SAs) while the relevant endpoints are communicating.

**Warning:** Test IPsec rules on a small group of computers first. Incorrect authentication or traffic requirements can interrupt connectivity. Keep a recovery path available before enforcing isolation or requiring authentication.

---

## 7. Enterprise CA Management and Maintenance

### 7.1. Manage Certificate Requests

Open the Certification Authority console:

```powershell
certsrv.msc
```

Review these folders:

- **Pending Requests:** Requests awaiting disposition.
- **Issued Certificates:** Certificates issued by the CA.
- **Revoked Certificates:** Certificates revoked by the CA.
- **Failed Requests:** Requests that could not be processed.

Approve, deny, or issue requests according to the organization's enrollment policy.

### 7.2. Renew a Certificate

For a certificate nearing expiration:

1. Identify the certificate and its purpose.
2. Check the template and renewal requirements.
3. Request a renewed certificate using the supported enrollment method.
4. Install the renewed certificate.
5. Update IIS bindings or other dependent services if necessary.
6. Verify the new certificate and confirm that clients can still connect.

Renewing the CA certificate does not automatically renew certificates previously issued by that CA.

### 7.3. Revoke a Certificate

1. Open `certsrv.msc`.
2. Select **Issued Certificates**.
3. Locate the certificate.
4. Right-click it and select **All Tasks > Revoke Certificate**.
5. Choose the appropriate revocation reason.
6. Publish an updated CRL when required.
7. Verify that clients can retrieve the new revocation information.

Publish the CRL when appropriate:

```powershell
certutil -crl
```

### 7.4. Back Up the CA

Run on the CA server from an elevated command prompt:

```cmd
mkdir C:\CABackup

certutil -backupdb C:\CABackup

certutil -backupkey C:\CABackup
```

The private-key backup operation requires appropriate credentials and protection for the backup password.

**Security Requirements**

- Store backups on protected storage, not only on the CA's local disk.
- Restrict access to private-key backups.
- Protect backup passwords separately.
- Document and test CA recovery procedures.
- Back up CA configuration and registry settings as part of a complete recovery plan.

### 7.5. Troubleshooting Commands

```powershell
# Check the CA service
Get-Service CertSvc

# View certificates in the local computer store
Get-ChildItem Cert:\LocalMachine\My

# Inspect the certificate chain and revocation status
certutil -verify C:\Path\certificate.cer

# Display CA configuration information
certutil -CAInfo

# Check Group Policy application
gpresult /r
```

### 7.6. Important Security Notes

- Use least privilege for CA administration and certificate enrollment.
- Restrict who can modify certificate templates and CA configuration.
- Never distribute a CA private key to ordinary clients.
- Do not issue certificates with unnecessary EKUs or overly broad enrollment permissions.
- Publish and maintain CRL and AIA locations that clients can reach.
- Use secure protocols and supported cryptographic algorithms.
- Test certificate renewal, revocation, and recovery before production deployment.

---
