# Microsoft 365 SMTP AUTH: device and app guides

Generated from https://mailpassed.com/m365-smtp. One section per device or app: whether it can use OAuth, the steps, the fallback when it can't, and Microsoft's or the vendor's sources.

## HP printers and MFPs

HP Enterprise and Managed MFPs on current FutureSmart 5 firmware can sign in to Microsoft 365 with OAuth 2.0 in the Scan to Email wizard. HP says LaserJet Pro MFPs can't: use HP's cloud Scan Destinations or a Microsoft 365 relay connector before Basic auth goes off in December 2026.

**OAuth:** **LaserJet, PageWide and OfficeJet Enterprise / Managed MFPs (FutureSmart 5):** HP's firmware readmes refer to OAuth2.0 in the Embedded Web Server's **Scan to Email setup wizard**, and FutureSmart 5.9 fixed an error in that step. The readmes don't name the first version with OAuth, so install the newest FutureSmart 5 bundle for your model (5.10.0.7, dated August 2026, in the readmes we checked) and start from **Scan/Digital Send → E-mail Setup** or the **E-mail Setup Wizard**. **LaserJet Pro MFPs:** HP states they do not support OAuth 2.0 for SMTP as of spring 2026. **OfficeJet Pro, ENVY, Smart Tank and other home models:** OAuth for SMTP is not documented by HP. **HP Workpath:** an HP-made Microsoft 365 email app is not documented by HP.

**Errors you may see:**
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft's reply; reported in Microsoft Q&A)
- "unexpected error" in the Embedded Web Server while setting up OAuth2.0 in the Scan to Email wizard (HP FutureSmart readme; fixed in FutureSmart 5.9)
- Reported: the OAuth sign-in step stays on "Getting Code"; one admin fixed it by turning off IPv6 on the printer (HP Community)
- Reported: scan to email fails with event 44.10.01 SMTP protocol error even though the Microsoft sign-in succeeded (HP Community)
- Reported: the test email fails with an authentication error while the same mailbox still works in Outlook.

**Steps:**
1. Install the latest FutureSmart 5 firmware for the printer from HP's support site, then check that OAuth 2.0 is offered in the Scan to Email setup wizard.
2. Use a licensed mailbox for the scanner and keep **Authenticated SMTP** on for it: OAuth replaces the password, not SMTP AUTH itself.
3. In the Embedded Web Server open **Scan/Digital Send → E-mail Setup**, tick **Enable Send to E-mail**, and under **Outgoing E-mail Servers (SMTP)** click **Add**: `smtp.office365.com`, **Port Number** `587`, **Enable SMTP SSL Protocol** on.
4. At the authentication step choose OAuth 2.0 and finish the Microsoft sign-in with the scanner mailbox.
5. Set **Default From** to that same mailbox. Microsoft rejects other From addresses unless the mailbox has Send As permission (`5.7.60`).
6. Send a test from **Summary and Test**, then open the **SMTP AUTH clients** report in the Exchange admin center: the printer should no longer show Basic auth.

