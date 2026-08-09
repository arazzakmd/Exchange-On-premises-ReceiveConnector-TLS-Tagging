# 🚀 Tagging Exchange On-Premises Receive Connectors with M365 Exchange Online Protection (EOP)

This guide provides PowerShell commands to verify and tag TLS certificates on an Exchange On-Premises server receive connector. This is essential for setting up secure TLS mail flow with **Microsoft 365 Exchange Online Protection (EOP)** during Hybrid Configuration.

---

## 📌 Context & Purpose

In an **Exchange Hybrid Deployment**, Microsoft 365 EOP validates your On-Premises Exchange Server identity using the Subject Name or Issuer Name of your TLS Certificate.

To ensure secure and successful mail flow between On-Premises and M365, you must:
1. Locate the valid third-party TLS certificate installed on your Exchange Server.
2. Verify the certificate bound to the SMTP service (Port 25).
3. Assign/Tag the formatted `TlsCertificateName` string to the **Receive Connector** (and Send Connector).

---

## 🛠️ PowerShell Commands & Verification Steps

Run the following commands in the **Exchange Management Shell (EMS)** on your On-Premises Exchange Server:

### 1️⃣ List Installed Exchange Certificates
Find the valid third-party certificate (e.g., DigiCert, Sectigo, GoDaddy) enabled for the SMTP service.

```powershell
Get-ExchangeCertificate
```
> **Note:** Copy the `Thumbprint` of the valid certificate that has `Services: IP.....S..` (SMTP enabled) and is not expired.

---

### 2️⃣ Check Certificate Details Bound to Port 25
Check the specific TLS certificate properties listening on Port 25 for your server.

```powershell
Get-SMTPCertificate -Server EXMB-01 -Port 25 | Format-List *
```
> 💡 *Note:* `Get-SMTPCertificate` is a custom PowerShell helper function commonly used by Exchange admins to inspect TLS handshake certificates on specific ports.

---

### 3️⃣ Check `TlsCertificateName` on the Receive Connector
Verify which TLS certificate is currently assigned to your Frontend Receive Connector.

```powershell
Get-ReceiveConnector "EX1\Default Frontend EX1" | Format-List TlsCertificateName
```
> **Why this matters:** For M365 EOP to send mail back to On-Premises securely, this string must match what is configured in your Office 365 Inbound Connector.

---

## 💡 How to Set/Tag `TlsCertificateName` (If Missing or Empty)

If `TlsCertificateName` returns blank or needs to be updated, execute the following script in EMS:

```powershell
# Step A: Get the certificate using its Thumbprint
$TLSCert = Get-ExchangeCertificate -Thumbprint "YOUR_CERTIFICATE_THUMBPRINT"

# Step B: Format the certificate attribute string (<I>Issuer<S>Subject)
$TLSCertName = "<I>$($TLSCert.Issuer)<S>$($TLSCert.Subject)"

# Step C: Assign to the Receive Connector
Set-ReceiveConnector "EX1\Default Frontend EX1" -TlsCertificateName $TLSCertName
```


### 4️⃣ Verify the Change
Confirm that the `TlsCertificateName` attribute has been updated successfully on the Send Connector:

```powershell
Get-ReceiveConnector "EX1\Default Frontend EX1" | Format-List Name, TlsCertificateName
```
---


