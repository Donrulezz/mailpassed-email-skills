---
name: m365-smtp-auth
description: Microsoft 365 / Exchange Online SMTP AUTH Basic authentication is turned off by default at the end of December 2026. Find every printer, scanner, application and script that sends through smtp.office365.com with a password, and move each one to OAuth, an SMTP relay connector, Direct Send or High Volume Email, with device-specific steps for HP, Canon, Ricoh, Xerox, Konica Minolta, Brother, Kyocera, Sharp, Toshiba, FUJIFILM, Lexmark, Epson, Synology, QNAP, WordPress SMTP plugins, SQL Server Database Mail, Veeam, Zabbix, PRTG, Jenkins, Sage, QuickBooks, IIS and PowerShell Send-MailMessage. Use for 535 5.7.139, 550 5.7.30, scan-to-email or app email failing with Microsoft 365, or planning the SMTP AUTH migration.
---

# Microsoft 365 SMTP AUTH migration

## The change (Microsoft, confirmed)

- End of December 2026: Basic authentication (username and password) for SMTP AUTH is turned off by default in existing
  Exchange Online tenants. Admins can still turn it back on for now.
- Tenants created after December 2026 get no Basic auth for SMTP AUTH; OAuth is the supported method.
- Microsoft says it will announce the final removal date in the second half of 2027.
- Outlook, Outlook on the web and Outlook mobile don't use SMTP AUTH, so people's normal mail keeps working. What breaks
  is anything that logs in to `smtp.office365.com` (port 587) with a password.
- Errors: `535 5.7.139 Authentication unsuccessful, basic authentication is disabled.` and, once Basic auth is removed for
  good, `550 5.7.30 Basic authentication is not supported for Client Submission.`

## 1. Inventory: who still sends with a password

The free script **Find-SmtpAuthSenders** (PowerShell Gallery; source and tests at
github.com/Donrulezz/Find-SmtpAuthSenders) is read-only: it changes nothing in the tenant and writes a CSV and an HTML
report with a verdict per account. The admin installs and runs it in their own PowerShell window and signs in with their
own account. Give them the commands; don't run them yourself.

```powershell
Install-Module ExchangeOnlineManagement -Scope CurrentUser
Install-Script Find-SmtpAuthSenders -Scope CurrentUser
Find-SmtpAuthSenders.ps1                                     # settings only: who may use SMTP AUTH
Find-SmtpAuthSenders.ps1 -IncludeUsageReport -IncludeSignInLogs -AdminCenterCsv .\smtpauth.csv
```

- `-IncludeUsageReport`: 180 days of the Microsoft 365 email app usage report (Microsoft Graph, Reports.Read.All).
- `-IncludeSignInLogs`: Entra ID sign-ins with client app "Authenticated SMTP", with the device's IP address
  (AuditLog.Read.All, Entra ID P1 or P2). Both Graph options need the Microsoft.Graph.Authentication module.
- `-AdminCenterCsv`: the export of Exchange admin center > Reports > Mail flow > SMTP AUTH clients. It is the only source
  that shows Basic auth (`TlsAuthLogin`) versus OAuth (`XOAUTH2`).
- If PowerShell blocks the script, the user can allow it for the current window only:
  `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`.

Without the script: in Exchange Online PowerShell, `Get-TransportConfig | Format-List SmtpClientAuthenticationDisabled`
and `Get-CASMailbox -Identity user@example.com | Format-List SmtpClientAuthenticationDisabled` show where SMTP AUTH is on.

## 2. Pick the way out for each sender

Search `references/devices.md` for the device or app by vendor or product name. Each section says whether it can use
OAuth, the exact steps and menu names, the settings to enter, the fallback, and the vendor's or Microsoft's sources. The
general order:

1. **OAuth** (Microsoft calls it Modern authentication) when the device or app supports it: the long-term method. The
   mailbox keeps Authenticated SMTP on; OAuth replaces the password, not SMTP AUTH.
2. **SMTP relay connector** for devices that can't do OAuth: an inbound connector in the Exchange admin center that
   trusts the device's static public IP or TLS certificate; the device sends to the domain's MX host on port 25 without a
   login. Add that IP to SPF. Relay doesn't work from third-party clouds such as Azure.
3. **Direct Send** to the MX host on port 25 needs no connector, but only reaches the organization's own mailboxes.
4. **High Volume Email (HVE)** for mail that only goes to people inside the organization: it accepts Basic auth or OAuth
   but can't send to outside recipients.
5. **Stop-gap only:** re-enable Basic auth (`Set-TransportConfig -SmtpClientAuthenticationDisabled $false`, or per
   mailbox with `Set-CASMailbox`). It doesn't work while security defaults are on, and it ends at the final removal date.

`references/devices.md` also has the step-by-step relay connector guide (the "SMTP relay connector" section).

## 3. Verify

Send a test from each device or app, then check the SMTP AUTH clients report again until no Basic auth rows remain. If a
device reports `5.7.60`, the From address isn't the signed-in mailbox and that mailbox lacks Send As permission.

## Further help

Each device has a full page under `https://mailpassed.com/m365-smtp`. The change, with Microsoft's sources:
`https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement`.