**If it can't do OAuth:** 
- **LaserJet Pro MFPs:** HP's own answer is cloud **Scan Destinations**. Pair the printer with the HP Cloud, then in the **HP Smart Admin Portal** go to **Printers** → your printer → **Scan Destinations** → **Scan to Email** → **Add Contact**. Scans go out through HP's relay, not your tenant; HP says availability varies by region and model.
- **OfficeJet Pro, ENVY and other home models:** use Scan to Email with an HP account, or Shortcuts set up in the HP Smart app on models that only support those, instead of an SMTP login.
- **Relay connector** (any HP with a static public IP): in the Exchange admin center go to **Mail flow → Connectors → Add a connector**, choose **Connection from: Your organization's email server**, and authenticate by the office's public IP. Find your MX host with our checker (https://mailpassed.com/).
- In the EWS add an SMTP server with the values below and choose **Server does not require authentication**. **Default From** can be any address in your domain; it doesn't need a mailbox.
- Add the office IP to SPF (`ip4:203.0.113.10`) with the SPF generator (https://mailpassed.com/spf-generator), then send a test scan to an outside address.
- Other ways out: direct send to the MX host on port 25 needs no connector but only reaches your own mailboxes; a third-party SMTP service gives the printer its own login; and re-enabling SMTP AUTH for just the scanner mailbox works only until Microsoft's final removal date (how (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth), timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement)).

**Settings:**
- Server Name (SMTP server): yourdomain-com.mail.protection.outlook.com
- Port Number: 25
- Authentication: Server does not require authentication
- Enable SMTP SSL Protocol: On if the test passes on port 25; HP's older help ties it to port 587, so leave it off if the test fails
- Default From: scanner@yourdomain.com (any address in your domain)
- Exchange connector: Connection from: Your organization's email server; authenticated by your static public IP

**Sources:**
- HP white paper: HP LaserJet Pro MFPs – Embedded Scan to Email and Microsoft Deprecation of Basic Authentication (v1.0, May 2026): https://kaas.hpcloud.hp.com/pdf-public/pdf_14857364_en-US-1.pdf
- HP firmware readme: Color LaserJet Managed MFP E47528f, FutureSmart 5.10.0.7 (OAuth2.0 in the Scan to Email setup wizard): https://ftp.hp.com/pub/softlib/software13/printers/E47528/readme_E47528_fs5.pdf
- HP Embedded Web Server help: Set up Scan to Email: https://ftp.hp.com/pub/printers/ews_help/MultiFunctionPeripheral/en-US/ews-email-setup.htm
- HP Support: Scan from the control panel on HP OfficeJet Pro, ENVY, DeskJet, Color LaserJet Pro, DesignJet printers: https://support.hp.com/us-en/document/ish_7279439-7279483-16
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/hp-printer-scan-to-email-microsoft-365

## Canon imageRUNNER ADVANCE and i-SENSYS/imageCLASS

imageRUNNER ADVANCE DX and newer imageCLASS/i-SENSYS models can send with Microsoft OAuth 2.0: Remote UI → Settings/Registration → Send (or TX Settings) → OAuth 2.0 (Microsoft) Settings. If that screen is missing after a firmware update, use a Microsoft 365 relay connector or uniFLOW Online.

**OAuth:** **imageRUNNER ADVANCE DX:** Canon's DX user guides (for example the C3900 series) document OAuth 2.0 for Microsoft Exchange Online. Log in to Remote UI as administrator → **Settings/Registration → Send → OAuth 2.0 (Microsoft) Settings → Basic Settings → Edit** → tick **Use OAuth 2.0 (Microsoft)**. **i-SENSYS / imageCLASS:** newer models such as the imageCLASS MF756Cx II / MF752Cdw II have the same screen under **Settings/Registration → TX Settings → OAuth 2.0 (Microsoft) Settings**. Canon says it can only be set from Remote UI, not the control panel. Canon publishes no model list or minimum firmware for OAuth in the pages we could read, so update the firmware and look for that screen. Older imageRUNNER ADVANCE (non-DX) and i-SENSYS/imageCLASS models: OAuth support not documented by Canon.

**Errors you may see:**
- #839: The user name or password specified in Communication Settings is incorrect. (Canon error code list; what a refused password login shows)
- #841: no encryption algorithm matches the mail server (Canon error code list; check the TLS setting and firmware)
- OAuth 2.0 (Microsoft) Settings: Server Connection Status not "Successfully Connected" or Token Status not "Active" means setup isn't finished (Canon user guide)
- Reported: OAuth sending stops after some days until the machine is authorized again (Canon Community)
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft's reply; reported in Microsoft Q&A)

**Steps:**
1. Update the firmware, then check the machine's date and time. Canon says to redo the OAuth setup if you change them later.
2. Use a licensed mailbox for the scanner and keep **Authenticated SMTP** on for it.
3. In Remote UI go to **Settings/Registration → Send** (**TX Settings** on i-SENSYS/imageCLASS) **→ OAuth 2.0 (Microsoft) Settings → Basic Settings → Edit**, tick **Use OAuth 2.0 (Microsoft)** and keep **Verify Server Certificate** on.
4. As **Microsoft Entra ID Authorization Server Endpoint** enter `https://login.microsoftonline.com/organizations/oauth2/v2.0` (Canon also allows `common` or `consumers`), click OK and refresh until **Server Connection Status** shows Successfully Connected.
5. Click the link next to the endpoint, enter the **User Code** at Microsoft before it expires, sign in with the scanner mailbox, then refresh until **Token Status** shows Active.
6. In the e-mail settings set the SMTP server to `smtp.office365.com` with TLS on (Microsoft's values: port 587, TLS 1.2 or later), use the signed-in mailbox as the machine's e-mail address, and send a test scan.

**If it can't do OAuth:** 
- **Relay connector:** in the Exchange admin center go to **Mail flow → Connectors → Add a connector**, choose **Connection from: Your organization's email server**, and authenticate by the office's static public IP. Find your MX host with our checker (https://mailpassed.com/).
- On imageCLASS/i-SENSYS: Remote UI → **Settings/Registration → TX Settings → Email/I-Fax Settings → Edit**. On imageRUNNER ADVANCE: the E-Mail/I-Fax **Communication Settings**. Enter the values below, turn off **SMTP Authentication (SMTP AUTH)**, and restart if asked.
- Click **Check SMTP Connection** (imageCLASS) or send a test scan to an outside address.
- Add the office IP to SPF (`ip4:203.0.113.10`) with the SPF generator (https://mailpassed.com/spf-generator) so outside recipients don't junk the scans.
- **uniFLOW Online** (Canon's paid cloud print and scan service) lists Microsoft Exchange Online and email as scan destinations, so the device itself needs no SMTP login. Ask your Canon dealer.
- Other ways out: direct send to the MX host on port 25 needs no connector but only reaches your own mailboxes; a third-party SMTP service gives the device its own login; and re-enabling SMTP AUTH for just the scanner mailbox works only until Microsoft's final removal date (how (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth), timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement)).

**Settings:**
- SMTP Server: yourdomain-com.mail.protection.outlook.com
- SMTP port: 25 (Canon's default)
- SMTP Authentication (SMTP AUTH): Off
- TLS: Use TLS for SMTP TX (imageCLASS) or Allow SSL (imageRUNNER ADVANCE): On, STARTTLS if supported
- Email Address (the machine's): scanner@yourdomain.com (any address in your domain)
- Exchange connector: Connection from: Your organization's email server; authenticated by your static public IP

**Sources:**
- Canon imageRUNNER ADVANCE DX C3935i/C3930i/C3926i/C3922i User's Guide: Using a Cloud Mail Server with OAuth 2.0: https://oip.manual.canon/USRMA-8183-zz-CS-3900-enAU/contents/devu-setup-cloud.html
- Canon imageCLASS MF756Cx II / MF752Cdw II User's Guide: Configuring a Cloud Mail Server (Remote UI): https://oip.manual.canon/USRMA-9677-zz-SSM-750II-enUV/contents/devu-scan-prep-mail-cloud_rui.html
- Canon imageCLASS MF756Cx II / MF752Cdw II User's Guide: Configuring Detailed Information of the E-mail Server (Remote UI): https://oip.manual.canon/USRMA-9677-zz-SSM-750II-enUV/contents/devu-scan-prep-mail-mail_srv_rui.html
- Canon imageRUNNER ADVANCE: List of Error Codes without Messages (#839, #841): https://oip.manual.canon/USRMA-0618-zz-CS-enLN/contents/1T0002910536.html
- uniFLOW Online: document distribution (scan destinations): https://www.uniflowonline.com/en/features/document-scanning/distribution/
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/canon-printer-scan-to-email-microsoft-365

## Ricoh MFPs and printers

Ricoh has released OAuth 2.0 firmware for Exchange Online (from July 2025) for the IM, MP, SP and P models on its support list. Update, then in Web Image Monitor go to Device Management → Configuration → Email and pick OAuth 2.0 (for Exchange Online). Models not on the list need a relay connector.

**OAuth:** Ricoh's support notice (updated 6 February 2026) lists OAuth 2.0 firmware as available for its color and monochrome IM, MP and SP MFPs, Aficio MP models, IP, P and SP printers, and small models such as the M C250FW and P C300W. Most models update with the **RICOH Firmware Update Tool**; production Pro models and the IM 2702 get firmware from your Ricoh representative. Then in **Web Image Monitor → Device Management → Configuration → Email** (under Device Settings) set **Authentication Method for Sending Email** to **OAuth 2.0 (for Exchange Online)**. Ricoh also warns that some models, including some with OAuth, need separate firmware for Microsoft's newer TLS cipher suites (released late February to late April 2026).

**Errors you may see:**
- Not documented by Ricoh: its notice only says an error occurs when scan-to-email, Internet Fax, fax-to-email and email notifications send through Exchange Online.
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft's reply; reported in Microsoft Q&A)
- "(Authenticated)" missing next to Start Authentication on the Email page means the Microsoft sign-in didn't finish (Ricoh setup guide)
- With OAuth, mail sent with a user-chosen sender fails unless it matches the authenticated address (Ricoh setup guide)
- Reported: 535 errors or timeouts with OAuth when the tenant hasn't given admin consent to Ricoh's app (Microsoft Q&A)

**Steps:**
1. Find your model on Ricoh's OAuth 2.0 list and update it with the **RICOH Firmware Update Tool**. Keep **Authenticated SMTP** on for the scanner mailbox in Microsoft 365.
2. Log in to Web Image Monitor as administrator → **Device Management → Configuration → Email** (under **Device Settings**) and set **Authentication Method for Sending Email** to **OAuth 2.0 (for Exchange Online)**. Ricoh then fixes port 587, STARTTLS and SMTP Authentication on.
3. Set **SMTP Server Name** to `smtp.office365.com`, and enter the scanner mailbox as both **Authentication Email Address** and **Authentication User Name**.
4. Click **Start Authentication** next to OAuth 2.0 Authentication, note the code, click **Microsoft**, enter the code and sign in with the scanner mailbox.
5. Back on the machine's page click **OK**, check that "(Authenticated)" appears next to Start Authentication, then click **OK** again.
6. Enter the same mailbox as **Administrator Email Address** and leave **Auto Specify Sender Name** on: with OAuth, Ricoh only sends when the sender matches the signed-in address. Then send a test scan.

**If it can't do OAuth:** 
- Ricoh's own advice for models without OAuth firmware: send without email, or use an email service other than Exchange Online. Ricoh doesn't name a cloud scan-to-email service in these notices.
- **Relay connector:** in the Exchange admin center go to **Mail flow → Connectors → Add a connector**, choose **Connection from: Your organization's email server**, and authenticate by the office's static public IP. Find your MX host with our checker (https://mailpassed.com/).
- In Web Image Monitor → **Device Management → Configuration → Email**, set **Authentication Method for Sending Email** to anything other than OAuth 2.0 (for Exchange Online) and enter the values below.
- If the model is on Ricoh's TLS cipher list with no update, set **Use Secure Connection (SSL)** to Off: Microsoft lists TLS as optional for SMTP relay.
- Add the office IP to SPF (`ip4:203.0.113.10`) with the SPF generator (https://mailpassed.com/spf-generator), then send a test scan to an outside address.
- Other ways out: direct send to the MX host on port 25 needs no connector but only reaches your own mailboxes; a third-party SMTP service gives the device its own login; and re-enabling SMTP AUTH for just the scanner mailbox works only until Microsoft's final removal date (how (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth), timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement)).

**Settings:**
- SMTP Server Name: yourdomain-com.mail.protection.outlook.com
- SMTP Port No.: 25
- SMTP Authentication: Off
- Use Secure Connection (SSL): STARTTLS (Off if the model can't do current TLS)
- Administrator Email Address: scanner@yourdomain.com (any address in your domain)
- Exchange connector: Connection from: Your organization's email server; authenticated by your static public IP

**Sources:**
- Ricoh: Impact of Microsoft basic authentication deprecation in Exchange Online and support for affected Ricoh products (updated 6 Feb 2026): https://www.ricoh.com/info/2025/0526_1
- Ricoh: Guide to Set Up OAuth 2.0 Authentication for Sending Microsoft Exchange Online E-mail (PDF): https://www.ricoh.com/-/Media/Ricoh/Sites/com/info/2025/pdf/0526_1.pdf
- Ricoh: Microsoft 365 TLS cipher suite update and affected Ricoh products (updated 30 Jan 2026): https://www.ricoh.com/info/2025/1105_1
- RICOH Firmware Update Tool: https://www.ricoh.com/support/firmware-update-tool
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/ricoh-printer-scan-to-email-microsoft-365

## Xerox AltaLink and VersaLink

Xerox has OAuth 2.0 firmware (Device Code Flow) for current AltaLink and VersaLink models. In the Embedded Web Server open Apps → Email → SMTP Setup and pick OAuth 2.0 with SMTP (AltaLink) or Service Provider: Exchange Online/Outlook.com (most VersaLink). Unlisted models need a relay connector.

**OAuth:** Xerox's Exchange Online notice (February 2026) lists Device Code Flow firmware as **Available Now** for the AltaLink B8100, C8100, B8200 and C8200 series and for VersaLink B400/C400, B405/C405, B415/C415, C500/C505, B600 to C605, B610/B615, B620/C620/B625/C625 and the B7000, C7000, B7100 and C7100 series; AltaLink and some VersaLink models also support Client Credentials Flow. Minimum software: AltaLink 81xx/82xx `122.xxx.025.20910`, VersaLink B400 to C605 `xx.83.11` (PL8-R10). **AltaLink:** **Properties → Apps → Email → Setup** → Edit next to SMTP → **SMTP Authentication** = **OAuth 2.0 with SMTP** → **Device Code Flow (DCF) Configuration**. **VersaLink:** **Apps → Email** → SMTP **Setup** → **Service Provider** = **Exchange Online/Outlook.com**. Xerox says products not in its list don't support OAuth 2.0 email.

**Errors you may see:**
- 016-781 Server Connection Error on the confirmation report (VersaLink; Xerox says a business or school account may need admin consent)
- Test configuration fails because the account lacks "Permission to use SMTP" (AltaLink; admin consent or Authenticated SMTP missing)
- Mail stops after working for a while: Xerox says to de-register and re-register Device Code Flow, as the printer's sign-in may have expired
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft's reply; reported in Microsoft Q&A)

**Steps:**
1. Update the printer to at least the software level above; Xerox keeps the current list at Xerox.com/MSOAuth2.
2. Have a Microsoft 365 admin approve Xerox's Device Code Flow app at `https://www.xerox.com/oauth2-email-admin-consent` (optionally limited to the scanner mailbox), and keep **Authenticated SMTP** on for that mailbox.
3. **AltaLink:** Embedded Web Server → **Properties → Apps → Email → Setup** → Edit next to SMTP → set **SMTP Authentication** to **OAuth 2.0 with SMTP**, choose **Device Code Flow (DCF) Configuration**, save and apply.
4. **VersaLink:** Embedded Web Server → **Apps → Email** → SMTP **Setup** → set **Service Provider** to **Exchange Online/Outlook.com** (it fills in `smtp.office365.com`, port 587) → OK.
5. Copy the code, open the Microsoft registration page, enter the code and sign in with the scanner mailbox, then accept the permissions and restart if asked. That mailbox becomes the From address for all device mail.
6. Test with **Test configuration** (AltaLink) or the Email app on the control panel (VersaLink).

**If it can't do OAuth:** 
- **Relay connector:** in the Exchange admin center go to **Mail flow → Connectors → Add a connector**, choose **Connection from: Your organization's email server**, and authenticate by the office's static public IP. Find your MX host with our checker (https://mailpassed.com/).
- In the Embedded Web Server's SMTP settings (AltaLink: **Properties → Apps → Email → Setup**), enter the values below under **Required Information**, set no login for SMTP Authentication, and set the **From Field**.
- Add the office IP to SPF (`ip4:203.0.113.10`) with the SPF generator (https://mailpassed.com/spf-generator), then send a test scan to an outside address.
- Xerox notes that other providers, such as Gmail, aren't affected by Microsoft's change, so a third-party SMTP service with its own login also works.
- Other ways out: direct send to the MX host on port 25 needs no connector but only reaches your own mailboxes, and re-enabling SMTP AUTH for just the scanner mailbox works only until Microsoft's final removal date (how (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth), timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement)).

**Settings:**
- Host Name/IP: yourdomain-com.mail.protection.outlook.com
- Port: 25
- SMTP Authentication: None (no login)
- Connection Encryption: STARTTLS (if supported)
- From Field (Default From): scanner@yourdomain.com (any address in your domain)
- Exchange connector: Connection from: Your organization's email server; authenticated by your static public IP

**Sources:**
- Xerox: Our Response to Microsoft Exchange Online Basic Authentication Phase-out (products and firmware availability, Feb 2026): https://www.office.xerox.com/latest/SFTDS-12U.pdf
- Xerox: Device Configuration with OAuth 2.0 for Microsoft 365 Exchange Online Configuration Guide (Aug 2025): https://download.support.xerox.com/pub/docs/ALB81XX/userdocs/any-os/en_GB/Xerox_Device_Configuration_OAuth_Microsoft_365_Exchange_Online_Config_Guide_en-US.pdf
- Xerox KB0431230: Configuring Scan to Email Using OAuth2.0 (AltaLink): https://www.support.xerox.com/en-us/article/KB0431230
- Xerox KB0431245: Configuring Scan to Email Using OAuth2.0 (VersaLink): https://www.support.xerox.com/en-us/article/KB0431245
- Xerox KB0426874: How to Configure Scan to Email (SMTP) (AltaLink): https://www.support.xerox.com/en-us/article/KB0426874
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/xerox-printer-scan-to-email-microsoft-365

## Brother printers, MFCs and scanners

Many Brother MFC, DCP, HL and ADS models support OAuth 2.0 for Exchange Online, some only after a firmware update: check Brother's support list. Set it in Web Based Management → Network → Protocol → POP3/IMAP4/SMTP Client → Third party (OAuth2) → Outlook. Unlisted models need a relay connector.

**OAuth:** Brother's **OAuth 2.0 Authentication Support Statement** has a Product Support List. Models marked ※1 already support OAuth 2.0 and only need configuring (for example MFC-L2800DW, MFC-L2960DW, DCP-L2620DW, HL-L2865DW, MFC-L8730CDW, MFC-EX570 and MFC-J3660DW). Models marked ※2 need a firmware update first (for example MFC-L2710DW, DCP-L2530DW, HL-L3210CW, MFC-L6970DW and the ADS document scanners). Brother says models not on the list don't support OAuth 2.0. Path: Web Based Management (`https://printer-IP/`) → **Network → Network → Protocol** → **Advanced Settings** next to **POP3/IMAP4/SMTP Client** → **Third party (OAuth2)** → **Outlook**.

**Errors you may see:**
- Authentication Error (Brother FAQ: Microsoft Exchange Online without OAuth 2.0)
- Receiving Error (same Brother FAQ)
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft's reply; reported in Microsoft Q&A)
- Reported: Scan to E-mail Server stops while the same mailbox still works in Outlook (Microsoft Q&A)

**Steps:**
1. Find your model on Brother's Product Support List. If it is marked ※2, install the latest firmware first.
2. Keep **Authenticated SMTP** on for the scanner mailbox in Microsoft 365.
3. Open Web Based Management at `https://printer-IP/` and log in (the default password is on the label marked "Pwd"). Go to **Network → Network → Protocol** and click **Advanced Settings** next to **POP3/IMAP4/SMTP Client**.
4. Click **Third party (OAuth2)** → **Outlook**, enter the scanner mailbox address, pick the functions and protocol, and click **OK**.
5. Note the code, click **Sign in**, enter it at Microsoft and finish the sign-in. Close that tab without clicking Sign in again, then click **OK** and **Submit**.
6. Send a test scan to an outside address.

**If it can't do OAuth:** 
- Brother's own advice for unlisted models is another email service, such as Gmail. A third-party SMTP service with its own login works the same way.
- **Relay connector:** in the Exchange admin center go to **Mail flow → Connectors → Add a connector**, choose **Connection from: Your organization's email server**, and authenticate by the office's static public IP. Find your MX host with our checker (https://mailpassed.com/).
- In Web Based Management → **Network → Network → Protocol** → **POP3/IMAP4/SMTP Client** (POP3/SMTP on older models), enter the values below and set **Server Authentication Method** to None.
- Add the office IP to SPF (`ip4:203.0.113.10`) with the SPF generator (https://mailpassed.com/spf-generator), then send a test scan to an outside address.
- Other ways out: direct send to the MX host on port 25 needs no connector but only reaches your own mailboxes, and re-enabling SMTP AUTH for just the scanner mailbox works only until Microsoft's final removal date (how (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth), timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement)).

**Settings:**
- SMTP server: yourdomain-com.mail.protection.outlook.com
- SMTP port: 25
- Server Authentication Method: None
- SMTP over SSL/TLS: TLS (STARTTLS) if supported, otherwise None
- Machine's e-mail address: scanner@yourdomain.com (any address in your domain)
- Exchange connector: Connection from: Your organization's email server; authenticated by your static public IP

**Sources:**
- Brother: OAuth 2.0 Authentication Support Statement (Product Support List): https://support.brother.com/g/b/oscontents.aspx?c=eu_ot&lang=en&ossid=42
- Brother FAQ: I get the "Authentication Error" or "Receiving Error" message when using Microsoft Exchange Online: https://support.brother.com/g/b/faqend.aspx?c=us_ot&lang=en&prod=hll5210dw_us_eu_as&faqid=faq00100853_000
- Brother FAQ: Send an Email with User Authentication: https://support.brother.com/g/b/faqend.aspx?c=us&lang=en&prod=mfcl8900cdw_all&faqid=faq00100234_508
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/brother-printer-scan-to-email-microsoft-365

## Konica Minolta bizhub

Most bizhub i-Series and many older bizhub models get Microsoft 365 OAuth 2.0 from firmware F233-G00-T8(00) or F000-GC3-YE(00). After the update, fetch the token on the control panel. Models marked N/A get no update: use an SMTP relay connector or scan to a folder.

**OAuth:** Konica Minolta has released OAuth 2.0 firmware for its main bizhub models (notice updated 9 March 2026). Product Group #1, bizhub i-Series models such as C250i, C300i, C360i, C450i, C550i, C650i, C750i, the C251i–C751i successors, C3300i/C4000i, C3350i/C4050i and the mono 226i–950i, needs **F233-G00-T8(00)**. Product Group #2, for example C258/C308/C368, C458/C558/C658, C659/C759, 308e/368e, 458e/558e/658e, 758/808/958 and C3351/C3851, needs **F000-GC3-YE(00)**. Smaller A4 models such as bizhub 4000i/5000i, 4020i/5020i, C3100i/C3120i, 225i/245i and 4702P get OAuth for Exchange Online at model-specific versions. The token is fetched on the control panel at **Utility > Administrator > Network > E-mail Setting > E-mail TX (SMTP) > OAuth Settings** (Group #2: **Utility > Administrator Settings > Network Settings > E-Mail Settings > E-Mail TX (SMTP) > Detail Settings > OAuth 2.0**); Konica Minolta says this can't be done in Web Connection. No firmware is planned for models marked N/A, such as bizhub C226/C256/C266, 266/306, 4052/4752, 4050/4750, C3110, C654e/C754e and 206/226/246.

**Errors you may see:**
- Device-side error text for a refused password login: not documented by Konica Minolta (its notice only says the e-mail functions "may result in errors").
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled (reported)
- 535 5.7.3 Authentication unsuccessful (reported)
- "The OAuth authentication fails when sending Microsoft E-mails." (Konica Minolta OAuth setup guide; its fix is to tick Authenticated SMTP for the mailbox)
- A warning on the [Information] screen when the administrator's token has expired (Konica Minolta OAuth setup guide).

**Steps:**
1. Ask your Konica Minolta dealer for the OAuth firmware: **F233-G00-T8(00)** for Group #1 or **F000-GC3-YE(00)** for Group #2. Konica Minolta applies it remotely or at the next service visit.
2. In the Microsoft 365 admin center, open the scanner's mailbox (**Users > Active users > Mail > Manage email apps**) and tick **Authenticated SMTP**. OAuth sign-in still needs SMTP AUTH on for that mailbox.
3. Enter the Microsoft 365 mailbox as the administrator address: **[System Settings] > [Machine Setting] > [Administrator Registration] > [E-mail Address]** in Web Connection. Group #2 models also need the Web browser function on first: **Utility > Administrator Settings > Network Settings > Web Browser Setting > Web Browser Usage Settings**.
4. On the control panel, open **OAuth Settings** (path above), set **OAuth 2.0** to ON and **OAuth 2.0 Provider** to Microsoft, tap OK and sign in. **Refresh Tokens (Administrator)** changes from Required to Acquired.
5. Set the SMTP server address to `smtp.office365.com` under **[Network] > [E-mail Setting] > [E-mail TX (SMTP)]**, with Start TLS on port 587 (Microsoft's values). Don't set both the minimum and maximum TLS version to TLSv1.3: Konica Minolta says Microsoft OAuth then fails.
6. Choose the sender in **Change From Address Setting**: Fix Administrator Address sends every scan with the admin token; Login User Address makes each user sign in on the panel the first time.
7. Send a test scan. The refresh token lasts several months; when the **[Information]** warning appears, tap **[Acquire]** to sign in again. More on the 535 5.7.139 error (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth).

**If it can't do OAuth:** 
- No OAuth firmware for your model (marked N/A or not listed)? Konica Minolta's own workarounds are scanning to SMB, Home, FTP, WebDAV or Box instead of e-mail, or using a mail service other than Exchange Online. To keep Scan to E-mail on Microsoft 365, use an SMTP relay connector.
- In the Exchange admin center go to **Mail flow > Connectors > Add a connector**, choose **Connection from: Your organization's email server**, and let it trust the office's static public IP (one nobody else uses) or a TLS certificate for your domain. Relay only works from your own network, not from a cloud host such as Azure.
- On the panel, open **Utility > Administrator > Network > E-mail Setting > E-mail TX (SMTP)** and enter the values in the table. Under **Detail Settings**, turn **SMTP Authentication** and **POP Before SMTP** OFF.
- Add the office's public IP to your SPF record, for example `v=spf1 ip4:203.0.113.10 include:spf.protection.outlook.com -all`, with our SPF generator (https://mailpassed.com/spf-generator). Run our checker (https://mailpassed.com/) to see your MX host and current SPF.
- Only scanning to colleagues? Direct send uses the same MX host and port 25 with no connector and no login, but it can't deliver to outside addresses, and Microsoft says it is working on turning it off by default.
- Or give the device its own SMTP login at an email sending service (for example Amazon SES, Postmark, Brevo or Mailgun) and publish that service's SPF and DKIM records for your domain.
- Stopgap only: an admin can turn SMTP AUTH back on for just the device's mailbox with `Set-CASMailbox -Identity scanner@yourdomain.com -SmtpClientAuthenticationDisabled $false`. It fails while security defaults are on and ends when Microsoft sets the final removal date (to be announced in the second half of 2027). See the timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- E-mail TX Setting: ON
- SMTP Server Address: yourdomain-com.mail.protection.outlook.com (your MX host)
- SSL/TLS Settings: Start TLS
- Port No.: 25
- SMTP Authentication: OFF
- Device Mail Address: scanner@yourdomain.com (any address in your Microsoft 365 domain)

**Sources:**
- Konica Minolta: Notice on the discontinuation of Basic Authentication for SMTP in Exchange Online / Google Workspace (updated 9 Mar 2026): https://www.konicaminolta.com/global-en/security/notices/pdf/OAuth2.0_support_notice_for_SMTP_authentication.pdf
- Konica Minolta Europe: OAuth2.0 Information, impact on Konica Minolta products (1 Apr 2026): https://www.konicaminolta.eu/eu-en/support/oauth2.0-information
- Konica Minolta: OAuth 2.0 Setup Guide #1 (ADXJ-9624-00): https://assets-konicaminolta-eu.canto.global/direct/document/808m94828h6j54u3te44a2in68/fbrpYvVO8Pi7aUQVlXWWufOit_I/original?content-type=application%2Fpdf&name=OAuth_2.0_Setup_Guide_1_ADXJ-9624-00.pdf
- Konica Minolta: OAuth 2.0 Setup Guide #2 (A7AK-9576-00): https://assets-konicaminolta-eu.canto.global/direct/document/sp6ha07pk575dbjfm3jr3tl74r/ToDNYKMl48oOOzMSbIofcdw8L5Y/original?content-type=application%2Fpdf&name=OAuth_2.0_Setup_Guide_2_A7AK-9576-00.pdf
- Konica Minolta bizhub 750i online manual: E-mail TX (SMTP): https://manuals.konicaminolta.eu/bizhub-750i-UD/EN/contents/opkey_557_S.html
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365 (updated 5 Oct 2026): https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/konica-minolta-bizhub-scan-to-email-microsoft-365

## Kyocera TASKalfa and ECOSYS

Newer Kyocera TASKalfa MZ and ECOSYS MA/PA models sign in to Microsoft 365 with OAuth2 in Command Center RX, and TASKalfa xx54ci and x004i models get it by firmware update. Older Kyocera devices can use Kyocera's free Exchange Online Connector or an SMTP relay connector.

**OAuth:** Kyocera lists native OAuth 2.0 for SMTP on TASKalfa MZ2501ci–MZ7001ci, MZ5001i–MZ7001i, MZ3200i/MZ4000i and MA3500ci/MA4500ci, and on ECOSYS models such as MA2101/MA2600, PA2101/PA2600, MA3500/MA4000/MA4500, MA5500ifx/MA6000ifx, PA3500–PA6000 and P4060dn/P8060dn. TASKalfa 2554ci–7054ci and 5004i–7004i are on Kyocera UK's list but need a firmware update (Kyocera Australia: end of March 2026 for the 2554ci). In **Command Center RX** go to **Function Settings > E-mail > SMTP > Authentication Protocol**, choose **OAuth2**, then **OAuth2 Status > Authorize**. Kyocera says only the mail address set for the device can be used, not a separate login per user. Firmware version numbers: not documented by Kyocera.

**Errors you may see:**
- "1102 – Login Username and/or Password is incorrect." (Kyocera error code)
- "3101 – SMTP Server rejected email. Refer issue to IT or check Authentication/Sender address" (Kyocera error code)
- "1104 – SMTP denied access." (Kyocera error code)
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled (reported)

**Steps:**
1. Install the latest firmware and check that your model is on Kyocera's OAuth 2.0 list.
2. In the Microsoft 365 admin center, open the scanner's mailbox (**Users > Active users > Mail > Manage email apps**) and tick **Authenticated SMTP**. OAuth sign-in still needs SMTP AUTH on for that mailbox. Whoever authorizes the device must be allowed to grant consent for your organization.
3. In Command Center RX, open **Network Settings > Protocol**: set **SMTP (E-mail TX)** under Send Protocols to On and **SMTP Security** to **STARTTLS**, then Submit.
4. Open **Function Settings > E-mail**: **SMTP Server Name** `smtp.office365.com`, **SMTP Port Number** 587, **Authentication Protocol** OAuth2, then Submit.
5. Click **OAuth2 Status > Authorize**, copy the code shown under Microsoft Exchange Server Authorization, open the link, paste the code and sign in (with MFA if asked). Choose **Consent on behalf of your organization > Accept**, then refresh in Command Center RX and click OK and Submit.
6. Set **Sender Address** to the same mailbox and click **Test**. Behind a proxy? Set it in **Network Settings > TCP/IP** (Proxy Settings) before you authorize. More on the 535 5.7.139 error (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth).

**If it can't do OAuth:** 
- Kyocera's free **Exchange Online Connector** (Windows PC or server, 4-core CPU, 8 GB RAM) signs in to Microsoft 365 with OAuth for the device, with no firmware change. Install it, click **Add**, sign in with the scanner mailbox (Authenticated SMTP must be on), then click **Start**.
- Point the device at the connector in **Function Settings > E-mail**: **SMTP Server Name** = the connector PC's IP, **SMTP Port Number** and **SMTP Security** = the values set in the connector (587 with STARTTLS, or 25 unencrypted), **SMTP Server Timeout** 60 or more, **Authentication Protocol** On, **Authentication as** Other, and the **Login User Name** and **Login Password** the connector shows.
- No PC to spare? Use an SMTP relay connector instead. In the Exchange admin center go to **Mail flow > Connectors > Add a connector**, choose **Connection from: Your organization's email server**, and let it trust the office's static public IP (one nobody else uses) or a TLS certificate for your domain. Relay only works from your own network, not from a cloud host such as Azure.
- Enter the relay values from the table in Command Center RX (**Function Settings > E-mail**; SMTP Security is under **Network Settings > Protocol**).
- Add the office's public IP to your SPF record, for example `v=spf1 ip4:203.0.113.10 include:spf.protection.outlook.com -all`, with our SPF generator (https://mailpassed.com/spf-generator). Run our checker (https://mailpassed.com/) to see your MX host and current SPF.
- Only scanning to colleagues? Direct send uses the same MX host and port 25 with no connector and no login, but it can't deliver to outside addresses, and Microsoft says it is working on turning it off by default. Or give the device its own SMTP login at an email sending service (for example Amazon SES, Postmark, Brevo or Mailgun) and publish that service's SPF and DKIM records for your domain.
- Stopgap only: an admin can turn SMTP AUTH back on for just the device's mailbox with `Set-CASMailbox -Identity scanner@yourdomain.com -SmtpClientAuthenticationDisabled $false`. It fails while security defaults are on and ends when Microsoft sets the final removal date (to be announced in the second half of 2027). See the timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- SMTP (E-mail TX), under Network Settings > Protocol: On
- SMTP Security: STARTTLS
- SMTP Server Name: yourdomain-com.mail.protection.outlook.com (your MX host)
- SMTP Port Number: 25
- Authentication Protocol: Off
- Sender Address: scanner@yourdomain.com (any address in your Microsoft 365 domain)

**Sources:**
- Kyocera UK: Kyocera OAuth2.0 authentication support (4 Dec 2025): https://www.kyoceradocumentsolutions.co.uk/en/about-us/contact-us/press/kyocera-oauth2-authentication-support.html
- Kyocera ANZ: OAuth2.0 authentication support (30 Jan 2026): https://kyoceradocumentsolutions.com.au/news/oauth2.0-authentication-support
- Kyocera: Command Center RX User Guide 2025.11 (OAuth2 for Microsoft 365): https://dam.kyoceradocumentsolutions.com/content/dam/gdam_dc/dc_global/document/manual/product_086/commandcenterrx_ecosysma4000wifx_en.pdf
- Kyocera: Exchange Online Connector Utility User Guide (2025.01): https://kyoceradocumentsolutions.com.au/hubfs/ExchangeOnline_UserGuide_EN.pdf
- Kyocera ANZ: Set up scan to email via Command Centre (error codes): https://kyoceradocumentsolutions.com.au/knowledge/scan-to-email
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365 (updated 5 Oct 2026): https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/kyocera-printer-scan-to-email-microsoft-365

## Sharp MX and BP series

Sharp documents OAuth 2.0 for Microsoft 365 on its BP-series MFPs: choose OAuth 2.0 in the SMTP settings and get the token at the control panel. For MX-series models, OAuth is not documented by Sharp, so plan on an SMTP relay connector, direct send or another mail service.

**OAuth:** Sharp's BP-series manuals (for example BP-70C65, BP-70M65 and BP-71C65) document **OAuth 2.0** as the recommended SMTP method for Microsoft 365 and Exchange Online: **Settings (Administrator) > System Settings > Network Settings > Service Settings > SMTP** tab, **Authentication Option** **OAuth 2.0**, **Provider** Microsoft. The token must be fetched at the machine; Sharp says the setup can't be completed on the web page. For the MX series, OAuth is not documented by Sharp: the MX-2651, MX-3051, MX-3071, MX-3551, MX-3571, MX-4051 and MX-4071 user's manual offers only SMTP Authentication with a user name and password, and the MX-C428F/C528F guide lists Login / Plain for Microsoft 365. Sharp New Zealand says devices without OAuth 2.0 need another email service.

**Errors you may see:**
- "Token update failed. Please delete the token and obtain it again." (Sharp BP-series connection test)
- Token status "Invalid": the account can't be used or the token has expired (Sharp BP-series manual)
- Device-side error text for a refused password login on MX models: not documented by Sharp.
- 80-0000 on an MX-C304W sending to Microsoft 365 through a connector (reported in Microsoft Q&A)
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled (reported)

**Steps:**
1. Check that your model's online manual has an **OAuth 2.0 AUTHENTICATION** section. Sharp documents it for BP-series models such as the BP-70C65, BP-70M65 and BP-71C65.
2. In the Microsoft 365 admin center, open the scanner's mailbox (**Users > Active users > Mail > Manage email apps**) and tick **Authenticated SMTP**. OAuth sign-in still needs SMTP AUTH on for that mailbox.
3. At the machine, tap **Settings (Administrator) > System Settings > Network Settings > Service Settings > SMTP** tab and set **Primary Server** `smtp.office365.com`, **Port Number** 587, **Enable SSL/TLS** on and **Sender Address** to the mailbox (Microsoft's values).
4. Set **Authentication Option** to **OAuth 2.0**, **Provider** to Microsoft and **Account Name** to the mailbox. Tap **[Get Token]**, then OK, and sign in. **Token** changes from Not Retrieved to Retrieved.
5. E-mail alerts use a separate SMTP setting (**System Settings > System Control > E-mail Alert and Status > SMTP Settings**) that needs its own token.
6. Run the connection test. If the token update failed or the token shows Invalid, tap **[Delete]** and get a new token. More on the 535 5.7.139 error (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth).

**If it can't do OAuth:** 
- MX models and other devices without OAuth: Sharp New Zealand says to use another email service, such as your own mail server or a different provider. Through Microsoft 365, that means an SMTP relay connector.
- In the Exchange admin center go to **Mail flow > Connectors > Add a connector**, choose **Connection from: Your organization's email server**, and let it trust the office's static public IP (one nobody else uses) or a TLS certificate for your domain. Relay only works from your own network, not from a cloud host such as Azure.
- On the machine or its web page, open **Settings (administrator) > System Settings > Network Settings > Services Settings > SMTP** and enter the values in the table, then run **Connection Test**.
- Add the office's public IP to your SPF record, for example `v=spf1 ip4:203.0.113.10 include:spf.protection.outlook.com -all`, with our SPF generator (https://mailpassed.com/spf-generator). Run our checker (https://mailpassed.com/) to see your MX host and current SPF.
- Only scanning to colleagues? Direct send uses the same MX host and port 25 with no connector and no login, but it can't deliver to outside addresses, and Microsoft says it is working on turning it off by default.
- Or give the device its own SMTP login at an email sending service (for example Amazon SES, Postmark, Brevo or Mailgun) and publish that service's SPF and DKIM records for your domain.
- Stopgap only: an admin can turn SMTP AUTH back on for just the device's mailbox with `Set-CASMailbox -Identity scanner@yourdomain.com -SmtpClientAuthenticationDisabled $false`. It fails while security defaults are on and ends when Microsoft sets the final removal date (to be announced in the second half of 2027). See the timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Primary Server: yourdomain-com.mail.protection.outlook.com (your MX host)
- Port Number: 25
- Enable SSL: On (uses STARTTLS)
- SMTP Authentication: Off
- POP before SMTP: Off
- Sender Address: scanner@yourdomain.com (any address in your Microsoft 365 domain)

**Sources:**
- Sharp BP-70C65 User's Manual: OAuth 2.0 authentication: https://global.sharp/restricted/products/copier/downloads/manuals/bp70c65/us/contents_08-03_004.html
- Sharp BP-70M65 User's Manual: Network Settings (SMTP): https://global.sharp/restricted/products/copier/downloads/manuals/bp70m65/en/contents_09-07_018.html
- Sharp MX-2651/3051/3071/3551/3571/4051/4071 User's Manual (PDF): https://business.sharpusa.com/portals/0/downloads/Manuals/MX_2651-4071_MX_3071-4071_User_Manual.pdf
- Sharp MX-C428F/MX-C528F User's Guide (Jan 2023, PDF): https://business.sharpusa.com/portals/0/downloads/Manuals/MX-C428F_C528F_User-Guide.pdf
- Sharp New Zealand: Microsoft 365 / Exchange Online retiring Basic Authentication (SMTP Auth): https://www.sharp.net.nz/microsoft365-exchange
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365 (updated 5 Oct 2026): https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/sharp-mx-scan-to-email-microsoft-365

## Lexmark

Lexmark printers support Microsoft 365 OAuth 2.0 from firmware FW24 (240.043 or later). Update, register the printer in the Embedded Web Server's Email settings (Set Up OAuth 2.0 for Email Server), then use smtp.office365.com on port 587. Printers that can't run FW24 need an SMTP relay connector.

**OAuth:** Lexmark says its printers support OAuth 2.0 for Outlook Live and Microsoft 365 starting with the **FW24** firmware release; the FW24 release notes list support for OAuth2.0 with Microsoft SMTP servers from **240.043**. FW24 is published for Lexmark's firmware families CT, GM, GV, GW, LS, MM, PC, PM, SN and ZJ (the version looks like `CXTPC.240.229`). In the Embedded Web Server go to **Settings > Email > Set Up OAuth 2.0 for Email Server** and click **Register**. Fax server workflows can use their own OAuth registration, and the control-panel email setup wizard can't set up OAuth. A list of model names per firmware family: not documented by Lexmark in one place, so check the firmware version on your printer.

**Errors you may see:**
- "Email Transmission Failure" on the control panel, followed by a printed Scan Log (Lexmark)
- "Connection Failure ": user name and password not accepted (Lexmark Scan Log code)
- "Connection Failure ": the authentication setting doesn't match what the server requires (Lexmark Scan Log code)
- Status "Registration required" under Set Up OAuth 2.0 for Email Server: the printer isn't registered yet (Lexmark)
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled (reported)

**Steps:**
1. Update the printer to FW24 (firmware 240.043 or later) from Lexmark's drivers and downloads page.
2. In the Microsoft 365 admin center, open the scanner's mailbox (**Users > Active users > Mail > Manage email apps**) and tick **Authenticated SMTP**. OAuth sign-in still needs SMTP AUTH on for that mailbox.
3. In the Embedded Web Server, open **Settings > Email > Set Up OAuth 2.0 for Email Server**. The Status should say Registration required; click **Register**.
4. Go to `https://microsoft.com/devicelogin`, enter the code from the Embedded Web Server within 15 minutes, sign in with the scanner mailbox and follow the prompts. The Status changes to Registered.
5. Open **Settings > Email > Email Setup**: **Primary SMTP Gateway** `smtp.office365.com`, **Primary SMTP Gateway Port** 587, **Use SSL/TLS** Required, **Reply Address** = the registered mailbox, **SMTP Server Authentication** OAuth2.0, **Device-Initiated Email** and **User-Initiated Email** Use Device SMTP Credentials. Click Save.
6. Fax server workflows: in **Settings > Fax**, open **Fax Server Setup** and tick **Use Email SMTP Server**, or register a separate address under **Set Up OAuth 2.0 for Fax Server**.
7. Send a test scan. If the panel shows Email Transmission Failure, read the code on the printed Scan Log. More on the 535 5.7.139 error (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth).

**If it can't do OAuth:** 
- Firmware still not 240.x after updating? Then the model has no OAuth, and a Lexmark cloud alternative for e-mail is not documented by Lexmark. Use an SMTP relay connector.
- In the Exchange admin center go to **Mail flow > Connectors > Add a connector**, choose **Connection from: Your organization's email server**, and let it trust the office's static public IP (one nobody else uses) or a TLS certificate for your domain. Relay only works from your own network, not from a cloud host such as Azure.
- In the Embedded Web Server, open **Settings > Email > Email Setup** (older models: **Settings > Email/FTP Settings > Email Settings > Setup Email Server**) and enter the values in the table.
- Add the office's public IP to your SPF record, for example `v=spf1 ip4:203.0.113.10 include:spf.protection.outlook.com -all`, with our SPF generator (https://mailpassed.com/spf-generator). Run our checker (https://mailpassed.com/) to see your MX host and current SPF.
- Only scanning to colleagues? Direct send uses the same MX host and port 25 with no connector and no login, but it can't deliver to outside addresses, and Microsoft says it is working on turning it off by default.
- Or give the device its own SMTP login at an email sending service (for example Amazon SES, Postmark, Brevo or Mailgun) and publish that service's SPF and DKIM records for your domain.
- Stopgap only: an admin can turn SMTP AUTH back on for just the device's mailbox with `Set-CASMailbox -Identity scanner@yourdomain.com -SmtpClientAuthenticationDisabled $false`. It fails while security defaults are on and ends when Microsoft sets the final removal date (to be announced in the second half of 2027). See the timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Primary SMTP Gateway: yourdomain-com.mail.protection.outlook.com (your MX host)
- Primary SMTP Gateway Port: 25
- Use SSL/TLS: Negotiate
- SMTP Server Authentication: No authentication required
- Device-Initiated Email: None
- User-Initiated Email: None
- Reply Address: scanner@yourdomain.com (any address in your Microsoft 365 domain)

**Sources:**
- Lexmark: How to set up OAuth 2.0: https://support.lexmark.com/en_us/manuals-guides/online/CX735/how-to-set-up-oauth-2-authentication.html
- Lexmark: FW24.0 (240.229) release notes (May 2025): https://support.lexmark.com/content/support/guides/en/kb20211007050829871/firmware/firmware-release-notes/firmware-release-notes-history/fw24-0-240-229-release-notes.html
- Lexmark MB2236 MFP User's Guide (April 2026, PDF): https://support.lexmark.com/content/dam/support/collateral/lexmark_hardware/mb2236/ug/pdf/Lexmark_MB2236_UserGuide_en.pdf
- Lexmark: Email transmission error codes and recommendations: https://support.lexmark.com/content/support/guides/en/kb20211028132247705/e-mail-issues/e-mail-transmission-error-codes-and-recommendation.html
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365 (updated 5 Oct 2026): https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/lexmark-printer-scan-to-email-microsoft-365

## Epson WorkForce and EcoTank business

Epson added OAuth2 for Microsoft Exchange Online to listed WorkForce, WorkForce Pro, WorkForce Enterprise and EcoTank models by firmware update. Update, then in Web Config set Authentication Method to OAuth2 under Network, Email Server, Basic and sign in. Other models need an SMTP relay connector.

**OAuth:** Epson's FAQ **OAuth2.0 support for Epson printers and scanners** (KA-02047) lists the models that got OAuth2.0 for Microsoft Exchange Online and says to install the latest firmware. Business models on the list include WorkForce WF-3820/3825, WF-4820/4830, WF-7820/7830/7840, WorkForce Pro WF-C5290/C5390, WF-C5710/C5790/C5810/C5890, WF-M5299/M5399/M5799/M5899 and EM-C800/EM-C8100, WorkForce Enterprise AM-C4000/C5000/C6000, AM-C400/C550 and WF-C20590/C21000, and EcoTank ET-5800/5850/5880, ET-16600/16650/16680, ET-8700 and ET-M16600. In Web Config (logged in as administrator) go to **Network** tab > **Email Server** > **Basic**, set **Authentication Method** to **OAuth2** and **Email service** to **Microsoft Exchange Online**, then click **Sign In**. Minimum firmware versions: not documented by Epson. Models not on the list: OAuth not documented by Epson.

**Errors you may see:**
- "SMTP server authentication error. Check the followings. - Authentication Method - Authenticated Account -Authenticated Password" (Epson connection test)
- "Sender's Email Address is incorrect. Change to the email address for your email service." (Epson connection test)
- "Connection to SMTP server failed. Change Secure Connection to STARTTLS." (Epson connection test)
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled (reported)
- 535 5.7.3 Authentication unsuccessful (reported)

**Steps:**
1. Check that your model is on Epson's OAuth2.0 list (FAQ KA-02047) and install the latest firmware.
2. In the Microsoft 365 admin center, open the scanner's mailbox (**Users > Active users > Mail > Manage email apps**) and tick **Authenticated SMTP**. OAuth sign-in still needs SMTP AUTH on for that mailbox. Epson says the account that signs in needs the **Cloud Application Administrator** role.
3. Open Web Config, log in as administrator and go to **Network** tab > **Email Server** > **Basic**. Set **Authentication Method** to **OAuth2** and **Email service** to **Microsoft Exchange Online**.
4. Click **Sign In** > **Sign in with Microsoft**, copy the code, open the link shown, enter the code and sign in. Accept consent on behalf of your organization. When the status says Signed in, click OK.
5. Set **Sender's Email Address** to the signed-in mailbox, run the connection test and send a test scan. More on the 535 5.7.139 error (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth).

**If it can't do OAuth:** 
- Model not on Epson's list? If it shows **Scan to Cloud**, Epson Connect can e-mail scans to destinations you register in Epson Connect, but the mail comes from the printer's Epson Connect address, not your domain. To send from your domain, use an SMTP relay connector.
- In the Exchange admin center go to **Mail flow > Connectors > Add a connector**, choose **Connection from: Your organization's email server**, and let it trust the office's static public IP (one nobody else uses) or a TLS certificate for your domain. Relay only works from your own network, not from a cloud host such as Azure.
- In Web Config, open **Network** tab > **Email Server** > **Basic** and enter the values in the table, then run the connection test.
- Add the office's public IP to your SPF record, for example `v=spf1 ip4:203.0.113.10 include:spf.protection.outlook.com -all`, with our SPF generator (https://mailpassed.com/spf-generator). Run our checker (https://mailpassed.com/) to see your MX host and current SPF.
- Only scanning to colleagues? Direct send uses the same MX host and port 25 with no connector and no login, but it can't deliver to outside addresses, and Microsoft says it is working on turning it off by default.
- Or give the device its own SMTP login at an email sending service (for example Amazon SES, Postmark, Brevo or Mailgun) and publish that service's SPF and DKIM records for your domain.
- Stopgap only: an admin can turn SMTP AUTH back on for just the device's mailbox with `Set-CASMailbox -Identity scanner@yourdomain.com -SmtpClientAuthenticationDisabled $false`. It fails while security defaults are on and ends when Microsoft sets the final removal date (to be announced in the second half of 2027). See the timeline (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Authentication Method: Off
- Sender's Email Address: scanner@yourdomain.com (any address in your Microsoft 365 domain)
- SMTP Server Address: yourdomain-com.mail.protection.outlook.com (your MX host)
- SMTP Server Port Number: 25
- Secure Connection: STARTTLS
- Certificate Validation: Disable, or Enable after importing the CA certificate

**Sources:**
- Epson Europe FAQ KA-02047: OAuth2.0 support for Epson printers and scanners: https://www.epson.eu/en_EU/faq/KA-02047/contents
- Epson: Setting Up OAuth 2.0 Authentication (Web Config, DS-790WN User's Guide): https://files.support.epson.com/docid/cpd6/cpd61515/source/scanners/source/administration/tasks/setting_up_oauth2_webconfig.html
- Epson WF-3820 User's Guide: Email Server Settings: https://files.support.epson.com/docid/cpd5/cpd58812/source/printers/source/networking/reference/wf3820/email_server_settings_wf3820.html
- Epson: Mail Server Connection Report Messages: https://files.support.epson.com/docid/cpd6/cpd63622/source/printers/source/administration/references/email_connection_messages.html
- Epson Connect help: Scan to Cloud destination list: https://www.epsonconnect.com/help/en/html/scan_1.htm
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365 (updated 5 Oct 2026): https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission

Full page: https://mailpassed.com/m365-smtp/epson-workforce-scan-to-email-microsoft-365

## Toshiba e-STUDIO (e-BRIDGE Next) scan to email with Microsoft 365

e-BRIDGE Next Series II and III e-STUDIO models have Cloud Authentication for Email (OAuth) from firmware 2100 (Series II) or 1801 (Series III). In TopAccess, set the SMTP Client to Authentication: Application, assign Cloud Authentication for Email and sign in with the sending mailbox. Other models: relay connector.

**OAuth:** Toshiba calls its OAuth feature **Cloud Authentication for Email**. Toshiba's firmware table lists e-BRIDGE Next Series III models (e-STUDIO2020AC, 2525AC/3025AC/3525AC, 4525AC, 5525AC/6525AC, 6526AC/6527AC/7527AC, 2528A, 5528A/6528A, 7529A/9029A) from firmware **1801**, and Series II models (e-STUDIO2010AC, 2515AC/3015AC/3515AC/4515AC/5015AC, 5516AC/6516AC/7516AC, 2518A, 5518A/7518A/8518A, 330AC/400AC) from firmware **2100**. The SMTP Client still points at `smtp.office365.com` port 587, but the password field stays blank: the device signs in once through Microsoft's login page instead. That keeps it working after the Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement). Which Microsoft permissions Toshiba requests is not documented. Toshiba documents no OAuth option for e-BRIDGE Next Series I or older models, and Toshiba Tec's Japanese notice says models without OAuth support should move to another email service.

**Errors you may see:**
- Reported: Failed to connect. Check the following setting: Authentication, Login Name, Password. (Microsoft Q&A: e-STUDIO2515AC with Authentication: Login on smtp.office365.com port 587; that case was fixed by excluding the mailbox from a Conditional Access policy that blocks legacy sign-ins)
- "535 5.7.3 Authentication unsuccessful" or "5.7.57 Client not authenticated to send mail" (Microsoft Learn: SMTP AUTH login refused; check that Authenticated SMTP is on for the mailbox)
- Reported: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft Q&A; what a device logging in to smtp.office365.com with a password gets)
- The SMTP Client test (Execute) doesn't return Successfully Connected (Toshiba guide: recheck the settings)
- Scans reach internal mailboxes but never outside recipients (Microsoft Learn: the device uses direct send to the MX host without a relay connector)

**Steps:**
1. Open the SMTP Client in TopAccess: **Administration → Setup → Network → SMTP Client** (Toshiba's A3 guide: **Administration → Setup → Email → SMTP Client**). `smtp.office365.com` with a Login Name and password is Basic auth and fails once Microsoft turns it off by default at the end of December 2026. To find every device like this in your tenant, run the free SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders).
2. Check the model and firmware against Toshiba's table: Series III needs firmware 1801 or later, Series II needs 2100 or later. Update older firmware first.
3. Microsoft 365 admin center → **Settings → Org Settings → Modern authentication**: confirm **Turn on modern authentication for Outlook 2013 for Windows and later** is ticked (Toshiba says it is on by default). Keep **Authenticated SMTP** ticked for the sending mailbox under **Users → Active users → Mail → Manage email apps**: OAuth logins still use the SMTP AUTH protocol.
4. In TopAccess (log in as admin), set the SMTP Client: **Enable SMTP Client** Enable, **Enable SSL / TLS** Accept all certificates without CA, **SMTP Server Address** `smtp.office365.com`, **Authentication** Application, **Login Name** the mailbox address, **Password** blank, **Port Number** 587.
5. Click **Assignment**, select **Cloud Authentication for Email**, click OK, then **Save** and OK.
6. Open the **Cloud Authentication for Email** link. Type the same address under **Account Name**, tick **Send Email** and click **Account Registration**. Sign in with that mailbox on Microsoft's login page. On the permissions screen, tick **Consent on behalf of your organisation** if it applies, then click **Accept**.
7. Wait for **Access was approved. Please continue on the MFP panel.** Back in TopAccess, check that the account is registered to send email, then send a test scan.

**If it can't do OAuth:** 
- No OAuth firmware for your model (for example e-BRIDGE Next Series I)? Create a relay connector (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup) that trusts your office's static public IP.
- Then set the SMTP Client to Toshiba's direct send values: **SMTP Server Address** your MX host (for example `contoso-com.mail.protection.outlook.com`), **SSL/TLS** STARTTLS, **POP Before SMTP** Disable, **Authentication** Disable, Login Name and Password blank, **Port Number** 25. Click **Execute** and expect **Successfully Connected**.
- Without the connector this is direct send, which only reaches mailboxes inside your organization (Microsoft Learn). Toshiba also asks for an active, working mailbox address in **From Address** under **Administration → Setup → Email → General Setting**.
- Stopgap only: an admin can turn Basic auth for SMTP AUTH back on for now. It ends when Microsoft removes it for good; the date is due in the second half of 2027. See the SMTP AUTH change (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- OAuth feature: Cloud Authentication for Email (TopAccess)
- Minimum firmware: e-BRIDGE Next Series III: 1801. Series II: 2100. Series I and older: OAuth not documented
- TopAccess path: Administration > Setup > Network > SMTP Client (A3 guide: Administration > Setup > Email > SMTP Client)
- SMTP Client for OAuth: smtp.office365.com, port 587, Enable SSL/TLS: Accept all certificates without CA, Authentication: Application, Login Name: mailbox address, Password: blank
- Assignment: Cloud Authentication for Email, then Account Registration with Send Email ticked
- Microsoft 365 side: Modern authentication on (Org Settings); Authenticated SMTP on for the sending mailbox
- Microsoft permissions requested: Not documented
- Relay / direct send: MX host, port 25, SSL/TLS: STARTTLS, Authentication: Disable, no login

**Sources:**
- Toshiba (Australia): Cloud Authentication for E-mail, e-BRIDGE Next Series III (v3.0, May 2024; firmware table and setup): https://www.toshiba-business.com.au/-/media/files/toshiba/pdf/faqs/email-setup/scan-to-e-mail-using-microsoft365-modern-authentication.pdf
- Toshiba (New Zealand): Setup Microsoft 365 Account for Scanning to e-Mail, Toshiba A3 (direct send settings): https://www.toshiba-business.co.nz/-/media/files/toshiba/pdf/faqs/email-setup/toshiba-a3---scanning-to-e-mail-using-microsoft-365-basic-authentication.pdf
- Toshiba Tec (Japan): Impact of Exchange Online changes on our MFPs, 13 Jan 2026 (Japanese): https://www.toshibatec.co.jp/information/20260113_01.html
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Learn: Fix issues with printers, scanners, and LOB apps that send email using Microsoft 365: https://learn.microsoft.com/en-us/troubleshoot/exchange/email-delivery/fix-issues-with-printers-scanners-and-lob-applications-that-send-email-using-off
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication deprecation timeline (MC786329): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- Microsoft Q&A (user report): Scan-to-Email for Toshiba printer only works with one admin account in 365: https://learn.microsoft.com/en-us/answers/questions/5343684/scan-to-email-for-toshiba-printer-only-works-with
- Microsoft Q&A (user report): Invalid login: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled: https://learn.microsoft.com/en-us/answers/questions/4681098/invalid-login-535-5-7-139-authentication-unsuccess

Full page: https://mailpassed.com/m365-smtp/toshiba-e-studio-scan-to-email-microsoft-365

## FUJIFILM Apeos and ApeosPort (formerly Fuji Xerox) scan to email with Microsoft 365

Install FUJIFILM's countermeasure firmware, then set Service Provider to Exchange Online / Outlook.com under Network → Protocols → SMTP in Internet Services and sign in with the device mailbox (OAuth 2.0). Models FUJIFILM lists as Type (2) get no fix: use a relay connector or another mail service.

**OAuth:** FUJIFILM Business Innovation fixes the Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement) with firmware. After the update, the SMTP page in Internet Services (CentreWare Internet Services on older models) has a **Service Provider** setting. **Exchange Online / Outlook.com** signs in with OAuth 2.0: the device shows a code, you enter it on the sign-in page and sign in with the sending mailbox. A Global Administrator must consent once per tenant; FUJIFILM says the app reads users' basic information and accesses their email. Firmware depends on model and region, for example Apeos C3060/C2560/C2060 **21.50.7** (India, Europe, Middle East, North America, Africa) or Apeos 3560/3060/2560 **1.50.4** (Asia except India, and Oceania). Some models are still **Under planning**. Type (2) models (many ApeosPort-IV, ApeosPort-V and DocuCentre-IV/V devices) get no fix; FUJIFILM says to use an email service other than Exchange Online. Each device needs its own mailbox account (several devices may share one), and the From address is always that account.

**Errors you may see:**
- "027-779: Authentication failure with SMTP-AUTH" (FUJIFILM notice; in job records and error history)
- "016-781: Server connection error" (FUJIFILM notice)
- Scan or fax send fails with an error on the panel and an unsent report; the error history isn't shown on the panel, only in Internet Services (FUJIFILM notice)
- "SendAsDenied" after switching to OAuth (FUJIFILM setup guide: the From field is still editable or uses the logged-in user's address)
- Authentication error when sending after OAuth setup (FUJIFILM setup guide: the Device Email Address isn't the account's UPN, the sender was changed, or Login Credentials for Email Send went back to Remotely Authenticated User)
- Sign-in with the code fails (FUJIFILM setup guide: a Conditional Access policy such as Block Device Code Flow is blocking it)

**Steps:**
1. Find your model in FUJIFILM's product list for your region. Note the workaround type, the countermeasure firmware and whether it uses setup guide A or B. To list every device that still logs in with a password, run the free SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders).
2. Type (1): install the countermeasure firmware. Type (2), or firmware still **Under planning**: go to the fallback section.
3. Open the admin consent link from FUJIFILM's setup guide in a browser, sign in as a **Global Administrator** and click **Accept**. Do this once per tenant.
4. Lock the sender. Guide A: **Apps → Email → Common Settings → "From" Field**, disable **Use Email Address of Logged-In User** and set **Edit "From" Field** to **Disabled**. Guide B: **Properties → Services → Email → General → From Address → Edit**, set both **[From:]** edit options to **No**.
5. Log in to Internet Services as system administrator and open **Network → Protocols → SMTP** (guide B: **Connectivity → Protocols → SMTP Server → General**). If **Login Credentials for Email Send** is **Remotely Authenticated User**, change it to **System**. Set **Service Provider** to **Exchange Online / Outlook.com** and click **Save** (guide B: **Apply**). Leave the other fields as they are.
6. Click **Next** on **Enter Authentication Code** (guide B: **Open Authentication Page**), enter the code shown, then sign in with the sending mailbox. Keep **Authenticated SMTP** on for that mailbox in the Microsoft 365 admin center: OAuth logins still use SMTP AUTH.
7. The signed-in address becomes the **Device Email Address**. It must match the account's UPN; if the primary address differs, change it to the UPN under **Device Details**. Don't type an SMTP server on the control panel afterwards: that resets Service Provider to **Not Selected**. Send a test scan.

**If it can't do OAuth:** 
- Type (2) model or no firmware yet? Create a relay connector (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup) that trusts your office's static public IP and point the device's SMTP server at your MX host (for example `contoso-com.mail.protection.outlook.com`) on port 25 with no login. These are Microsoft's relay values; FUJIFILM doesn't document them and only says to use an email service other than Exchange Online.
- Microsoft lists relay for devices that can't sign in to a mailbox; it needs a static IP and doesn't work from a dynamic one.
- Stopgap only: an admin can turn Basic auth for SMTP AUTH back on for now. It ends when Microsoft removes it for good; the date is due in the second half of 2027. See the SMTP AUTH change (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Internet Services (guide A): Network > Protocols > SMTP > Service Provider: Exchange Online / Outlook.com
- CentreWare Internet Services (guide B): Properties > Connectivity > Protocols > SMTP Server > General > Service Provider
- Login Credentials for Email Send: System (not Remotely Authenticated User)
- From field (guide A): Apps > Email > Common Settings > "From" Field: Use Email Address of Logged-In User disabled, Edit "From" Field Disabled
- Admin consent: Global Administrator, once per tenant; app (client) ID c88859d7-e6a2-48cb-acee-669df3387654
- Device Email Address: Must equal the UPN of the signed-in account
- Firmware examples: Apeos C3060/C2560/C2060: 21.50.7. Apeos 3560/3060/2560: 21.50.7, or 1.50.4 in Asia (except India) and Oceania
- Error codes: 027-779 (SMTP-AUTH failure), 016-781 (server connection error)

**Sources:**
- FUJIFILM Business Innovation: notice on the end of Basic authentication for Exchange Online SMTP AUTH (products, firmware, error codes): https://www.fujifilm.com/fbglobal/eng/company/news/notice/2025/0307_announce.html
- FUJIFILM: Exchange Online/Outlook.com SMTP Server Settings, setup guide A (Internet Services): https://asset-fb.fujifilm.com/www/fb/files/2026-03/327743ee7316e4c33e96e6f8966f34e1/20260323_k04_settings_operation_smtp_e_a.pdf
- FUJIFILM: Exchange Online/Outlook.com SMTP Server Settings, setup guide B (CentreWare Internet Services): https://asset-fb.fujifilm.com/www/fb/files/2026-03/efbc05392795103f161c0c2e888ef609/20260323_k05_settings_operation_smtp_e_b.pdf
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication deprecation timeline (MC786329): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835

Full page: https://mailpassed.com/m365-smtp/fujifilm-apeos-scan-to-email-microsoft-365

## WordPress SMTP plugins (WP Mail SMTP, FluentSMTP, Post SMTP) with Microsoft 365

Move the plugin off Other SMTP (smtp.office365.com plus a password) to its Microsoft mailer: WP Mail SMTP 365 / Outlook (Pro), FluentSMTP Outlook (free) or Post SMTP Office 365 (Pro). They send through Microsoft Graph with OAuth, so the Basic auth switch-off doesn't touch them.

**OAuth:** All three plugins can send through Microsoft's API with OAuth 2.0, so WordPress stores no mailbox password and the SMTP AUTH Basic auth change (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement) doesn't affect them. WP Mail SMTP's **365 / Outlook** mailer uses the Microsoft Graph API and needs a Pro licence or higher; its docs name only the `mail.readwrite` scope (used for large attachments), the rest of its permission list is not documented. FluentSMTP's free **Outlook / Office 365** connection lists no permissions in its docs, but its source code calls Graph `/me/sendMail` and requests the delegated scopes `Mail.Send`, `Mail.Send.Shared` and `offline_access`. Post SMTP's **Office 365** mailer (Pro) authenticates against Microsoft's Graph API; its permission names are not documented. All three need a Microsoft Entra app registration, except WP Mail SMTP **One-Click Setup** (Pro) and Post SMTP **One-Click** (Business plan), where you only sign in with Microsoft.

**Errors you may see:**
- Reported: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (quoted in a WP Mail SMTP blog post for sites using Other SMTP with smtp.office365.com)
- "535 5.7.3 Authentication unsuccessful" (Microsoft Learn: the failed SMTP AUTH password login as many apps report it)
- "SMTP Error: MailboxNotEnabledForRESTAPI" (WP Mail SMTP docs: the mailbox can't use the API, for example an on-premises mailbox in a hybrid setup)
- "Error: InvalidAuthenticationToken: Token Expired" (WP Mail SMTP docs: remove the connection and authorise again)
- "SMTP Error: ErrorSendAsDenied" (WP Mail SMTP docs: the From Email isn't the mailbox you authorised)
- Error code 403 after Post SMTP One-Click sign-in (Post SMTP docs: you signed in with an account other than the From Email)

**Steps:**
1. Check which mailer the plugin uses. **Other SMTP** with `smtp.office365.com`, port 587 and a password is Basic auth and will fail once Microsoft switches it off at the end of December 2026.
2. Pick the plugin's Microsoft mailer: WP Mail SMTP **365 / Outlook** (Pro licence), FluentSMTP **Outlook / Office 365** (free), or Post SMTP **Office 365** (Pro; activate it first under **Post SMTP → Extensions**).
3. Fastest route: WP Mail SMTP **One-Click Setup** (Pro) or Post SMTP **One-Click** (Business plan). You sign in with Microsoft and skip the app registration.
4. Otherwise register an app: Azure portal or Microsoft Entra admin center → **App registrations → New registration**. All three guides use **Accounts in any organizational directory … and personal Microsoft accounts**. Set the Redirect URI platform to **Web** and paste the URI the plugin shows (FluentSMTP: `https://yourdomain.com/wp-json/fluent-smtp/outlook_callback`). Then **Certificates & secrets → New client secret** and copy the **Value**, not the Secret ID.
5. Paste the Application (client) ID and the secret Value into the plugin and click its connect button (WP Mail SMTP: **Allow plugin to send emails using your Microsoft account**; FluentSMTP: **Authenticate with Office365**; Post SMTP: **Connect to Office 365**). Accept Microsoft's permission prompt. If your tenant blocks user consent, a Global Administrator has to approve the app.
6. Sign in with the mailbox you send from and use that address as the From Email (in WP Mail SMTP, turn on **Force From Email**). Send the plugin's test email.
7. Write down the client secret's expiry date (WP Mail SMTP suggests 730 days, Post SMTP 24 months) and create a new secret before it runs out.

**If it can't do OAuth:** 
- Can't use Microsoft's API (for example **MailboxNotEnabledForRESTAPI** on a hybrid mailbox)? Switch the plugin to a third-party email service with its own login or API key, then add its SPF include with the SPF generator (https://mailpassed.com/spf-generator) and check the domain with our checker (https://mailpassed.com/).
- A relay connector is usually not an option for WordPress: Microsoft says SMTP relay can't be used from mail sent by a third-party hosted service (for example Azure), and most WordPress sites run on hosting like that.
- Stopgap only: an admin can turn SMTP AUTH back on for the site's mailbox with `Set-CASMailbox -Identity wordpress@yourdomain.com -SmtpClientAuthenticationDisabled $false`. It stops working when Microsoft removes Basic auth for good. Details: 535 5.7.139 (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth).

**Settings:**
- Microsoft mailer: WP Mail SMTP Pro: 365 / Outlook. FluentSMTP (free): Outlook / Office 365. Post SMTP Pro: Office 365
- How it sends: Microsoft Graph API with OAuth 2.0; no SMTP server, port or password
- Entra app: supported account types: Accounts in any organizational directory (multitenant) and personal Microsoft accounts; single tenant works if you give the plugin your tenant ID
- Entra app: redirect URI: Platform Web; copy the URI from the plugin settings
- Entra app: client secret: Copy the Value; WP Mail SMTP suggests 730 days, Post SMTP 24 months
- Permissions: FluentSMTP (source code): delegated Mail.Send, Mail.Send.Shared, offline_access. WP Mail SMTP: only mail.readwrite documented. Post SMTP: not documented
- No app registration needed: WP Mail SMTP One-Click Setup (Pro or higher); Post SMTP One-Click (Business plan)

**Sources:**
- WP Mail SMTP: How to Set Up the Microsoft 365 / Outlook.com Mailer: https://wpmailsmtp.com/docs/how-to-set-up-the-outlook-mailer-in-wp-mail-smtp/
- WP Mail SMTP: A Complete Guide to WP Mail SMTP Mailers (Outlook mailer uses Microsoft's Graph API, Pro): https://wpmailsmtp.com/docs/a-complete-guide-to-wp-mail-smtp-mailers/
- FluentSMTP: Configure FluentSMTP with Microsoft Outlook/Office email: https://docs.fluentsmtp.com/configure-fluentsmtp-with-microsoft-outlookoffice-email
- FluentSMTP source code: Outlook API provider (scopes and Graph sendMail endpoint): https://github.com/WPManageNinja/fluent-smtp/blob/master/app/Services/Mailer/Providers/Outlook/API.php
- Post SMTP: How to configure Post SMTP with Office 365: https://postmansmtp.com/docs/mailers/how-to-setup-office-365-with-post-smtp/
- Post SMTP: Microsoft Outlook / Office 365 One-Click mailer (Business plan): https://postmansmtp.com/docs/mailers/microsoft-365-one-click-smtp/
- Post SMTP: Microsoft Office 365 single-tenant code snippet (Graph API): https://postmansmtp.com/docs/code-snippets/microsoft-office-365/

Full page: https://mailpassed.com/m365-smtp/wordpress-smtp-microsoft-365

## Synology DSM and QNAP QTS / QuTS hero email notifications with Microsoft 365

Synology DSM and QNAP Notification Center both have an Outlook sign-in option (OAuth) for alert email. Use it instead of a custom SMTP server with smtp.office365.com and a password, which fails once Basic auth is off. A relay connector on port 25 is the fallback.

**OAuth:** **Synology:** in DSM the notification sender can use **Outlook** as the service provider with a **Sign In** button. Since DSM 7.1.1-42962 Update 2 (October 2022; also DSM 7.0.1-42218 Update 6 and DSM 6.2.4-25556 Update 7) the Outlook option requires OAuth 2.0, and Outlook senders set up before that update were moved to **Custom SMTP server**, which logs in with a password. Those NASes will hit the Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement). **QNAP:** Notification Center's **Outlook** provider signs in through a Microsoft window (**Add account**, then **Allow**) and has a **Re-authorization** button when access expires. Neither vendor says whether Microsoft 365 work or school accounts are supported (QNAP's FAQ calls the service Outlook.com), which Microsoft permissions are requested, or whether tenant admin consent is needed: not documented.

**Errors you may see:**
- Synology release notes: "Previously configured Outlook accounts will be migrated to "Custom SMTP server"." (DSM 7.1.1-42962 Update 2) - those senders log in with a password
- "Authorization has expired. Reauthorize the account." (QNAP Notification Center warning)
- Reported: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft Q&A; what a device logging in to smtp.office365.com with a password gets)
- "535 5.7.3 Authentication unsuccessful" or "5.7.57 Client not authenticated to send mail" (Microsoft Learn)
- The NAS test email fails while the same mailbox still works in Outlook.

**Steps:**
1. See how the NAS sends today. Synology: **Control Panel → Notification → Email**. QNAP: **Notification Center → Service Account and Device Pairing → E-mail**. A custom SMTP entry with `smtp.office365.com` and a password is Basic auth and will break.
2. Synology: set the sender up again with **Service provider: Outlook**, click **Sign In** and sign in with the Microsoft 365 mailbox that should send the alerts. Do this even if the NAS once used Outlook: older Outlook senders were moved to Custom SMTP server.
3. QNAP: click **Add SMTP Service**, choose **Outlook**, click **Add account**, sign in with the sending mailbox and click **Allow**. Tick **Set as default SMTP service account** (system notifications use the default), then **Create**. Remove the old Custom entry and add the new one to your notification rules.
4. Send a test email from the same screen on both systems.
5. If Microsoft's sign-in page says the app needs admin approval, a Microsoft 365 admin has to grant it. Synology and QNAP don't document this step.
6. QNAP only: when you see **Authorization has expired**, edit the SMTP service and click **Re-authorization**, or remove the service and add it again.

**If it can't do OAuth:** 
- Create an SMTP relay connector in the Exchange admin center (**Mail flow → Connectors**, connection from **Your organization's email server** to **Office 365**) that trusts your office's static public IP.
- Synology: **Service provider: Custom SMTP server**, SMTP server = your MX host (for example `contoso-com.mail.protection.outlook.com`), port 25, untick **Authentication required**, sender = any address in your domain.
- QNAP: choose **Custom** with the MX host and port 25. QNAP's form shows username and password fields and doesn't say they can be left empty: not documented. If it won't save without them, use the Outlook option or a third-party SMTP service instead.
- Add the office IP to your SPF record with the SPF generator (https://mailpassed.com/spf-generator), then check the domain with our checker (https://mailpassed.com/).
- Stopgap only: an admin can turn SMTP AUTH back on for the NAS mailbox (`Set-CASMailbox -Identity nas@yourdomain.com -SmtpClientAuthenticationDisabled $false`) until Microsoft removes Basic auth for good. See 535 5.7.139 (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth).

**Settings:**
- Synology menu: Control Panel > Notification > Email
- Synology OAuth option: Service provider: Outlook, then Sign In (OAuth 2.0)
- QNAP menu: Notification Center > Service Account and Device Pairing > E-mail > Add SMTP Service
- QNAP OAuth option: Outlook: Add account, sign in, Allow; Re-authorization when it expires
- Relay: SMTP server: Your MX host, e.g. contoso-com.mail.protection.outlook.com
- Relay: port and login: Port 25, no username or password (Synology: untick Authentication required; QNAP: empty login not documented)
- Relay: encryption: TLS on if the NAS supports it; Microsoft lists TLS as optional for relay. Whether DSM or QTS use STARTTLS on port 25: not documented

**Sources:**
- Synology Knowledge Center: DSM Email notification settings: https://kb.synology.com/en-global/DSM/help/DSM/AdminCenter/system_notification_email?version=7
- Synology: DSM release notes (DSM 7.1.1-42962 Update 2, Outlook OAuth 2.0 and migration to Custom SMTP server): https://www.synology.com/en-global/releaseNote/DSM
- QNAP QTS 5.2 User Guide: Configuring an email notification server: https://docs.qnap.com/operating-system/qts/5.2.x/en-us/configuring-an-email-notification-server-EB4E6D7F.html
- QNAP QTS 5.2 User Guide: Email notifications (Re-authorization): https://docs.qnap.com/operating-system/qts/5.2.x/en-us/email-notifications-B5C2D62A.html
- QNAP FAQ: "Authorization has expired. Reauthorize the account" in Notification Center: https://www.qnap.com/en/how-to/faq/article/how-do-i-resolve-the-warning-authorization-has-expired-reauthorize-the-account-in-notification-center
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365

Full page: https://mailpassed.com/m365-smtp/synology-qnap-nas-email-microsoft-365

## PowerShell Send-MailMessage with Microsoft 365

Send-MailMessage is obsolete (Microsoft Learn) and logs in to smtp.office365.com with a password, which fails once Basic auth is off. Replace it with Send-MgUserMail from the Microsoft Graph PowerShell SDK, using only the Mail.Send permission, or send through a relay connector.

**OAuth:** Send-MailMessage has no OAuth option: its only login is `-Credential`, a username and password, so it stops working with Microsoft 365 when the Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement) reaches your tenant. Microsoft Learn calls the cmdlet obsolete, says it doesn't guarantee secure connections, and points Exchange Online users to **Send-MgUserMail** from the Microsoft Graph PowerShell SDK (its DE0005 note suggests MailKit for other servers). Send-MgUserMail needs only `Mail.Send`: delegated for a script run by a signed-in user, application for unattended jobs. The application permission lets the app send as any user, so limit it to one mailbox with Exchange **RBAC for Applications**, which replaces Application Access Policies.

**Errors you may see:**
- "535 5.7.3 Authentication unsuccessful" (Microsoft Learn: Send-MailMessage -Credential against smtp.office365.com)
- Reported: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft Q&A)
- "5.7.60 SMTP; Client does not have permissions to send as this sender" (Microsoft Learn: -From is not the -Credential mailbox)
- "The Send-MailMessage cmdlet is obsolete." (Microsoft Learn, Send-MailMessage reference)

**Steps:**
1. Install the SDK (Microsoft Learn): `Install-Module Microsoft.Graph -Scope CurrentUser -Repository PSGallery -Force`. To keep it small, install just `Microsoft.Graph.Authentication` and `Microsoft.Graph.Users.Actions`.
2. Script run by a person: connect with only the send permission, `Connect-MgGraph -Scopes "Mail.Send"` (Microsoft Learn's interactive pattern, with Mail.Send as the scope).
3. Unattended script: register an app in Microsoft Entra with a certificate, then connect (Microsoft Learn): `Connect-MgGraph -ClientId "YOUR_APP_ID" -TenantId "YOUR_TENANT_ID" -CertificateThumbprint "YOUR_CERT_THUMBPRINT"`.
4. Give the app the send right for one mailbox only. In Exchange Online PowerShell run `New-ServicePrincipal`, then `New-ManagementRoleAssignment -App  -Role "Application Mail.Send" -CustomResourceScope ` (syntax from Microsoft Learn). Granting Mail.Send in Entra instead covers every mailbox in the tenant, and Microsoft says to remove that grant if you use the scoped role.
5. Send (Microsoft Learn example 1, shortened): `Import-Module Microsoft.Graph.Users.Actions`, then `$params = @{ message = @{ subject = "Meet for lunch?"; body = @{ contentType = "Text"; content = "The new cafeteria is open." }; toRecipients = @(@{ emailAddress = @{ address = "frannis@contoso.com" } }) } }` and `Send-MgUserMail -UserId $userId -BodyParameter $params`.
6. Graph answers **202 Accepted**: the message was accepted, not yet delivered. It is saved to Sent Items unless you set `saveToSentItems` to false.

**If it can't do OAuth:** 
- No Graph possible? Send through an SMTP relay connector from a server with a static public IP (Exchange admin center → **Mail flow → Connectors**, from **Your organization's email server** to **Office 365**). Relay needs no username or password.
- Point the script at your MX host on port 25 with TLS. Our example, not from Microsoft: `Send-MailMessage -SmtpServer contoso-com.mail.protection.outlook.com -Port 25 -UseSsl` without `-Credential`. Treat it as a bridge only, since Microsoft recommends not using Send-MailMessage.
- Add the server's IP to SPF with the SPF generator (https://mailpassed.com/spf-generator). Relay doesn't work from Azure or other third-party clouds; use Graph there.
- Stopgap only: `Set-CASMailbox -Identity script@yourdomain.com -SmtpClientAuthenticationDisabled $false` keeps the old `-Credential` login working until Microsoft removes Basic auth for good. See 535 5.7.139 (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth).

**Settings:**
- Replacement cmdlet: Send-MgUserMail (module Microsoft.Graph.Users.Actions)
- Install: Install-Module Microsoft.Graph -Scope CurrentUser -Repository PSGallery -Force
- Permission (least privileged): Mail.Send: delegated for a signed-in user, application for unattended scripts
- Graph endpoint: POST /users/{id | userPrincipalName}/sendMail
- Success response: 202 Accepted (accepted, not yet delivered)
- Limit app-only sending: Exchange RBAC for Applications role "Application Mail.Send" with a management scope
- Relay fallback: MX host, port 25, TLS, connector trusts a static IP or certificate, no credentials

**Sources:**
- Microsoft Learn: Send-MailMessage (obsolete; Send-MgUserMail for Exchange Online): https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/send-mailmessage
- Microsoft Learn: Send-MgUserMail (Microsoft.Graph.Users.Actions): https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.users.actions/send-mgusermail
- Microsoft Learn: user: sendMail, Microsoft Graph v1.0 (permissions, 202 Accepted): https://learn.microsoft.com/en-us/graph/api/user-sendmail?view=graph-rest-1.0
- Microsoft Learn: Use Microsoft Graph PowerShell authentication commands: https://learn.microsoft.com/en-us/powershell/microsoftgraph/authentication-commands?view=graph-powershell-1.0
- Microsoft Learn: Install the Microsoft Graph PowerShell SDK: https://learn.microsoft.com/en-us/powershell/microsoftgraph/installation?view=graph-powershell-1.0
- Microsoft Learn: Role Based Access Control for Applications in Exchange Online: https://learn.microsoft.com/en-us/exchange/permissions-exo/application-rbac
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365 (SMTP relay): https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365

Full page: https://mailpassed.com/m365-smtp/powershell-send-mailmessage-microsoft-365

## Veeam Backup & Replication email notifications with Microsoft 365

Switch the Veeam mail server from SMTP server (smtp.office365.com plus a password) to Microsoft 365 OAuth 2.0: Options → Email Settings, Mail server Microsoft 365, Authorize now. Veeam then signs in with a token, so the Basic auth switch-off doesn't affect it.

**OAuth:** Veeam Backup & Replication supports three mail server methods: **SMTP basic authentication**, **Google Gmail OAuth 2.0** and **Microsoft 365 OAuth 2.0**. The v12 guide (build 12.3.2) and the v13 guide both document the Microsoft 365 option. Only SMTP basic authentication logs in with a password, so only it is hit by the SMTP AUTH Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement). With Microsoft 365 you sign in once with an Exchange Online account, Veeam shows **Token is valid** and refreshes the token automatically. You can use Veeam's default app or your own Microsoft Entra app; the v13 web UI asks for your own app's client ID, tenant ID and client secret. Veeam's guide for your own app grants the Microsoft Graph **application** permission `Mail.Send` with admin consent, so that Veeam can call the Microsoft Graph API to send notifications. Microsoft Learn says this permission lets an app send mail as any user. Which permissions Veeam's default app requests: not documented.

**Errors you may see:**
- Reported: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft Q&A; what a password login to smtp.office365.com gets once Basic auth is off)
- "535 5.7.3 Authentication unsuccessful" (Microsoft Learn: sample of a failed SMTP AUTH login)
- "AADSTS90094 AdminConsentRequired - Administrator consent is required." (Microsoft Entra error reference: the app has no admin consent in your tenant yet)
- "AADSTS50011 InvalidReplyTo - The reply address is missing, misconfigured, or doesn't match reply addresses configured for the app." (Microsoft Entra error reference: your own app lacks the redirect URIs from Veeam's guide)
- "AADSTS7000222 InvalidClientSecretExpiredKeysProvided - The provided client secret keys are expired." (Microsoft Entra error reference: your own app's client secret ran out)

**Steps:**
1. Check how Veeam sends today: main menu → **Options** → **Email Settings** (v13 web UI: **Configuration** → **Email Settings**). **Mail server: SMTP server** with `smtp.office365.com` and **This SMTP server requires authentication** ticked is Basic auth. Microsoft turns Basic auth off by default at the end of December 2026. Our free, read-only SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders) lists every mailbox that still sends this way, including the one Veeam uses.
2. Use a dedicated mailbox for alerts: Veeam recommends a dedicated service account with granular SendMail permissions. On the Veeam server, turn off **Internet Explorer Enhanced Security Configuration** in Server Manager; Veeam says this is needed to sign in with Exchange Online credentials.
3. Console: in **Mail server** pick **Microsoft 365**, click **Authorize now** and sign in with the alert mailbox. Wait for the **Token is valid** notice.
4. v13 web UI: click **Configure** next to **Mail server**, choose **Microsoft 365 (modern authentication)**, enter the application client ID, tenant ID and client secret, then click **Authorize** and sign in.
5. Your own app (needed for the web UI; optional in the console under **Advanced → Use custom application registration settings**, which takes the client ID and tenant ID): Microsoft Entra ID → **App registrations → New registration**, **Accounts in this organizational directory only**. Under **Authentication** add **Mobile and desktop applications** with the redirect URIs `msal://auth`, `ms-appx-web://microsoft.aad.brokerplugin/` and `http://localhost`. Under **API permissions** add Microsoft Graph → **Application permissions** → `Mail.Send` and click **Grant admin consent**. When you authorize, tick **Consent on behalf of your organization**. How to create the client secret the web UI asks for: not documented in Veeam's guide.
6. Set **From** to the mailbox you signed in with (Veeam requires this for OAuth), then click **Test Message**.
7. If the token is revoked or the mailbox password changes, click **Re-authorize**.

**If it can't do OAuth:** 
- No Microsoft 365 option in your Veeam version? Create an SMTP relay connector (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup) that trusts the Veeam server's static public IP. Then set **Mail server: SMTP server** to your MX host (for example `contoso-com.mail.protection.outlook.com`) and under **Advanced** use port 25, untick **This SMTP server requires authentication** and tick **Connect using SSL**. Port 465 (implicit TLS) is not supported.
- HA cluster: Veeam sends from the IP address of the current primary node, not the cluster IP. Add both node IPs to the connector.
- Stopgap only: a password login needs SMTP AUTH on for the mailbox (`Set-CASMailbox -Identity veeam@yourdomain.com -SmtpClientAuthenticationDisabled $false`) and Basic auth still allowed. After the end of December 2026 admins can turn Basic auth back on until a final removal date, which Microsoft will announce in the second half of 2027. Details: Basic auth retirement (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Menu (console): Main menu > Options > Email Settings > Enable email notifications
- Menu (v13 web UI): Configuration > Email Settings > Configure next to Mail server
- Mail server (OAuth): Console: Microsoft 365, then Authorize now. Web UI: Microsoft 365 (modern authentication), then Authorize
- From address: The mailbox you signed in with
- Own app: account type: Accounts in this organizational directory only
- Own app: redirect URIs: Mobile and desktop applications: msal://auth, ms-appx-web://microsoft.aad.brokerplugin/, http://localhost
- Own app: permission: Microsoft Graph application permission Mail.Send, admin consent granted
- Own app: values in Veeam: Console: client ID and tenant ID. v13 web UI: client ID, tenant ID and client secret
- Default app permissions: Not documented
- Relay fallback: SMTP server = MX host, port 25, no authentication, Connect using SSL; port 465 not supported

**Sources:**
- Veeam Help Center (v13): Configuring Global Email Notification Settings Using Console: https://helpcenter.veeam.com/docs/vbr/userguide/general_email_notifications_console.html
- Veeam Help Center (v13): Configuring Global Email Notification Settings Using Web UI: https://helpcenter.veeam.com/docs/vbr/userguide/general_email_notifications_web.html
- Veeam Help Center (v13): Registering Application in Microsoft Azure Portal: https://helpcenter.veeam.com/docs/vbr/userguide/registering_azure_app.html
- Veeam Help Center (v12, archived, build 12.3.2): Configuring Global Email Notification Settings: https://helpcenter.veeam.com/archive/backup/120/vsphere/general_email_notifications.html
- Microsoft Learn: Role Based Access Control for Applications in Exchange Online (Mail.Send sends as any user): https://learn.microsoft.com/en-us/exchange/permissions-exo/application-rbac
- Microsoft Learn: Microsoft Entra authentication and authorization error codes: https://learn.microsoft.com/en-us/entra/identity-platform/reference-error-codes
- Microsoft Learn: Authenticate an IMAP, POP or SMTP connection using OAuth (535 5.7.3 sample): https://learn.microsoft.com/en-us/exchange/client-developer/legacy-protocols/how-to-authenticate-an-imap-pop-smtp-application-by-using-oauth
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365 (SMTP relay): https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication Deprecation Timeline (27 Jan 2026): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- Microsoft Q&A: 535 5.7.139 Authentication unsuccessful (reported error text): https://learn.microsoft.com/en-us/answers/questions/512954/535-5-7-139-authentication-unsuccessful

Full page: https://mailpassed.com/m365-smtp/veeam-backup-replication-email-microsoft-365

## Zabbix email media type with Microsoft 365

Zabbix 7.4 can sign in to Microsoft 365 with OAuth: set the email media type to Email provider Office365 and Authentication OAuth, then enter a client ID and secret. Zabbix 7.0 has no OAuth option, so use the Office365 relay provider with a relay connector, or upgrade.

**OAuth:** Zabbix added OAuth 2.0 for SMTP in 7.4. In 7.0 the **Authentication** field offers only **None** and **Username and password**, so a 7.0 media type that logs in to Office365 with a password breaks when the Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement) reaches your tenant. With OAuth, Zabbix still uses SMTP AUTH, but with a token: its Office365 defaults request `https://outlook.office.com/SMTP.Send` and `offline_access` through the authorization code grant at `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`. You enter only the Redirection endpoint, Client ID and Client secret. Zabbix says SmtpClientAuthentication must be enabled for the mailbox or the tenant. OAuth isn't supported for the **Office365 relay** provider. How to set up the Microsoft Entra app (account type, platform, permission type): not documented by Zabbix.

**Errors you may see:**
- Reported: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft Q&A; what a password login to Microsoft 365 SMTP gets once Basic auth is off)
- "535 5.7.3 Authentication unsuccessful" (Microsoft Learn: sample of a failed SMTP AUTH login)
- "AADSTS50011 InvalidReplyTo - The reply address is missing, misconfigured, or doesn't match reply addresses configured for the app." (Microsoft Entra error reference: the Zabbix Redirection endpoint isn't registered on the app)
- "Application '{appId}'({appName}) isn't configured as a multitenant application. Usage of the /common endpoint isn't supported for such applications created after '{time}'." (Microsoft Entra error AADSTS50194; Zabbix's Office365 defaults use the /common endpoint)
- "AADSTS7000222 InvalidClientSecretExpiredKeysProvided - The provided client secret keys are expired." (Microsoft Entra error reference: the app's client secret ran out)

**Steps:**
1. Check your version and the media type: **Alerts → Media types → Email**. Zabbix 7.0 can only log in with **Username and password**, which fails once Microsoft turns Basic auth off by default at the end of December 2026. Our free, read-only SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders) lists every SMTP AUTH sender in your tenant, so you can find the mailbox Zabbix uses.
2. Keep SMTP AUTH on for that mailbox, because OAuth still goes through SMTP AUTH: `Set-CASMailbox -Identity zabbix@yourdomain.com -SmtpClientAuthenticationDisabled $false`.
3. Register an app in Microsoft Entra (**App registrations → New registration**). Add the Zabbix Redirection endpoint, `https:///zabbix.php?action=oauth.authorize`, as a redirect URI and create a client secret. Give it the `SMTP.Send` permission that Microsoft Learn lists for SMTP AUTH. Zabbix's defaults use the `/common` endpoint, which Microsoft rejects for single-tenant apps (AADSTS50194), so register the app as multitenant. Which redirect platform type to choose: not documented by Zabbix.
4. Edit the media type: **Email provider: Office365**, **Email** = the sending mailbox, **Authentication: OAuth**. Next to **OAuth tokens** click **Configure**, check the Redirection endpoint and enter the Client ID and Client secret. Zabbix fills in the other values.
5. Submit. A browser popup opens Microsoft's sign-in page; sign in as the sending mailbox and accept. Zabbix exchanges the code for access and refresh tokens. If that fails, set **Authorization code** to **Manual** and paste the code.
6. Single-tenant app only (our suggestion, not covered by Zabbix): Zabbix says the **Generic SMTP** form can also set up OAuth for Office365. There you enter the endpoints yourself, so you can put your tenant ID in place of `common`, as Microsoft's AADSTS50194 message suggests.
7. Click **Test** in the media type list and send a test message. Note that Zabbix exports the client secret in clear text when you export the media type.

**If it can't do OAuth:** 
- Zabbix 7.0, or OAuth not allowed in your tenant? Use the **Office365 relay** provider with an SMTP relay connector (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup). It needs no password: Zabbix builds the server name from the sender's domain (`example-com.mail.protection.outlook.com`). The connector must trust the Zabbix server's static public IP or a certificate.
- Zabbix server in Azure or another third-party cloud? Microsoft says relay can't be used from there. Upgrade to 7.4 and use OAuth.
- Stopgap only: a password login needs SMTP AUTH on for the mailbox and Basic auth still allowed. After the end of December 2026 admins can turn Basic auth back on until a final removal date, which Microsoft will announce in the second half of 2027. Details: Basic auth retirement (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Menu: Alerts > Media types > Email (or Create media type)
- Email provider: Office365 (password or OAuth) or Office365 relay (no password, no OAuth)
- Authentication: Zabbix 7.4: None, Username and password, OAuth. Zabbix 7.0: None, Username and password
- Redirection endpoint: https:///zabbix.php?action=oauth.authorize
- Office365 OAuth defaults: Authorize: https://login.microsoftonline.com/common/oauth2/v2.0/authorize; token: https://login.microsoftonline.com/common/oauth2/v2.0/token; scope: https://outlook.office.com/SMTP.Send, offline_access; grant: authorization_code
- SMTP server, port, security: Filled in by Zabbix and hidden; shown in the Details column of the media type list
- Mailbox requirement: SMTP AUTH (SmtpClientAuthentication) enabled for the mailbox or the tenant
- Office365 relay server: .mail.protection.outlook.com, built from the sender address
- Entra app registration details: Not documented by Zabbix

**Sources:**
- Zabbix 7.4 manual: Email media type: https://www.zabbix.com/documentation/7.4/en/manual/config/notifications/media/email
- Zabbix 7.4 manual: Automated Gmail/Office365 media types (OAuth defaults, SmtpClientAuthentication): https://www.zabbix.com/documentation/7.4/en/manual/config/notifications/media/email/gmail_office
- Zabbix 7.4 manual: What's new in Zabbix 7.4.0 (OAuth 2.0 for SMTP): https://www.zabbix.com/documentation/7.4/en/manual/introduction/whatsnew740
- Zabbix 7.0 manual: Email media type (no OAuth): https://www.zabbix.com/documentation/7.0/en/manual/config/notifications/media/email
- Microsoft Learn: Authenticate an IMAP, POP or SMTP connection using OAuth (SMTP.Send scope, 535 5.7.3 sample): https://learn.microsoft.com/en-us/exchange/client-developer/legacy-protocols/how-to-authenticate-an-imap-pop-smtp-application-by-using-oauth
- Microsoft Learn: Microsoft Entra authentication and authorization error codes: https://learn.microsoft.com/en-us/entra/identity-platform/reference-error-codes
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365 (relay, Azure limit): https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication Deprecation Timeline (27 Jan 2026): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- Microsoft Q&A: 535 5.7.139 Authentication unsuccessful (reported error text): https://learn.microsoft.com/en-us/answers/questions/512954/535-5-7-139-authentication-unsuccessful

Full page: https://mailpassed.com/m365-smtp/zabbix-email-media-type-microsoft-365

## Paessler PRTG Network Monitor email notifications with Microsoft 365

PRTG has no OAuth option for SMTP, so a relay login to smtp.office365.com with a password stops working once Basic auth is off. Point Notification Delivery at a Microsoft 365 relay connector (MX host, port 25, no login), Azure Communication Services or High Volume Email.

**OAuth:** PRTG's **SMTP Relay Authentication** offers **Use no authentication (default)**, **Use standard SMTP authentication** and **Use SASL authentication**. The PRTG manual doesn't mention OAuth. Paessler's own Office 365 guide uses `smtp.office365.com`, port 587 and a mailbox username and password, which is Basic auth and is hit by the SMTP AUTH Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement). On Paessler's feedback portal the OAuth idea is marked Delivered, but Paessler's reply (December 2025) points to a knowledge base article with alternatives and names no release with OAuth. That knowledge base suggests High Volume Email for Microsoft 365 or an SMTP relay through Azure Communication Services. PRTG's relay fields (server, port, no authentication, TLS) also fit a Microsoft 365 relay connector.

**Errors you may see:**
- Reported: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft Q&A; what a password login to smtp.office365.com gets once Basic auth is off)
- "535 5.7.3 Authentication unsuccessful" (Microsoft Learn: failed SMTP AUTH login; check that SMTP AUTH is enabled on the mailbox)
- "Client was not authenticated to send anonymous mail during MAIL FROM" (Microsoft Learn: no login set while the server is smtp.office365.com; relay must use your MX host)
- "5.7.64 TenantAttribution; Relay Access Denied" (Microsoft Learn: after the switch to relay, no connector matches the PRTG server's IP or certificate)

**Steps:**
1. Open **Setup → System Administration → Notification Delivery**. If **SMTP Delivery** uses relay server `smtp.office365.com` with **Use standard SMTP authentication**, PRTG logs in with a password and will fail once Microsoft turns Basic auth off by default at the end of December 2026. Our free, read-only SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders) lists every SMTP AUTH sender in your tenant, including PRTG's mailbox.
2. Pick a route. Alerts to anyone, including outside addresses: relay connector or Azure Communication Services. Alerts only to people in your tenant: High Volume Email also works.
3. Relay connector: create it for the PRTG core server's static public IP (step by step (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup)). In PRTG set **Delivery Mechanism** to **Use one SMTP relay server (recommended in LANs/NATs)**, **SMTP Relay Server** to your MX host (for example `contoso-com.mail.protection.outlook.com`), **SMTP Relay Port** to 25, **SMTP Relay Authentication** to **Use no authentication (default)**, **Connection Security** to **Use SSL/TLS if the server supports it (default)** and **SSL/TLS Method** to **TLS 1.2**.
4. Azure Communication Services (Paessler's guide, which it says it tested): create a Communication Services resource and an Email Communication Services resource with your verified domain, an Entra app with a client secret, and a custom role assigned to that app. In PRTG use **SMTP Relay Server** `smtp.azurecomm.net`, port 587, **Use standard SMTP authentication**, user name `..` and the client secret as password.
5. High Volume Email: in the Exchange admin center go to **Mail flow → High Volume Email → Add an HVE account** and assign a pay-as-you-go billing policy (without one the account can't send). In PRTG use `smtp.hve.mx.microsoft`, port 587, **Use standard SMTP authentication** with the HVE account address and password. HVE delivers only to recipients in your tenant. If Security Defaults is on, HVE accepts only OAuth, which PRTG can't do.
6. Test: **Setup → Account Settings → Notification Templates**, open an email notification that includes you as a receiver (for example **Email to Admin**) and click **Test SMTP Settings**.

**If it can't do OAuth:** 
- No static IP, or port 25 blocked? Use Azure Communication Services as above; it signs in on port 587 with the Entra app's secret.
- PRTG core server in Azure or another third-party cloud? Microsoft says a relay connector can't be used from there. Microsoft's own pointers are High Volume Email for internal recipients and Azure Communication Services Email for internal and external recipients.
- PRTG Hosted Monitor: the relay server, port and HELO fields aren't available there, so this page applies to PRTG Network Monitor installed on your own server.
- Stopgap only: a password login needs SMTP AUTH on for the mailbox (`Set-CASMailbox -Identity prtg@yourdomain.com -SmtpClientAuthenticationDisabled $false`) and Basic auth still allowed. After the end of December 2026 admins can turn Basic auth back on until a final removal date, which Microsoft will announce in the second half of 2027. Details: Basic auth retirement (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Menu: Setup > System Administration > Notification Delivery > SMTP Delivery
- OAuth for SMTP: Not available (the PRTG manual lists no authentication, standard SMTP and SASL only)
- Delivery Mechanism: Use one SMTP relay server (recommended in LANs/NATs)
- Relay connector: Server: your MX host; port 25; Use no authentication (default); Use SSL/TLS if the server supports it (default); TLS 1.2
- Azure Communication Services: smtp.azurecomm.net; port 587; Use standard SMTP authentication; user ..; password = Entra client secret
- High Volume Email: smtp.hve.mx.microsoft; port 587; HVE account address and password; internal recipients only; billed per recipient
- Test: Setup > Account Settings > Notification Templates > Test SMTP Settings

**Sources:**
- Paessler PRTG Manual: Notification Delivery: https://www.paessler.com/manuals/prtg/notification_delivery_settings
- Paessler KB: Can I use Microsoft Office 365 as SMTP relay server for PRTG email delivery? (PRTG 25): https://helpdesk.paessler.com/en/support/solutions/articles/76000045334
- Paessler KB: How can I use SMTP without Microsoft Exchange Online basic authentication? (Azure Communication Services, PRTG 25): https://helpdesk.paessler.com/en/support/solutions/articles/76000085488
- Paessler feedback portal: OAuth for PRTG SMTP E-Mail Delivery (status and Paessler reply): https://uservoice.paessler.com/forums/966003-notifications/suggestions/50095731-oauth-for-prtg-smtp-e-mail-delivery
- Microsoft Learn: Manage High Volume Email for Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/high-volume-mails-m365
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Fix issues with printers, scanners, and LOB apps that send email using Microsoft 365: https://learn.microsoft.com/en-us/troubleshoot/exchange/email-delivery/fix-issues-with-printers-scanners-and-lob-applications-that-send-email-using-off
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication Deprecation Timeline (27 Jan 2026): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- Microsoft Q&A: 535 5.7.139 Authentication unsuccessful (reported error text): https://learn.microsoft.com/en-us/answers/questions/512954/535-5-7-139-authentication-unsuccessful

Full page: https://mailpassed.com/m365-smtp/prtg-email-notifications-microsoft-365

## Windows Server IIS 6.0 SMTP relay forwarding app mail to Microsoft 365

An IIS 6.0 SMTP server that forwards to smtp.office365.com with Basic authentication fails once Basic auth is off, and IIS has no OAuth option. Microsoft doesn't support IIS SMTP for relaying to Microsoft 365: move to Exchange Server or Azure Communication Services, or point each app at a relay connector.

**OAuth:** The IIS 6.0 SMTP virtual server has three outbound logins: **Anonymous access**, **Basic authentication** (plaintext) and **Integrated Windows Authentication**. There is no OAuth, so a smart host set to `smtp.office365.com` with Basic authentication breaks when the SMTP AUTH Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement) reaches your tenant. Microsoft Learn says Windows Server 2022 is the last version with the SMTP server feature (in IIS 6.0), that relaying from it to Microsoft 365 is unsupported, and to use a supported Exchange Server or Azure Communication Services instead. Windows Server 2025 removed the SMTP Server feature and the IIS 6 Management Console. For apps that send mail, Microsoft lists four methods: client SMTP submission (with OAuth), SMTP relay through a connector, Direct Send and High Volume Email.

**Errors you may see:**
- Reported: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft Q&A; what a Basic login to smtp.office365.com gets once Basic auth is off)
- "535 5.7.3 Authentication unsuccessful" (Microsoft Learn: failed SMTP AUTH login)
- "Client was not authenticated to send anonymous mail during MAIL FROM" (Microsoft Learn: anonymous delivery to smtp.office365.com instead of your MX host)
- "5.7.64 TenantAttribution; Relay Access Denied" (Microsoft Learn: no inbound connector matches the server's IP or certificate)
- "Because SMTP Server features were removed from Windows Server 2025, there's no replacement within the operating system." (Microsoft Learn: you can't add the feature to a new Windows Server 2025 relay)

**Steps:**
1. Check the virtual server in IIS 6.0 Manager: SMTP virtual server → **Properties → Delivery**. **Advanced → Smart host** = `smtp.office365.com` with **Outbound Security → Basic authentication** means it logs in with a password, which fails once Microsoft turns Basic auth off by default at the end of December 2026. Our free, read-only SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders) lists every SMTP AUTH sender in your tenant, so you can find the mailbox this server uses.
2. List what sends through this server (apps, scripts, scanners). Each one needs a new route; pick it per sender.
3. Microsoft's supported replacement for a local relay: a supported Exchange Server with anonymous relay (not open relay), or Azure Communication Services Email.
4. Apps on your network that can't do OAuth: send straight to Microsoft 365 through an SMTP relay connector (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup). Settings: your MX host (for example `contoso-com.mail.protection.outlook.com`), port 25, TLS 1.2 or 1.3, no login, any From address in an accepted domain. The connector trusts a certificate or a static public IP.
5. Apps that support OAuth: client SMTP submission to `smtp.office365.com`, port 587, TLS 1.2 or 1.3, with `SMTP.Send` (user sign-in) or `SMTP.SendAsApp` (app-only). SMTP AUTH must stay enabled on that mailbox.
6. Mail only to your own staff: High Volume Email (`smtp.hve.mx.microsoft`, port 587, HVE account or OAuth) or Direct Send (MX host, port 25, no login). Both reach internal recipients only. Microsoft says most customers don't need Direct Send and it is working on an option to turn Direct Send off by default.
7. Short-term bridge only, unsupported by Microsoft: keep IIS but drop the password. Create an IP-based relay connector for the server's static public IP, set **Advanced → Smart host** to your MX host, **Outbound connections → TCP port** to 25, and **Outbound Security** to **Anonymous access** with **TLS encryption** ticked. Whether IIS 6.0 can present a certificate for a certificate-based connector: not documented.

**If it can't do OAuth:** 
- Port 25 blocked or no static IP? Relay and Direct Send need port 25 and, for relay, a certificate or a static IP. Use OAuth client submission on port 587, High Volume Email (internal only) or Azure Communication Services.
- Server in Azure or another third-party cloud? Microsoft says relay can't be used from there; client submission and High Volume Email support third-party hosted senders.
- Stopgap only: Basic login from IIS needs SMTP AUTH on for the mailbox (`Set-CASMailbox -Identity relay@yourdomain.com -SmtpClientAuthenticationDisabled $false`) and Basic auth still allowed. After the end of December 2026 admins can turn Basic auth back on until a final removal date, which Microsoft will announce in the second half of 2027. Details: Basic auth retirement (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- IIS smart host (today): Delivery > Advanced > Smart host = smtp.office365.com with Outbound Security > Basic authentication: breaks
- IIS outbound logins: Anonymous access, Basic authentication, Integrated Windows Authentication; no OAuth
- Microsoft support status: IIS 6.0 SMTP relaying to Microsoft 365 is unsupported; SMTP Server removed in Windows Server 2025
- Relay connector route: MX host, port 25, TLS 1.2/1.3, no login; connector trusts a certificate or static IP
- Client submission route: smtp.office365.com, port 587, TLS 1.2/1.3, OAuth (SMTP.Send or SMTP.SendAsApp); SMTP AUTH on for the mailbox
- High Volume Email route: smtp.hve.mx.microsoft, port 587, HVE account or OAuth; internal recipients only
- Direct Send route: MX host, port 25, TLS optional, no login; internal recipients only
- Supported local relay: Exchange Server with anonymous relay, or Azure Communication Services Email

**Sources:**
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365 (methods; IIS 6.0 SMTP tip): https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Features removed or no longer developed in Windows Server (SMTP Server, IIS 6 Management Console): https://learn.microsoft.com/en-us/windows-server/get-started/removed-deprecated-features-windows-server
- Microsoft Learn (archived, IIS 6.0): Requiring Authentication for Outbound Messages: https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc757569(v=ws.10)
- Microsoft Learn (archived, IIS 6.0): Configuring Smart Hosts: https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc779371(v=ws.10)
- Microsoft Learn (archived, IIS 6.0): Setting Connections (outbound TCP port): https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc782956(v=ws.10)
- Microsoft Learn (archived, IIS 6.0): Requiring TLS Encryption (Outbound Security > TLS encryption): https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc759573(v=ws.10)
- Microsoft Learn: Manage High Volume Email for Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/high-volume-mails-m365
- Microsoft Learn: Authenticate an IMAP, POP or SMTP connection using OAuth (SMTP.Send, SMTP.SendAsApp): https://learn.microsoft.com/en-us/exchange/client-developer/legacy-protocols/how-to-authenticate-an-imap-pop-smtp-application-by-using-oauth
- Microsoft Learn: Fix issues with printers, scanners, and LOB apps that send email using Microsoft 365: https://learn.microsoft.com/en-us/troubleshoot/exchange/email-delivery/fix-issues-with-printers-scanners-and-lob-applications-that-send-email-using-off
- Microsoft Learn: Allow anonymous relay on Exchange servers: https://learn.microsoft.com/en-us/exchange/mail-flow/connectors/allow-anonymous-relay
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication Deprecation Timeline (27 Jan 2026): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- Microsoft Q&A: 535 5.7.139 Authentication unsuccessful (reported error text): https://learn.microsoft.com/en-us/answers/questions/512954/535-5-7-139-authentication-unsuccessful

Full page: https://mailpassed.com/m365-smtp/windows-server-iis-smtp-relay-microsoft-365

## Sage 50 Accounts (UK and Ireland) and Sage 200 emailing invoices with Microsoft 365

Sage 50 Accounts v31 and later can send through Webmail with Modern (OAuth) sign-in to a Microsoft account. On v30 or earlier, and in Sage 200, Sage's answer for Microsoft accounts is to send through classic Outlook. SMTP with a Microsoft 365 password fails once Basic auth is off.

**OAuth:** **Sage 50 Accounts v31.0** added modern authentication (OAuth) for Microsoft accounts: in **Settings → Email Defaults → Email Client** choose Email Program **Webmail** and **Modern (OAuth)**, then authorise with the button in **Login Details**, which opens a browser sign-in. The same options are in Report Designer. Sage doesn't say whether this covers Microsoft 365 work or school accounts as well as Outlook.com, which Microsoft permissions it requests, or whether tenant admin consent is needed: not documented. **Sage 200** (Professional and Standard) documents no OAuth option; for Microsoft accounts Sage says to send using Outlook. Sage says sending through MAPI or Microsoft Outlook isn't affected by the SMTP authentication changes, so the Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement) doesn't touch it. It has to be classic Outlook: Sage says the new Outlook isn't compatible with Sage software.

**Errors you may see:**
- Sending through Internet Mail (SMTP) or Webmail with a Microsoft account and password stops working (Sage: those senders are affected by the SMTP authentication changes; MAPI and Outlook senders aren't)
- "535 5.7.3 Authentication unsuccessful" or "5.7.57 Client not authenticated to send mail" (Microsoft Learn: the SMTP AUTH login is refused)
- Emailing from Sage stops working after switching to the new Outlook (Sage: the new Outlook isn't compatible; revert to classic Outlook)
- "Emails sent using SMTP settings don't appear in your Inbox or Sent Items." (Sage: normal for SMTP, not a fault)

**Steps:**
1. Check how Sage sends today. Sage 50 Accounts: **Settings → Email Defaults**, Email Program **Microsoft Outlook** or **Webmail**. Also check layouts in Report Designer (**Tools → Options → Email Setup**): a Default Provider of **Internet Mail (SMTP)** with a Microsoft 365 address and password is Basic auth and fails once Microsoft turns it off by default at the end of December 2026. The free SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders) shows whether the mailbox Sage uses still logs in over SMTP AUTH.
2. Sage 50 Accounts v31 or later: **Settings → Email Defaults → Email Client** tab, Email Program **Webmail**, select **Modern (OAuth)**, click the button in **Login Details** and sign in through your browser. Click **Apply**, then **OK**.
3. Move Report Designer layouts that use Internet Mail (SMTP) to the same Webmail and Modern (OAuth) settings, or to Microsoft Outlook. Settings on a layout override Email Defaults.
4. Sage 50 Accounts v30 or earlier: set Email Program to **Microsoft Outlook** in Email Defaults, set up each layout tab, click **Apply**, select the layouts and click **OK**. Choose **Save emails to mailbox** or **Send emails immediately**. Sage recommends upgrading to v31 or later.
5. Sage 200: **Tools → Run Sage Report Designer → File Explorer → Default → Layouts → Report → Email Settings**, set **Mail Provider** to **Microsoft Outlook**. Do it for each layout. Classic Outlook must be installed on each workstation that sends.
6. Make Outlook the default email program in Windows and test with **Send To → Mail Recipient**. Run Outlook and Sage with the same **Run as administrator** setting.
7. Once nothing sends through SMTP any more, turn **Authenticated SMTP** off for that mailbox: Outlook doesn't use SMTP AUTH. Leave it on if you use Webmail with Modern (OAuth), since Sage doesn't document which protocol that uses.

**If it can't do OAuth:** 
- A relay connector is not a documented option for Sage. Sage's Webmail login offers only the logged-on user or a username and password, and its SMTP setup asks for a password. Use classic Outlook or Webmail with Modern (OAuth) instead.
- App passwords are no way out: Sage says Microsoft doesn't support App Passwords for SMTP.
- Stopgap only: an admin can turn Basic auth for SMTP AUTH back on for now. It ends when Microsoft removes it for good; the date is due in the second half of 2027. See the SMTP AUTH change (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Sage 50 Accounts OAuth (v31.0 or later): Settings > Email Defaults > Email Client > Email Program: Webmail > Modern (OAuth) > authorise in Login Details
- Sage 50 Accounts Outlook: Settings > Email Defaults > Email Program: Microsoft Outlook (classic Outlook only)
- Sage 50 SMTP per layout: Report Designer > Tools > Options > Email Setup > Internet Mail (SMTP) > Configure
- Sage 200 Outlook per layout: Tools > Run Sage Report Designer > File Explorer > Default > Layouts > Report > Email Settings > Mail Provider: Microsoft Outlook
- New Outlook for Windows: Not compatible with Sage software (Sage)
- Microsoft 365 work or school accounts with Modern (OAuth): Not documented (Sage says Microsoft accounts)
- Microsoft permissions / admin consent: Not documented
- Relay connector (no-login SMTP): Not documented

**Sources:**
- Sage UK: Modern authentication methods for email (Sage 50 Accounts v31.0): https://gb-kb.sage.com/portal/app/portlets/results/viewsolution.jsp?solutionid=241108102838610
- Sage UK: Changes to SMTP email authentication affecting Sage software: https://gb-kb.sage.com/portal/app/portlets/results/viewsolution.jsp?solutionid=240828130736293
- Sage UK: How to update email settings due to authentication changes (Sage 50 Accounts): https://gb-kb.sage.com/portal/app/portlets/results/view2.jsp?k2dockey=240829104044380
- Sage UK: Sage 200 email settings after authentication changes: https://gb-kb.sage.com/portal/app/portlets/results/view2.jsp?k2dockey=240829124704800
- Sage UK: Set up your documents for email (Email Defaults): https://gb-kb.sage.com/portal/app/portlets/results/view2.jsp?k2dockey=200427112407835
- Sage UK: Use webmail to email documents (Report Designer, Internet Mail (SMTP)): https://gb-kb.sage.com/portal/app/portlets/results/view2.jsp?k2dockey=200427112408752
- Sage UK: Troubleshoot email issues in Sage 50 Accounts: https://gb-kb.sage.com/portal/app/portlets/results/view2.jsp?k2dockey=221206081645653
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online (Outlook doesn't use SMTP AUTH): https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Learn: Fix issues with printers, scanners, and LOB apps that send email using Microsoft 365: https://learn.microsoft.com/en-us/troubleshoot/exchange/email-delivery/fix-issues-with-printers-scanners-and-lob-applications-that-send-email-using-off
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication deprecation timeline (MC786329): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835

Full page: https://mailpassed.com/m365-smtp/sage-50-accounts-sage-200-email-microsoft-365

## QuickBooks Desktop emailing invoices with Microsoft 365 (Web Mail or Outlook)

QuickBooks Desktop sends through Web Mail or Outlook. Update to the latest release, then either reauthorize Web Mail (Microsoft sign-in, grant Intuit access) or switch to Outlook 2010 or newer with a profile. Outlook doesn't use SMTP AUTH, so the Basic auth switch-off doesn't affect it.

**OAuth:** Intuit's **Web Mail** setup opens your provider's login page, where you sign in and grant Intuit access; the reauthorize flow ends with **Continue** and **Allow**, and Intuit says you don't need to change any default permissions set by Microsoft. Intuit doesn't call this OAuth, and which protocol and Microsoft permissions it uses are not documented. Intuit's provider table lists **Microsoft Outlook 365** as `outlook.office365.com`, port 587, SSL enabled. QuickBooks Community users reported in 2021 that Office 365 Web Mail only worked with Authenticated SMTP turned on, so older setups may still depend on SMTP AUTH. The **Outlook** option hands the email to Outlook 2010 or newer with an Outlook profile. Outlook doesn't use SMTP AUTH (Microsoft Learn), so the Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement) doesn't touch it. Whether the new Outlook for Windows works: not documented by Intuit.

**Errors you may see:**
- "QuickBooks is unable to send emails due to network connection failure" (Intuit: reauthorize Web Mail after Outlook or Google security changes)
- "QuickBooks is unable to send your email to Outlook" (Intuit: usually a settings issue on the Outlook route)
- "Outlook cannot log on" after updating to Office 365 version 2402 (Intuit points to a separate article)
- Outlook is missing in Send Forms preferences (Intuit: unsupported QuickBooks version, no Outlook profile, Outlook not the default email program, or Click-to-Run Office before QuickBooks Desktop 2024)
- Reported: the web authorization page just sits there after the Office 365 password is entered (QuickBooks Community, Nov 2021)

**Steps:**
1. Check the send method: **Edit → Preferences → Send Forms → My Preferences** shows **Web Mail** or **Outlook**. To see whether the QuickBooks mailbox still logs in over SMTP AUTH, run the free SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders).
2. Update QuickBooks Desktop to the latest release. Intuit lists this as the first step before reauthorizing.
3. Web Mail: **Edit → Preferences → Send Forms → Web Mail → Add**. Fill in **Add Email Info** (Intuit's values for Microsoft Outlook 365: `outlook.office365.com`, port 587, SSL on) and click OK twice. On Microsoft's login page, sign in and grant Intuit access. Intuit says webmail accounts need two-step verification turned on.
4. If QuickBooks says it can't send due to a network connection failure: open any form, click **Email**, then **Send**. Sign in with your Intuit account if asked, click **Continue** in the **Webmail Authorization** window, then **Allow**.
5. Outlook route: install Outlook 2010 or newer, create an Outlook profile, make Outlook the default email program with a default profile, then choose **Outlook** under **Send Forms**. Before QuickBooks Desktop 2024, Click-to-Run Office isn't supported; Intuit says 64-bit Outlook can help when Outlook doesn't appear.
6. If QuickBooks can't hand the email to Outlook: turn off **Run this program as Administrator** for`QuickBooks.exe`, select Outlook again under **Send Forms**, and if that fails run `C:\Windows\System32\Fixmapi.exe`.
7. Hosted QuickBooks (for example Rightworks) may still ask you to sign in to Web Mail each time you send. That is expected, per Intuit.

**If it can't do OAuth:** 
- Web Mail keeps failing with Microsoft 365? Switch to **Outlook**: it signs in to Microsoft 365 itself and doesn't depend on SMTP AUTH.
- A relay connector is not a documented option: Intuit's Web Mail always signs in to the provider, and a no-login server setting is not documented.
- Stopgap only: if your Web Mail setup still logs in with a password, an admin can turn Basic auth for SMTP AUTH back on for now. It ends when Microsoft removes it for good; the date is due in the second half of 2027. See the SMTP AUTH change (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Send method: Edit > Preferences > Send Forms > My Preferences: Web Mail or Outlook
- Web Mail, Microsoft (Intuit table): Provider Microsoft Outlook 365, server outlook.office365.com, port 587, SSL enabled
- Web Mail sign-in: Microsoft login page, grant Intuit access; reauthorize with Continue, then Allow
- OAuth / Microsoft permissions: Not documented by Intuit
- Outlook option: Outlook 2010 or newer with an Outlook profile, set as the default email program
- Click-to-Run Office: Not supported before QuickBooks Desktop 2024
- New Outlook for Windows: Not documented by Intuit
- Relay connector (no-login SMTP): Not documented

**Sources:**
- Intuit: Connect your email to QuickBooks Desktop (Web Mail and Outlook setup, provider table): https://quickbooks.intuit.com/learn-support/en-us/help-article/account-management/connect-email-quickbooks-desktop/L00d2EZek_US_en_US
- Intuit: Reauthorize QuickBooks Desktop to use Outlook and Gmail: https://quickbooks.intuit.com/learn-support/en-us/help-article/email-reports/reauthorize-quickbooks-desktop-keep-using-gmail/L6ligEQGU_US_en_US
- Intuit: Error: QuickBooks is unable to send your email to Outlook: https://quickbooks.intuit.com/learn-support/en-us/help-articles/error-quickbooks-is-unable-to-send-your-email-to-outlook/00/262232
- Intuit: Outlook is missing in QuickBooks Desktop Send Forms preferences: https://quickbooks.intuit.com/learn-support/en-us/help-articles/outlook-is-missing-in-quickbooks-desktop-send-forms-preferences/00/262487
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online (Outlook doesn't use SMTP AUTH): https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication deprecation timeline (MC786329): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- QuickBooks Community (user reports): Emailing invoice with Office 365 Exchange stopped working: https://quickbooks.intuit.com/learn-support/en-us/other-questions/emailing-invoice-with-office-365-exchange-stopped-working/00/496092/page/2

Full page: https://mailpassed.com/m365-smtp/quickbooks-desktop-email-microsoft-365

## Microsoft SQL Server Database Mail (and SQL Agent alerts) with Microsoft 365

Database Mail has no OAuth option: its SMTP Authentication choices are Windows, Basic or Anonymous. A Basic authentication account on smtp.office365.com stops working when Basic auth is switched off. Point the account at your MX host on port 25 with Anonymous authentication behind a relay connector, or use a local relay.

**OAuth:** Not documented. Microsoft's Database Mail pages (SQL Server 2017 to 2025 and Azure SQL Managed Instance) list three **SMTP Authentication** options: **Windows Authentication using Database Engine service credentials** (NTLM, which Microsoft says is only available with Exchange on-premises), **Basic Authentication** (user name and password) and **Anonymous authentication**. `sysmail_add_account_sp` has no token or OAuth parameter either. So an account that logs in to `smtp.office365.com` with a mailbox password is Basic auth and is hit by the SMTP AUTH Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement). SQL Server 2016 is not covered by the current pages; OAuth for it is not documented either. The TDS 8.0 support added in SQL Server 2025 is about client connections to SQL Server, not about SMTP sign-in.

**Errors you may see:**
- "Failure to authenticate with the SMTP server." (Microsoft Learn: one of the failures logged in sysmail_event_log)
- Mail items stay in sysmail_unsentitems with status retrying, then move to sysmail_faileditems (Microsoft Learn: Database Mail retries each account, by default 5 attempts 60 seconds apart)
- Reported: The mail could not be sent to the recipients because of the mail server failure. ... Cannot send mails to mail server. (The SMTP server requires a secure connection or the client was not authenticated. The server response was: 5.7.57 Client not authenticated to send mail. Error: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (Microsoft Q&A, Dec 2024)
- "535 5.7.3 Authentication unsuccessful" or "5.7.57 Client not authenticated to send mail" (Microsoft Learn: SMTP AUTH login refused; check that Authenticated SMTP is on for the mailbox)
- SQL Agent job and alert notifications to operators stop arriving (they go through the same Database Mail profile)

**Steps:**
1. Find the failing account. In SSMS go to **Management → Database Mail → View Database Mail Log**, or run `SELECT * FROM msdb.dbo.sysmail_event_log;` and join `sysmail_faileditems` on `mailitem_id`. To list every mailbox in the tenant that still logs in over SMTP AUTH, run the free SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders).
2. Open the account: right-click **Database Mail → Configure Database Mail → Manage Database Mail accounts and profiles → View, change, or delete an existing account**. **Server name** `smtp.office365.com` with **Basic Authentication** is the setup that fails once Microsoft turns Basic auth off by default at the end of December 2026.
3. In Exchange admin center, create a relay connector (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup) (**Mail flow → Connectors**, from **Your organization's email server**) that trusts the static public IP your SQL Server sends from.
4. Change the account: **Server name** your MX host (for example `contoso-com.mail.protection.outlook.com`), **Port number** 25, **SMTP Authentication** **Anonymous authentication**. Scripting it? In `sysmail_add_account_sp`, `@port = 25` and `@username = NULL` means no authentication (Microsoft Learn).
5. Set **E-mail address** to an address in a domain your tenant owns, or Microsoft 365 rejects the message (see 550 5.7.64 TenantAttribution (https://mailpassed.com/errors/microsoft365-550-5-7-64-tenantattribution)).
6. Right-click **Database Mail → Send Test E-Mail**. If nothing happens, check `EXEC msdb.dbo.sysmail_help_status_sp;` and start it with `EXEC msdb.dbo.sysmail_start_sp;`, then read `sysmail_event_log` again.
7. Azure SQL Managed Instance: SQL Agent mail must use the profile `AzureManagedInstance_dbmail_profile`, and the SMTP port needs an outbound rule in the instance's network security group (Microsoft Learn). The relay connector needs a static IP that you control; how that maps to Managed Instance outbound traffic is not documented.

**If it can't do OAuth:** 
- No static IP for a relay connector? Point Database Mail at a local SMTP relay on your network that signs in to Microsoft 365 itself. Microsoft's Managed Instance guide recommends an authenticated SMTP relay service and suggests one already in your on-premises servers.
- Without a connector, the MX host on port 25 is direct send, which only reaches mailboxes inside your organization (Microsoft Learn). Fine for a DBA team mailbox, not for outside recipients. See 550 5.7.68 (https://mailpassed.com/errors/microsoft365-550-5-7-68-direct-send-rejected) if direct send is blocked in your tenant.
- Don't use **Windows Authentication using Database Engine service credentials** for Microsoft 365: Microsoft says it is only available with Exchange on-premises.
- Stopgap only: an admin can turn Basic auth for SMTP AUTH back on for now. It ends when Microsoft removes it for good; the date is due in the second half of 2027. See the SMTP AUTH change (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Wizard path (SSMS): Object Explorer > Management > Database Mail > Configure Database Mail
- SMTP Authentication options: Windows Authentication using Database Engine service credentials, Basic Authentication, Anonymous authentication
- OAuth / XOAUTH2: Not documented (no OAuth option in the wizard or in sysmail_add_account_sp)
- Relay connector values: Server name: your MX host, Port number: 25, SMTP Authentication: Anonymous authentication
- T-SQL account parameters: @mailserver_name, @port (default 25), @username (NULL = no authentication), @password, @use_default_credentials, @enable_ssl
- Logs and views: msdb.dbo.sysmail_event_log, sysmail_faileditems, sysmail_unsentitems, sysmail_sentitems, sysmail_allitems
- Retry defaults: Account Retry Attempts 5, Account Retry Delay 60 seconds
- Azure SQL Managed Instance: SQL Agent profile must be named AzureManagedInstance_dbmail_profile; outbound NSG rule for the SMTP port
- Azure SQL Database: Database Mail not available

**Sources:**
- Microsoft Learn: Configure Database Mail (wizard fields, SMTP Authentication options): https://learn.microsoft.com/en-us/sql/relational-databases/database-mail/configure-database-mail
- Microsoft Learn: Database Mail (architecture, availability): https://learn.microsoft.com/en-us/sql/relational-databases/database-mail/database-mail
- Microsoft Learn: sysmail_add_account_sp (Transact-SQL): https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sysmail-add-account-sp-transact-sql
- Microsoft Learn: Troubleshoot Database Mail issues (sysmail views, Database Mail log): https://learn.microsoft.com/en-us/troubleshoot/sql/tools/troubleshoot-database-mail-issues
- Microsoft Learn: General Database Mail troubleshooting: https://learn.microsoft.com/en-us/sql/relational-databases/database-mail/database-mail-general-troubleshooting
- Microsoft Learn: Job automation with SQL Agent jobs in Azure SQL Managed Instance (Database Mail): https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/job-automation-managed-instance
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Fix issues with printers, scanners, and LOB apps that send email using Microsoft 365: https://learn.microsoft.com/en-us/troubleshoot/exchange/email-delivery/fix-issues-with-printers-scanners-and-lob-applications-that-send-email-using-off
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication deprecation timeline (MC786329): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- Microsoft Q&A (user report): Unable to send emails from SQL Server (535 5.7.139): https://learn.microsoft.com/en-us/answers/questions/2135824/unable-to-send-emails-from-sql-server

Full page: https://mailpassed.com/m365-smtp/sql-server-database-mail-microsoft-365

## Dell iDRAC and HPE iLO alert email (AlertMail) with Microsoft 365

Neither Dell iDRAC nor HPE iLO documents OAuth for alert email: both log in to SMTP with a user name and password, which stops working when Microsoft switches Basic auth off. Point them at your MX host on port 25 with authentication off, behind a relay connector that trusts your static IP.

**OAuth:** Not documented. Dell's iDRAC9 SMTP settings offer **Authentication** with a **Username** and **Password**, and Dell's email alert article doesn't mention OAuth or Microsoft 365. The iDRAC10 user guide's email alert section has SMTP server settings but no OAuth option; its OAuth 2.0 topics are separate from email alerts. HPE's AlertMail pages for **iLO 5**, **iLO 6** and **iLO 7** offer **SMTP Authentication** with an **SMTP Username** and password only, and HPE says iLO supports only STARTTLS SMTP servers. So a controller that logs in to `smtp.office365.com` with a mailbox password is Basic auth and is hit by the SMTP AUTH Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement). Dell says SMTP alerts need an **iDRAC Express** license or higher.

**Errors you may see:**
- Reported: RAC0225: Sending the test mail failed (Dell Community threads on iDRAC test email)
- The iDRAC Test Email or iLO Send Test AlertMail doesn't arrive (HPE: check the iLO event log to confirm a test message was sent)
- "535 5.7.3 Authentication unsuccessful" or "5.7.57 Client not authenticated to send mail" (Microsoft Learn: SMTP AUTH login refused; check that Authenticated SMTP is on for the mailbox)
- iDRAC mail stops after a firmware update to 4.00.00.00 or later (Dell: StartTLS became the default connection encryption)
- Alerts reach internal mailboxes but not outside addresses (Microsoft Learn: the controller uses direct send to the MX host without a relay connector)

**Steps:**
1. Find controllers that log in to Microsoft 365: an SMTP server of `smtp.office365.com` with authentication on. The free SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders) lists mailboxes that still log in over SMTP AUTH.
2. Create a relay connector (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup) in Exchange admin center (**Mail flow → Connectors**, from **Your organization's email server**) that trusts the static public IP the management network sends from.
3. iDRAC9: **Configuration → System Settings → SMTP (Email) Configuration**. Set **SMTP (Email) Server IP Address or FQDN/DNS Name** to your MX host (for example `contoso-com.mail.protection.outlook.com`), **SMTP Port Number** 25, turn **Authentication** off, and set **Sender Address** to an address in your domain. Then **Alert Configuration → Alerts**: Enable, tick the **Email** alerts you want and Apply.
4. iDRAC9: add a destination address, tick **State**, Apply, then click **Send** under **Test Email**. Dell recommends **Register iDRAC on DNS**. Leave connection encryption at StartTLS unless the test fails; Dell's fallback is `racadm set idrac.RemoteHosts.ConnectionEncryption None`. iDRAC7/8 use **Server → Alerts → SNMP and Email Settings** and have no STARTTLS.
5. iLO 5 and iLO 6: **Management → Mail** tab. iLO 7: **iLO Settings → Alert and Logging**, ellipsis next to **Mail Settings**, **Edit Settings**. Keep **Enable iLO AlertMail** on, set **SMTP Server** to your MX host, **SMTP Port** 25, keep **SMTP Secure Connection** on (STARTTLS) and turn **SMTP Authentication** off.
6. iLO: set **Sender Domain or Email Address** to an address in a domain your tenant owns (a bare domain becomes `iLO-hostname@domain`). A sender outside your domains is rejected; see 550 5.7.64 TenantAttribution (https://mailpassed.com/errors/microsoft365-550-5-7-64-tenantattribution). Click Apply (iLO 7: Update), then **Send Test AlertMail** and check the iLO event log.
7. iDRAC10: Dell's guide documents SMTP server settings under email alert settings (RACADM attribute `iDRAC.RemoteHosts.SMTPServerIPAddress`); use the same relay values. The iDRAC10 web menu path is not covered here.

**If it can't do OAuth:** 
- No static IP? Without a connector, the MX host on port 25 is direct send, which only reaches mailboxes inside your organization (Microsoft Learn). That is usually enough for an ops team mailbox. If your tenant blocks it, see 550 5.7.68 (https://mailpassed.com/errors/microsoft365-550-5-7-68-direct-send-rejected).
- Or send alerts to a local SMTP relay on your network that signs in to Microsoft 365 itself. Dell and HPE only ask for an SMTP server address, so any relay you run works.
- Stopgap only: an admin can turn Basic auth for SMTP AUTH back on for now. It ends when Microsoft removes it for good; the date is due in the second half of 2027. See the SMTP AUTH change (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- iDRAC9 SMTP path: Configuration > System Settings > SMTP (Email) Configuration
- iDRAC9 alert switch: Configuration > System Settings > Alert Configuration > Alerts: Enable, Email alerts ticked
- iDRAC connection encryption: StartTLS by default from firmware 4.00.00.00; racadm set idrac.RemoteHosts.ConnectionEncryption None to disable
- iDRAC license: iDRAC Express or higher (not iDRAC Basic)
- iLO 5 / iLO 6 path: Management > Mail tab
- iLO 7 path: iLO Settings > Alert & Logging > Mail Settings > Edit Settings
- iLO encryption: STARTTLS only; SMTP Authentication requires the secure connection option
- Relay values (both): SMTP server: your MX host, port 25, authentication off, sender address in your domain
- OAuth for alert email: Not documented (iDRAC9, iDRAC10, iLO 5, iLO 6, iLO 7)

**Sources:**
- Dell: PowerEdge: How to Configure iDRAC Email Alerts (KB 000131098): https://www.dell.com/support/kbdoc/en-us/000131098/dell-idrac-configuring-email-notifications-for-system-alerts-on-idrac7-8-and-idrac9
- Dell: iDRAC9 User's Guide 7.x, Configuring SMTP email server address settings using iDRAC web interface: https://www.dell.com/support/manuals/en-us/vsan-ready-node-r7615/idrac9_7.xx_ug/configuring-smtp-email-server-address-settings-using-idrac-web-interface?guid=guid-435ad044-047c-451a-841e-fe9822d373eb
- Dell: iDRAC10 User's Guide 1.20.xx, Configuring SMTP email server address settings using RACADM: https://www.dell.com/support/manuals/en-ru/poweredge-m7725/idrac10_1.20.xx_ug/configuring-smtp-email-server-address-settings-using-racadm?guid=guid-6e8770fa-6bb6-440d-8519-931ef7f8e8e7&lang=en-us
- HPE: iLO 5 User Guide, AlertMail options: https://support.hpe.com/hpesc/public/docDisplay?docId=a00105236en_us&page=GUID-D7147C7F-2016-0901-06D0-000000001241.html&docLocale=en_US
- HPE: iLO 6 User Guide, Enabling AlertMail: https://support.hpe.com/hpesc/public/docDisplay?docId=sd00002007en_us&page=GUID-D7147C7F-2016-0901-06D0-000000001259.html&docLocale=en_US
- HPE: iLO 6 User Guide, AlertMail options: https://support.hpe.com/hpesc/public/docDisplay?docId=sd00002007en_us&page=GUID-D7147C7F-2016-0901-06D0-000000001241.html&docLocale=en_US
- HPE: iLO 7 User Guide, Enabling AlertMail: https://support.hpe.com/hpesc/public/docDisplay?docId=sd00005342en_us&page=GUID-693AA755-60D8-4E36-9261-7CA6387A0646.html&docLocale=en_US
- HPE: iLO 7 User Guide, AlertMail options: https://support.hpe.com/hpesc/public/docDisplay?docId=sd00005342en_us&page=GUID-255AA6A7-D9CE-404D-A7FF-0CA58B020E03.html&docLocale=en_US
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Fix issues with printers, scanners, and LOB apps that send email using Microsoft 365: https://learn.microsoft.com/en-us/troubleshoot/exchange/email-delivery/fix-issues-with-printers-scanners-and-lob-applications-that-send-email-using-off
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication deprecation timeline (MC786329): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- Dell Community (user reports): RAC0225: Sending the test mail failed with iDRAC 4.X.X.X: https://www.dell.com/community/en/conversations/systems-management-general/rac0225-sending-the-test-mail-failed-with-idrac-4xxx/647f8691f4ccf8a8de59f745

Full page: https://mailpassed.com/m365-smtp/dell-idrac-hpe-ilo-alert-email-microsoft-365

## Jenkins email notifications (Mailer and Email Extension plugins) with Microsoft 365

The Mailer plugin only logs in with a user name and password, so it fails once Basic auth is off. Email Extension has a Use OAuth 2.0 box, but it doesn't fetch or refresh a Microsoft token. The working fix is a relay connector: SMTP server = your MX host, port 25, no credentials.

**OAuth:** **Mailer** (Manage Jenkins → Configure System → **E-mail Notification**): **Use SMTP Authentication** with a user name and password, nothing else. Its code has no OAuth option. **Email Extension** (section **Extended E-mail Notification**) has a **Use OAuth 2.0** checkbox under Advanced. In the plugin's source it only switches the SMTP login to `XOAUTH2` and sends the password of the selected **Credentials** as the token; it doesn't request or refresh a token from Microsoft. Jenkins issue **JENKINS-73486** ("OAuth 2.0 does not work with O365", In Progress) reports that a client secret in that field fails with 535 5.7.3, and that a token fetched by hand works but is only valid for about an hour. Jenkins selected a 2026 Google Summer of Code project to add Microsoft OAuth (client credentials) to Email Extension; it is not in a release yet. Until then, a Jenkins account on `smtp.office365.com` is hit by the SMTP AUTH Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Errors you may see:**
- "535 5.7.3 Authentication unsuccessful" (Microsoft Learn: SMTP AUTH or XOAUTH2 login refused)
- "5.7.57 Client not authenticated to send mail" (Microsoft Learn: the SMTP AUTH login didn't succeed)
- Reported: AUTH XOAUTH2 failed ... 535 5.7.3 Authentication unsuccessful with Use OAuth 2.0 and the app's client secret as the password (Jenkins issue JENKINS-73486)
- Reported: Email Extension with OAuth works for about an hour, then fails again (JENKINS-73486: the hand-fetched access token expired)
- Test configuration in E-mail Notification stops working after Basic auth is switched off (Mailer only logs in with a user name and password)

**Steps:**
1. Check both sections under **Manage Jenkins → Configure System**: **E-mail Notification** (Mailer) and **Extended E-mail Notification** (Email Extension). `smtp.office365.com` with a user name and password or Credentials is Basic auth and fails once Microsoft turns it off by default at the end of December 2026. The free SMTP AUTH sender script (https://mailpassed.com/m365-smtp/find-smtp-auth-senders) shows whether the Jenkins mailbox still logs in over SMTP AUTH.
2. Create a relay connector (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup) in Exchange admin center (**Mail flow → Connectors**, from **Your organization's email server**) that trusts the static public IP your Jenkins controller sends from.
3. Mailer (**E-mail Notification**): **SMTP server** your MX host (for example `contoso-com.mail.protection.outlook.com`), **SMTP Port** 25, untick **Use SMTP Authentication**, untick **Use SSL** (that is implicit SSL, port 465) and tick **Use TLS** (STARTTLS).
4. Email Extension (**Extended E-mail Notification**): same **SMTP server** and **SMTP Port** 25. Under Advanced, set **Credentials** to none, untick **Use SSL** and **Use OAuth 2.0**. Check any extra SMTP accounts in that section too.
5. Make sure the sender address is in a domain your tenant owns, or Microsoft 365 rejects it (see 550 5.7.64 TenantAttribution (https://mailpassed.com/errors/microsoft365-550-5-7-64-tenantattribution)).
6. Tick **Test configuration by sending test e-mail** in E-mail Notification, enter a recipient and click **Test configuration**. Then run a job with an email step.

**If it can't do OAuth:** 
- No static IP (for example Jenkins in a cloud with changing egress)? Point Jenkins at a local SMTP relay that signs in to Microsoft 365 itself. Jenkins only needs an SMTP server and port.
- Without a connector, the MX host on port 25 is direct send: it only reaches mailboxes inside your organization (Microsoft Learn). If your tenant rejects it, see 550 5.7.68 (https://mailpassed.com/errors/microsoft365-550-5-7-68-direct-send-rejected).
- Email Extension with **Use OAuth 2.0** plus a token you fetch yourself (Entra app with `SMTP.SendAsApp`, `New-ServicePrincipal`, `Add-MailboxPermission`, scope `https://outlook.office365.com/.default`) can send, but JENKINS-73486 reports the token must be replaced about every hour. Not a set-and-forget fix.
- Stopgap only: an admin can turn Basic auth for SMTP AUTH back on for now. It ends when Microsoft removes it for good; the date is due in the second half of 2027. See the SMTP AUTH change (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement).

**Settings:**
- Mailer path: Manage Jenkins > Configure System > E-mail Notification
- Mailer fields: SMTP server, Default user e-mail suffix, Use SMTP Authentication (User Name, Password), Use SSL, Use TLS, SMTP Port, Reply-To Address, Charset
- Email Extension path: Manage Jenkins > Configure System > Extended E-mail Notification
- Email Extension account fields: SMTP server, SMTP Port (default 25); Advanced: Credentials, Use SSL, Use TLS, Use OAuth 2.0, Advanced Email Properties
- Use OAuth 2.0 (Email Extension): Sets XOAUTH2 and sends the credential password as the token; no Microsoft token request or refresh
- Mailer OAuth: Not documented (password login only)
- Use SSL vs Use TLS: Use SSL = implicit SSL (465 if no port set); Use TLS = STARTTLS required (587 if no port set)
- Relay values: SMTP server: your MX host, SMTP Port: 25, no authentication or Credentials, OAuth off
- Microsoft OAuth status in Email Extension: JENKINS-73486 In Progress; GSoC 2026 project selected, not released

**Sources:**
- Jenkins plugins: Email Extension (configuration, Extended E-mail Notification): https://plugins.jenkins.io/email-ext/
- Jenkins plugins: Mailer (E-mail Notification fields): https://plugins.jenkins.io/mailer/
- GitHub jenkinsci/email-ext-plugin: MailAccount config (SMTP account fields, Use OAuth 2.0): https://github.com/jenkinsci/email-ext-plugin/blob/main/src/main/resources/hudson/plugins/emailext/MailAccount/config.groovy
- GitHub jenkinsci/email-ext-plugin: ExtendedEmailPublisherDescriptor (XOAUTH2 handling): https://github.com/jenkinsci/email-ext-plugin/blob/main/src/main/java/hudson/plugins/emailext/ExtendedEmailPublisherDescriptor.java
- GitHub jenkinsci/mailer-plugin: Mailer.java (SMTP session, SSL/TLS, password authenticator): https://github.com/jenkinsci/mailer-plugin/blob/master/src/main/java/hudson/tasks/Mailer.java
- Jenkins Jira: JENKINS-73486 Email Extension: OAuth 2.0 does not work with O365: https://issues.jenkins.io/browse/JENKINS-73486
- Jenkins Jira: JENKINS-69414 Email Extension plugin and OAuth 2.0 query: https://issues.jenkins.io/browse/JENKINS-69414
- Jenkins: GSoC 2026 project, Jenkins email notifications using Outlook SMTP with OAuth: https://www.jenkins.io/projects/gsoc/2026/projects/jenkins-email-notifications-using-outlook-smtp-with-oath/
- Microsoft Learn: Authenticate an IMAP, POP or SMTP connection using OAuth: https://learn.microsoft.com/en-us/exchange/client-developer/legacy-protocols/how-to-authenticate-an-imap-pop-smtp-application-by-using-oauth
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Fix issues with printers, scanners, and LOB apps that send email using Microsoft 365: https://learn.microsoft.com/en-us/troubleshoot/exchange/email-delivery/fix-issues-with-printers-scanners-and-lob-applications-that-send-email-using-off
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication deprecation timeline (MC786329): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835

Full page: https://mailpassed.com/m365-smtp/jenkins-email-notification-microsoft-365

## Microsoft 365 SMTP relay connector (any printer, scanner, NAS or app)

Create an inbound connector in the Exchange admin center (Mail flow → Connectors, from Your organization's email server to Office 365) that trusts your TLS certificate or static public IP. Point the device at your MX host on port 25 with TLS and no login, and add the IP to SPF.

**OAuth:** A relay connector uses neither OAuth nor a password. Exchange Online recognises the device by its TLS certificate (recommended) or by a static public IP that belongs only to your organization, so it keeps working after the SMTP AUTH Basic auth switch-off (https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement). It suits printers, scanners and old apps that can't do OAuth. If a device can do OAuth, Microsoft lists client submission with Modern authentication (`SMTP.Send`, or `SMTP.SendAsApp` for unattended apps) before relay, and apps you control can use Graph `sendMail`. Relay can't be used for mail from a third-party hosted service such as Azure, and dynamic IP addresses aren't supported.

**Errors you may see:**
- "550 5.7.64 TenantAttribution; Relay Access Denied" (Microsoft Learn: no connector matched the sending IP or certificate, often after an IP change or a renewed certificate)
- "4.4.62 Mail sent to the wrong Office 365 region" (Microsoft Learn: same cause as 5.7.64)
- "550 5.7.64 Relay Access Denied ATTR36" (Microsoft Learn: the certificate's domain isn't a verified accepted domain in your tenant)
- "Client was not authenticated to send anonymous mail during MAIL FROM" (Microsoft Learn: the device points at smtp.office365.com instead of your MX host)
- "550 5.7.68 TenantInboundAttribution; Direct Send not allowed for this organization from unauthorized sources" (Microsoft Exchange Team: RejectDirectSend is on and the message didn't match a connector)
- External mail stops after an ISP or network change (Microsoft Learn: the static IP changed; update the connector and SPF)
- Mail from the device lands in Junk (Microsoft Learn: add the device's IP to your SPF record)

**Steps:**
1. Find your MX host: Microsoft 365 admin center → **Settings → Domains** → your domain → **DNS records** → MX **Points to address or value**, for example `contoso-com.mail.protection.outlook.com`. Our checker (https://mailpassed.com/) shows it too.
2. Choose how Exchange Online will recognise the device: a TLS certificate whose Subject or SAN contains one of your accepted domains (recommended), or a static public IP that no other organization uses.
3. Exchange admin center → **Mail flow → Connectors → Add a connector**. **Connection from:** Your organization's email server. **Connection to:** Office 365 (set automatically). Name it, keep **Turn it on** ticked and leave **Retain internal Exchange email headers** unticked.
4. On **Authenticating sent email**, pick the certificate option and type the accepted domain from the certificate (`*.contoso.com` if it names several hosts), or pick the IP option and type the device's public IP. Review, then **Create connector**.
5. On the device: server = your MX host (never an IP address), port **25**, TLS/STARTTLS on (TLS 1.2 or 1.3), no username or password, From = any address in your accepted domain. It doesn't need a mailbox or licence.
6. Publish the IP in SPF, for example `v=spf1 ip4:203.0.113.10 include:spf.protection.outlook.com -all`. Keep one SPF record per domain; the SPF generator (https://mailpassed.com/spf-generator) merges it for you.
7. Send a test to an outside address. If it bounces, compare the code with the common errors on this page, or open the full fix: 550 5.7.64 TenantAttribution (https://mailpassed.com/errors/microsoft365-550-5-7-64-tenantattribution), 550 5.7.68 Direct Send not allowed (https://mailpassed.com/errors/microsoft365-550-5-7-68-direct-send-rejected), 5.7.60 send as this sender (https://mailpassed.com/errors/microsoft365-5-7-60-send-as-permission).

**If it can't do OAuth:** 
- Port 25 blocked by your ISP, or no static IP? Use OAuth instead: `smtp.office365.com` port 587 with XOAUTH2 (`SMTP.Send` delegated, `SMTP.SendAsApp` application). SMTP AUTH must stay enabled on that mailbox.
- Apps you can change: send with Microsoft Graph `sendMail` and the `Mail.Send` permission.
- Device or app in Azure or another cloud: relay won't work there. Use OAuth, Graph, or a third-party SMTP service with its own login, and add its include with the SPF generator (https://mailpassed.com/spf-generator).
- Stopgap only: an admin can turn SMTP AUTH back on for one mailbox while you migrate. See 535 5.7.139 (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth).

**Settings:**
- Server / smart host: Your MX endpoint, e.g. contoso-com.mail.protection.outlook.com (not an IP address)
- TCP port: 25
- TLS / STARTTLS: Enabled (TLS 1.2 or 1.3)
- Authentication: None on the device; the connector trusts your certificate (Subject or SAN = accepted domain) or a static public IP only you use
- From address: Any address in an accepted domain; no mailbox or licence needed
- Connector: Connection from: Your organization's email server. Connection to: Office 365
- SPF: v=spf1 ip4:203.0.113.10 include:spf.protection.outlook.com -all

**Sources:**
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Learn: Fix issues with printers, scanners, and LOB apps that send email using Microsoft 365: https://learn.microsoft.com/en-us/troubleshoot/exchange/email-delivery/fix-issues-with-printers-scanners-and-lob-applications-that-send-email-using-off
- Microsoft Learn: 550 5.7.64 TenantAttribution when users send mails externally: https://learn.microsoft.com/en-us/troubleshoot/exchange/email-delivery/ndr/relay-access-denied-smtp
- Microsoft Learn: 550 5.7.64 Relay Access Denied ATTR36: https://learn.microsoft.com/en-gb/troubleshoot/exchange/hybrid-configuration-wizard-errors/relay-access-denied-attr36-ndr
- Microsoft Learn: Set up SPF to identify valid email sources for your Microsoft 365 domain: https://learn.microsoft.com/en-us/defender-office-365/email-authentication-spf-configure
- Microsoft Exchange Team: Introducing more control over Direct Send in Exchange Online (550 5.7.68): https://techcommunity.microsoft.com/blog/exchange/introducing-more-control-over-direct-send-in-exchange-online/4408790
- Microsoft Learn: Authenticate an IMAP, POP or SMTP connection using OAuth: https://learn.microsoft.com/en-us/exchange/client-developer/legacy-protocols/how-to-authenticate-an-imap-pop-smtp-application-by-using-oauth

Full page: https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup
