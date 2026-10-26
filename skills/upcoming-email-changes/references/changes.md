# Dated changes that affect email and certificates

Generated from https://mailpassed.com/changes. Each entry has a status (confirmed, expected, reported), dates, who is affected, how to check and how to fix, with primary sources.

## CAs Must Validate DNSSEC (Since 15 Mar 2026): Broken DNSSEC Now Blocks Certificate Renewal

**Status:** confirmed · **Who:** CA/Browser Forum, IETF and IANA

Since 15 March 2026 every public CA must validate DNSSEC on its CAA and domain-validation lookups (CA/Browser Forum ballot SC-085), and a validation error is never permission to issue. A domain with a broken DNSSEC chain, usually a DS record left behind after a DNS host move, can't get or renew any certificate for its website, mail server or MTA-STS host. Domains without DNSSEC are not affected.

**Dates:**
- 2025-06-18: CA/Browser Forum adopts ballot SC-085v2: validate DNSSEC (when present) for CAA and domain-validation lookups
- 2025-10-13: S/MIME ballot SMC014 adopted: the same rule for CAA checks on S/MIME certificates
- 2025-11: RFC 9905: stop signing DNSSEC zones with SHA-1 (algorithms 5 and 7); resolvers that follow it treat them as unsigned
- 2026-02-16: Ballot SC-094: for email-based validation, DNSSEC is a must only on the CNAME, CAA and TXT lookups that find the domain
- 2026-03-03: DigiCert starts validating DNSSEC
- 2026-03-15: SC-085 and SMC014 take effect: every public CA validates DNSSEC back to the IANA root
- 2026-09-07: Ballot SC-100: the DNSSEC rules move to Baseline Requirements section 4.2.2.2
- 2026-10-11: Root zone key KSK-2024 becomes active; resolvers that only trust KSK-2017 fail validation
- 2027-01-11: IANA plans to revoke the old root key KSK-2017
- 2027-03-15: Maximum certificate validity drops to 100 days, so renewals (and these DNSSEC checks) come twice as often

**Affected:** Any domain with DNSSEC turned on (a DS record at the registrar) whose chain is broken: a DS left over from the previous DNS host, signatures the DNS host stopped refreshing, a key rollover gone wrong, or a DNS host that returns an error for CAA queries. Issuance and renewal fail at every public CA, for the website, the mail server certificate (ports 25, 465, 587, 993) and the MTA-STS policy host. Most domains notice only at their first renewal after March 2026, because certificates issued before then could last up to 398 days. Domains without DNSSEC are not affected.

**What you see:**
- Let's Encrypt: DNS problem: SERVFAIL looking up CAA for example.com - the domain's nameservers may be malfunctioning
- Let's Encrypt, when the resolver explains the failure: DNS problem: looking up A for www.example.com: DNSSEC: DNSKEY Missing (or Signature Expired, RRSIGs Missing, Bogus)
- DigiCert: We detected a DNSSEC validation issue on your domain. Please contact your DNS provider to review the configuration. (code dns_sec_validation_error)
- Expected: certbot, acme.sh, Caddy or cert-manager keep retrying, and the certificate expires on the website, the mail server and the MTA-STS host
- Expected: senders that validate DNSSEC defer mail to the domain, because their resolvers return SERVFAIL for its MX records too

**How to check:**
1. Run our checker (https://mailpassed.com/): the DNSSEC row validates the chain through 1.1.1.1 and shows the resolver's reason, and the CAA row tests the lookup every CA makes before it issues.
2. By hand: `dig CAA example.com @1.1.1.1`, then `dig +cd CAA example.com @1.1.1.1`. SERVFAIL on the first and an answer on the second means DNSSEC is the cause.
3. Let's Encrypt recommends DNSViz (https://dnsviz.net/) to see which link in the chain is broken (DS, DNSKEY or signatures).

**Fix:**
1. DS left from a previous DNS host: delete it at the registrar, or replace it with the DS your current host shows. Our checker prints the replacement DS when the old one doesn't match the zone's key.
2. Moving DNS hosts: Cloudflare's order is remove the DS, wait for its TTL to expire (often 24–48 hours), change the nameservers, then turn DNSSEC on again and add the new DS.
3. Signatures expired or missing: ask the DNS host to re-sign the zone, or delete the DS to go unsigned until they do. An unsigned (insecure) domain can get certificates again right away.
4. Zones signed with algorithm 5 or 7: roll to 13 (ECDSAP256SHA256) or 8 (RSASHA256), the algorithms RFC 9904 recommends.
5. CAA lookups that error while DNSSEC is fine: ask the DNS host to answer CAA queries (an empty answer is fine), or move DNS. Then retry the renewal and reload the services that use the certificate.

**Sources:**
- CA/Browser Forum: Ballot SC-085v2, validation of DNSSEC (when present) for CAA and DCV lookups: https://cabforum.org/2025/06/18/ballot-sc-085v2-require-validation-of-dnssec-when-present-for-caa-and-dcv-lookups/
- CA/Browser Forum Baseline Requirements, section 4.2.2.2 (DNSSEC validation requirements): https://github.com/cabforum/servercert/blob/main/docs/BR.md
- CA/Browser Forum: Ballot SC-100, DNSSEC clarification and consolidation: https://cabforum.org/2026/08/06/ballot-sc100-dnssec-clarification-and-consolidation/
- CA/Browser Forum: Ballot SC-094v2, DNSSEC exception in email DCV methods: https://cabforum.org/2026/01/15/ballot-sc094v2-dnssec-exception-in-email-dcv-methods/
- CA/Browser Forum: Ballot SMC014, DNSSEC for CAA (S/MIME): https://cabforum.org/2025/08/15/ballot-smc-014/
- DigiCert: validating DNSSEC to verify domain control and perform CAA checks: https://knowledge.digicert.com/alerts/digicert-validating-dnssec-to-verify-domain-control-and-perform-caa-checks
- Let's Encrypt: Certificate Authority Authorization (CAA): https://letsencrypt.org/docs/caa/
- Let's Encrypt Boulder source: DNS problem messages (bdns/problem.go): https://github.com/letsencrypt/boulder/blob/main/bdns/problem.go
- RFC 9905: Deprecating the use of SHA-1 in DNSSEC signature algorithms: https://www.rfc-editor.org/rfc/rfc9905.html
- RFC 9904: DNSSEC cryptographic algorithm recommendation update process: https://www.rfc-editor.org/rfc/rfc9904.html
- RFC 8914: Extended DNS Errors: https://www.rfc-editor.org/rfc/rfc8914.html
- Cloudflare: Troubleshooting DNSSEC: https://developers.cloudflare.com/dns/dnssec/troubleshooting/
- Cloudflare: Disable DNSSEC before moving your domain: https://developers.cloudflare.com/learning-paths/get-started/add-domain-to-cf/disable-dnssec/
- IANA: Root zone DNSSEC trust anchors (KSK-2024 active, KSK-2017 revocation): https://www.iana.org/dnssec/files

Full page: https://mailpassed.com/changes/cas-validate-dnssec-certificate-renewal

## Public TLS Certificates Lose clientAuth (2026–2027): Which Mail Servers Break

**Status:** confirmed · **Who:** Chrome Root Program, Let's Encrypt and the public CAs

Public CAs are removing TLS Client Authentication (clientAuth) from server certificates. Let's Encrypt's last dual-purpose certificates expired around 6 Oct 2026, Sectigo and DigiCert stop on 10 Feb and 1 Mar 2027, and Chrome requires serverAuth-only for certificates issued from 15 Mar 2027. Mail breaks only where a receiver verifies your public certificate as a client certificate, such as Postfix with `permit_tls_all_clientcerts`. Exchange Online connectors don't need clientAuth.

**Dates:**
- 2025-10-01: DigiCert stops adding clientAuth by default (still available on request)
- 2025-10-14: Sectigo stops adding clientAuth by default
- 2026-02-11: Let's Encrypt's default (classic) profile drops clientAuth
- 2026-04-13: Google Trust Services stops issuing clientAuth (week of 13 Apr 2026)
- 2026-07-08: Let's Encrypt ends its temporary tlsclient profile; its last dual-purpose certificates (90 days) expired around 6 Oct 2026
- 2026-09-13: GlobalSign issues public TLS only from its TLS-only roots
- 2027-02-10: Sectigo stops clientAuth completely, "no exceptions"
- 2027-03-01: DigiCert stops clientAuth completely (moved from 1 May 2026)
- 2027-03-15: Chrome Root Program: certificates issued from this date may only assert serverAuth
- 2027-09: The last dual-purpose certificates from Sectigo and DigiCert expire (up to 200 days after their last issue date; our estimate)

**Affected:** Only setups where a receiving mail server or gateway verifies the sending server's public certificate as a TLS client certificate: OpenSSL-based Postfix with permit_tls_all_clientcerts or smtpd_tls_req_ccert, Exim with tls_verify_hosts, and partner or relay gateways that require mutual TLS with a publicly trusted certificate. Postfix fingerprint matching (permit_tls_clientcerts, check_ccert_access) keeps working, and so do Exchange Online certificate-based connectors: Microsoft says certificates used to relay to Exchange Online don't need the clientAuth EKU. Most server-to-server mail uses no client certificate at all.

**What you see:**
- OpenSSL 3.0.8 and later: unsuitable certificate purpose (X509_V_ERR_INVALID_PURPOSE, error 26); OpenSSL 1.1.1 to 3.0.7 print unsupported certificate purpose
- Postfix (receiver): certificate verification failed for []: not designated for use as a client certificate (logged with smtpd_tls_req_ccert or the untrusted log level; at the default level only "Untrusted TLS connection established from ..." appears)
- Postfix with smtpd_tls_req_ccert = yes: the sender gets 421 4.7.1  Error: Client certificate not trusted
- Exim (OpenSSL build): SSL verify error: depth=0 error=unsuitable certificate purpose cert=, followed by TLS error on connection from ... (SSL_accept)
- A partner or relay that accepted your mail by certificate stops relaying right after your certificate was renewed.

**How to check:**
1. See what your server presents: our certificate check (https://mailpassed.com/certificate-check) reads the certificate on port 587, 465, 993 or 443 and lists its Extended Key Usage and expiry. Or run `openssl s_client -starttls smtp -connect mail.example.com:587 /dev/null | openssl x509 -noout -ext extendedKeyUsage -enddate`.
2. If the list shows `TLS Web Client Authentication`, the date that matters is its expiry: the renewal will come without it. On 26 Oct 2026 the certificates of outlook.office365.com (DigiCert), smtp.gmail.com (Google Trust Services) and mailpassed.com (Let's Encrypt) were already serverAuth only.
3. Find out whether anyone verifies your certificate as a client certificate: on Postfix receivers look for `permit_tls_all_clientcerts` or `smtpd_tls_req_ccert = yes`, on Exim for `tls_verify_hosts`, and ask partners or relays that "authenticate you by certificate".

**Fix:**
1. Use a private CA for client certificates and have the receiver trust that CA only. DigiCert, Sectigo, Google Trust Services and SSL.com all point to private or dedicated client-authentication certificates.
2. Postfix receivers: match senders by fingerprint with `check_ccert_access` or `permit_tls_clientcerts` instead of trusting any public certificate with `permit_tls_all_clientcerts`. Fingerprint matching doesn't need a trusted chain, so the missing EKU doesn't matter; update the fingerprint at every renewal.
3. Exim receivers: switch `tls_verify_hosts` to `tls_try_verify_hosts` and check the result in an ACL, or trust a private CA with `tls_verify_certificates`. Exim's own documentation suggests a private CA for client certificates.
4. Exchange Online: nothing to change. Certificate-based connectors don't need clientAuth, and Microsoft recommends them over IP-based connectors for on-premises servers (MC1226222).
5. Stop-gap only: Sectigo (until 10 Feb 2027) and DigiCert (until 1 Mar 2027) still add clientAuth on request, which buys at most 200 more days.

**Sources:**
- Chrome Root Program Policy (v1.8): serverAuth-only certificates from 15 Mar 2027: https://googlechrome.github.io/chromerootprogram/crp/policy
- Let's Encrypt: Ending TLS client authentication certificate support: https://letsencrypt.org/2025/05/14/ending-tls-client-authentication/
- Let's Encrypt community: Ending TLS client authentication certificate support in 2026: https://community.letsencrypt.org/t/ending-tls-client-authentication-certificate-support-in-2026/237403
- DigiCert: Sunsetting the client authentication EKU from public TLS certificates: https://knowledge.digicert.com/alerts/sunsetting-client-authentication-eku-from-digicert-public-tls-certificates
- Sectigo: Client authentication EKU deprecation FAQ: https://www.sectigo.com/faq-client-authentication-eku-deprecation
- Google Trust Services: clientAuth EKU removal: https://developers.google.com/public-key-infrastructure/updates/may2025-clientauth
- GlobalSign: Upcoming changes to TLS roots and certificate profiles: https://support.globalsign.com/ssl/upcoming-changes-tls-roots-and-certificate-profiles
- Microsoft Learn: How Exchange Online uses TLS to secure email connections: https://learn.microsoft.com/en-us/purview/exchange-online-uses-tls-to-secure-email-connections
- Postfix: postconf(5) (permit_tls_clientcerts, permit_tls_all_clientcerts, smtpd_tls_req_ccert): https://www.postfix.org/postconf.5.html
- Exim: Encrypted SMTP connections using TLS/SSL: https://www.exim.org/exim-html-current/doc/html/spec_html/ch-encrypted_smtp_connections_using_tlsssl.html
- OpenSSL: certificate purpose checks (crypto/x509/v3_purp.c): https://github.com/openssl/openssl/blob/master/crypto/x509/v3_purp.c

Full page: https://mailpassed.com/changes/tls-client-auth-eku-removed-mail-servers

## Gmail Stops Fetching POP Mail and Ends Gmailify in January 2027: Switch to Forwarding or IMAP

**Status:** confirmed · **Who:** Google

Google says existing users can use Gmail's "Check mail from other accounts" (POP) and Gmailify only until January 2027 (no exact day given). After that, new mail from the other account stops arriving in Gmail. Set up forwarding with DKIM on your domain, or read that mailbox over IMAP.

**Dates:**
- 2026-04: No longer available to new users (Google: after the first quarter of 2026)
- 2027-01: Existing users lose POP fetching and Gmailify (Google gives the month, not a day)

**Affected:** People who use Gmail on a computer to pull mail from another mailbox over POP, often a domain mailbox at a web host, and people who linked another account with Gmailify. Google's notice names only these two features.

**What you see:**
- New mail from the other account no longer shows up in Gmail. Nothing bounces; the mail stays in the other mailbox. Messages fetched before the change stay in Gmail.
- The "Check mail from other accounts" option is gone from Gmail settings on a computer.
- Accounts linked with Gmailify lose Gmail's spam protection, inbox categories, better mobile notifications and advanced search.
- Expected after a careless forwarding setup: forwarded mail lands in spam, or bounces with 550-5.7.26 Unauthenticated email from domain-name is not accepted due to domain's DMARC policy.

**How to check:**
1. In Gmail on a computer, open the settings tab **Accounts and Import**. If an account is listed under **Check mail from other accounts**, you are affected.
2. Run our checker (https://mailpassed.com/) on that account's domain and read the DKIM line. Google warns that forwarded mail often fails SPF, so a passing DKIM key for your own domain is what keeps it out of spam.
3. In the same report, check that SPF and DMARC exist and are valid for the domain.
4. Ask your old provider whether its forwarding rewrites the envelope sender to the forwarding domain (often called SRS), filters spam before forwarding, and can add ARC headers.

**Fix:**
1. Simplest long-term fix: point the domain's MX at a mailbox provider you read directly (for example Google Workspace (https://mailpassed.com/guides/google-workspace), Microsoft 365 (https://mailpassed.com/guides/microsoft-365) or Zoho Mail (https://mailpassed.com/guides/zoho-mail)), so nothing has to be fetched.
2. Or do what Google suggests: set up automatic forwarding at the old provider to your Gmail address. Have the envelope sender rewritten to the forwarding domain, filter spam before forwarding, and don't change the subject, body or headers on the way, or DKIM breaks.
3. Turn on DKIM for your domain at that provider. Keep one SPF record that lists it (SPF generator (https://mailpassed.com/spf-generator)) and publish DMARC (DMARC generator (https://mailpassed.com/dmarc-generator)).
4. If the forwarding host can add ARC headers, turn that on: ARC keeps the earlier SPF and DKIM results of a forwarded message.
5. Or skip forwarding: on a phone, add the account to the Gmail app (it uses IMAP); on a computer, use any IMAP mail client.
6. Google's notice doesn't mention "Send mail as". After switching, send a test reply to confirm it still works.

**Sources:**
- Google: Learn about upcoming changes to Gmailify & POP in Gmail: https://support.google.com/mail/answer/16604719
- Google: Best practices for forwarding email to Gmail: https://support.google.com/a/answer/175365
- Google Workspace: ARC email authentication: https://knowledge.workspace.google.com/admin/security/arc-email-authentication
- Google: Gmail SMTP errors and codes: https://knowledge.workspace.google.com/admin/support/troubleshooting/gmail-smtp-errors-and-codes

Full page: https://mailpassed.com/changes/gmail-pop-fetch-gmailify-ending

## DMARCbis (RFC 9989): pct Is Gone, t=, np= and psd= Are New — Update Your DMARC Record

**Status:** expected, no date · **Who:** IETF, then each mailbox provider

RFC 9989 (May 2026) replaces the old DMARC spec and drops `pct`: receivers that follow it apply your policy to all failing mail. New tags: `t=y` (test mode), `np=` and `psd=`. If your record has `pct` below 100, plan for full enforcement now.

**Dates:**
- 2026-05: RFC 9989 (DMARC), RFC 9990 (aggregate reports) and RFC 9991 (failure reports) published; RFC 7489 and RFC 9091 are obsolete. Mailbox providers have not announced switch dates

**Affected:** Domains that ramp up DMARC with `pct` below 100 (for example `p=quarantine; pct=10`), and domains that rely on `pct=0`. Also anyone who wants to block mail from made-up subdomains with `np=`. In our 23 Oct 2026 scan of the world's top 3,000 domains, 741 of the 2,022 with DMARC (36.6%) still publish `pct`, and 45 of them use a partial value such as 10, 25 or 50. Check your record (https://mailpassed.com/).

**What you see:**
- Expected: mail that fails DMARC is quarantined or rejected in full at RFC 9989 receivers, although your record still says pct=10 or pct=25.
- Expected at Gmail when the policy is reject: 550-5.7.26 Unauthenticated email from yourdomain.com is not accepted due to domain's DMARC policy.
- Expected: a domain that used pct=0 to make forwarders and mailing lists rewrite the From address loses that effect at receivers that only read t=.
- New-style aggregate reports (RFC 9990) carry a discovery_method field: "psl" (old method) or "treewalk" (RFC 9989).

**How to check:**
1. Run our checker (https://mailpassed.com/) and read the DMARC line. Look for `pct=` with any value below 100, and for `rf=` or `ri=`. RFC 9989 lists all three as historic and no longer defines them.
2. If you have `pct` below 100, open your DMARC aggregate reports (our free monitoring (https://mailpassed.com/monitor) reads them) and find the mail that fails DMARC today. That mail gets your full policy at RFC 9989 receivers.
3. If you have `pct=0`, note why: RFC 9989 says `t=y` now carries that meaning.
4. In new aggregate reports, look at `discovery_method`: `treewalk` means that receiver already uses RFC 9989.

**Fix:**
1. Replace percentage ramps with time-based steps: `p=none`, then `p=quarantine`, then `p=reject`, moving up only when reports show every sender passes. Build each record with the DMARC generator (https://mailpassed.com/dmarc-generator).
2. Need a soft step that works everywhere? Publish `pct=0` and `t=y` together, for example `v=DMARC1; p=reject; pct=0; t=y; rua=mailto:dmarc@yourdomain.com`. Old (RFC 7489) receivers treat failing mail one level lower because of `pct=0`; RFC 9989 receivers do the same because of `t=y`.
3. Remove `pct` values between 1 and 99, and remove `rf=` and `ri=`.
4. Add `np=reject` if no subdomain that doesn't exist in DNS should ever send mail. Don't add `psd=y`: it is for public suffix operators such as registries.
5. Keep `v=DMARC1` as the first tag. The version doesn't change.

**Sources:**
- RFC 9989: Domain-Based Message Authentication, Reporting, and Conformance (DMARC), May 2026: https://www.rfc-editor.org/info/rfc9989
- RFC 9989 Appendix A.6: Removal of the "pct" tag: https://www.rfc-editor.org/rfc/rfc9989.html#appendix-A.6
- RFC 9990: DMARC aggregate reporting (discovery_method): https://www.rfc-editor.org/info/rfc9990
- RFC 7489 section 6.6.4: how older receivers apply pct: https://www.rfc-editor.org/rfc/rfc7489.html#section-6.6.4
- Google: Gmail SMTP errors and codes: https://knowledge.workspace.google.com/admin/support/troubleshooting/gmail-smtp-errors-and-codes

Full page: https://mailpassed.com/changes/dmarcbis-rfc-9989-pct-removed

## Outlook "Promotions" Tag in Microsoft 365: What It Means for Your Newsletters and Notifications

**Status:** confirmed · **Who:** Microsoft

Since late July 2026 Microsoft 365 tags bulk promotional mail as "Promotions" in every tenant, whatever its complaint level; the rollout should finish by the end of October. Admins can also move it to a Promotions folder. Keep transactional mail separate, authenticated and easy to unsubscribe from.

**Dates:**
- 2026-07: Worldwide rollout starts (late July): bulk promotional mail gets the Promotions tag, on by default in all tenants
- 2026-10: Worldwide rollout expected to finish (late October)
- 2026-11: GCC, GCC High and DoD rollout expected to finish (late November)

**Affected:** Anyone who sends newsletters, marketing or other bulk mail to Microsoft 365 business mailboxes, plus automated mail (invoices, alerts) that Microsoft classifies as bulk. Tenants can't switch the tag off; admins can only exempt chosen senders and decide whether tagged mail moves to a folder.

**What you see:**
- Recipients see a Promotions tag on your message in Outlook on the web and the new Outlook for Windows.
- Your message lands in a Promotions folder instead of the Inbox where the recipient's admin turned on "Bulk moves enabled".
- Automated mail such as invoices from accounting software gets tagged too (reported in Microsoft Q&A: "Emails labeled promotions that are not promotions").
- The message headers of a received copy carry a BCL (bulk complaint level) value above 0.

**How to check:**
1. Send a test to a Microsoft 365 mailbox in another organization (mail from a tenant's own domains is never tagged) and look for the Promotions tag in Outlook on the web.
2. Open that message's headers and find the BCL value: 0 means not bulk, 1–3 bulk with few complaints, 4–7 mixed complaints, 8–9 many complaints.
3. Check whether invoices, password resets and alerts leave from the same domain and service as your newsletters.
4. Run our checker (https://mailpassed.com/) on each sending domain: SPF, DKIM and DMARC should all pass. That doesn't remove the tag, but Outlook.com rejects high-volume senders without them (550 5.7.515).

**Fix:**
1. Split your streams: send marketing from a subdomain (for example `news.yourdomain.com`) and keep transactional mail apart. Microsoft recommends a subdomain for bulk email services so their reputation doesn't affect your main domain.
2. Keep complaints low, because the BCL rises with them. Make unsubscribing easy with a visible link and a List-Unsubscribe header; Microsoft tells admins to have users look for an unsubscribe option. Microsoft doesn't document either as a way to avoid the tag.
3. Authenticate every sending domain: one SPF record (SPF generator (https://mailpassed.com/spf-generator)), DKIM with your own domain, and DMARC (DMARC generator (https://mailpassed.com/dmarc-generator)).
4. Ask business recipients to move your messages out of Promotions. Microsoft says the system learns from these moves for future messages.
5. For mail that must never be tagged (for example invoices to a client), the recipient's admin can add a mail flow rule that sets `X-MS-Exchange-Organization-BulkStamping` to `0` for your sender, or add you to their allow list.

**Sources:**
- Microsoft Learn: Bulk email detection (Promotions tag and folder), updated 22 Oct 2026: https://learn.microsoft.com/en-us/defender-office-365/anti-spam-bulk-complaint-level-bcl-about
- Microsoft 365 Message Center MC1279093: Enhancing how we handle promotional mail (mirror): https://mc.merill.net/message/MC1279093
- Microsoft Learn: What's the difference between junk email and bulk email?: https://learn.microsoft.com/en-us/defender-office-365/anti-spam-spam-vs-bulk-about
- Microsoft Learn: Set up SPF for Microsoft 365 (use a subdomain for bulk email services): https://learn.microsoft.com/en-us/defender-office-365/email-authentication-spf-configure
- Microsoft Support: Fix NDR error 550 5.7.515 in Outlook.com: https://support.microsoft.com/en-us/outlook/fix-ndr-error-550-5-7-515-in-outlook-com
- Microsoft Q&A: Emails labeled promotions that are not promotions (23 Oct 2026): https://learn.microsoft.com/en-us/answers/questions/6047683/emails-labeled-promotions-that-are-not-promotions

Full page: https://mailpassed.com/changes/outlook-promotions-tab-microsoft-365

## Microsoft 365 MX Records Move to *.mx.microsoft: What Changes for New and Existing Domains

**Status:** confirmed · **Who:** Microsoft

Since July 2026, new Microsoft 365 domains get an MX like `contoso-com.o-v1.mx.microsoft`, not `contoso-com.mail.protection.outlook.com`. Existing domains keep their MX; Microsoft has given no date to move them. Copy the MX from the admin center; never build it from a pattern.

**Dates:**
- 2026-07-01: New accepted domains get MX hosts under mx.microsoft (rolled out through July 2026; first planned for 1 Feb 2026). No date for existing domains

**Affected:** Admins and IT providers who add new domains to Microsoft 365, and any script, DNS template, connector or third-party gateway that writes `.mail.protection.outlook.com`. Existing domains keep working on the old MX and can move early by turning on DNSSEC for the domain in Exchange Online.

**What you see:**
- Mail flow fails during setup of a new domain because a DNS template or script created the old mail.protection.outlook.com MX (Microsoft MC1048624).
- The Microsoft 365 admin center domain health check doesn't show the MX record as OK.
- Get-DnssecStatusForVerifiedDomain: EX002: Value of MX record didn't match the expected one.
- With MTA-STS: ES001: Expected MX record missing in policy while mode is "enforced".

**How to check:**
1. Run our checker (https://mailpassed.com/) and look at the MX line. `.mail.protection.outlook.com` is the old style and `.mx.microsoft` the new one. A mix of both is a half-finished move.
2. Compare it with Microsoft 365 admin center → Settings → Domains → your domain → **DNS records** (or the `mailExchange` value from the Graph serviceConfigurationRecords API). They must match exactly.
3. If you use MTA-STS, open `https://mta-sts.yourdomain.com/.well-known/mta-sts.txt`. Every MX you publish needs its own `mx:` line; a new-style MX isn't covered by an old `mx:` line.
4. SPF needs no change: Microsoft's SPF value is `include:spf.protection.outlook.com` and doesn't contain the MX name.

**Fix:**
1. New domains: copy the MX value from the admin center or Graph into DNS. Microsoft says this is now the only source of truth; the new suffix can't be guessed from the domain name.
2. Update scripts, DNS templates, connectors and third-party gateways (their smart host setting) that hard-code `mail.protection.outlook.com`.
3. Moving an existing domain (optional, needed for inbound DANE): lower the MX TTL, put MTA-STS in testing mode, run `Enable-DnssecForVerifiedDomain -DomainName yourdomain.com`, add the returned MX at priority 20, test it, then set it to priority 0 and delete the old MX.
4. MTA-STS during the move: in testing mode, replace the old `mx:` line with the new host, change the policy id, then return to enforce once mail flows. Rebuild the policy from your live MX with our MTA-STS generator (https://mailpassed.com/mta-sts-generator).
5. If DNSSEC at your DNS host breaks mail flow, Microsoft's fallback is to turn DNSSEC off for the domain at the DNS provider and ask them how to re-enable it safely.

**Sources:**
- Microsoft 365 Message Center MC1048624: DNS provisioning change (mirror): https://mc.merill.net/message/MC1048624
- Microsoft Learn: How SMTP DANE works (moving a domain to mx.microsoft, MTA-STS steps, EX/ES errors): https://learn.microsoft.com/en-us/purview/how-smtp-dane-works
- Microsoft Learn: Set up SPF to identify valid email sources for your Microsoft 365 domain: https://learn.microsoft.com/en-us/defender-office-365/email-authentication-spf-configure
- CaptainDNS: Microsoft 365 MX and DNSSEC migration 2026 (secondary; no Microsoft date for existing domains): https://www.captaindns.com/en/blog/microsoft-365-mx-dnssec-migration-2026

Full page: https://mailpassed.com/changes/microsoft-365-mx-records-mx-microsoft

## Microsoft 365 SMTP AUTH: Basic Authentication Off by Default at the End of December 2026

**Status:** confirmed · **Who:** Microsoft

At the end of December 2026 Microsoft turns off Basic authentication (username and password) for SMTP AUTH in existing Exchange Online tenants. Printers, scanners, apps and scripts that log in to `smtp.office365.com` with a password will fail. Move them to OAuth or an SMTP relay now.

**Dates:**
- 2026-12: Basic auth for SMTP AUTH disabled by default in existing tenants (end of December); admins can still turn it back on
- 2027-01: Tenants created after December 2026 get no Basic auth for SMTP AUTH; OAuth is the supported method
- 2027-H2: Microsoft announces the final removal date

**Affected:** Every Microsoft 365 tenant where a printer, scanner, application, reporting server, script or POP/IMAP mail client sends through `smtp.office365.com` with a username and password. Outlook, Outlook on the web and Outlook mobile don't use SMTP AUTH, so people's normal mail keeps working.

**What you see:**
- 535 5.7.139 Authentication unsuccessful, basic authentication is disabled. (reported in Microsoft Q&A)
- 535 5.7.139 Authentication unsuccessful, SmtpClientAuthentication is disabled for the Tenant. Visit https://aka.ms/smtp_auth_disabled for more information. (what tenants with SMTP AUTH off see today; expected after the default change)
- 550 5.7.30 Basic authentication is not supported for Client Submission. (Microsoft's stated response once Basic auth is removed for good)
- The device's "test email" or "scan to email" fails with an authentication error, while the same mailbox still works in Outlook.

**How to check:**
1. Run our checker (https://mailpassed.com/) on your domain. If the MX ends in `.mail.protection.outlook.com` or `.mx.microsoft`, or SPF contains `include:spf.protection.outlook.com`, your mail runs on Microsoft 365 and this change applies to you.
2. In the Exchange admin center, open the **SMTP AUTH clients** report. Microsoft added a column that shows whether each sender uses Basic auth or OAuth. Every row with Basic auth will break.
3. Look at each device or app's mail settings: server `smtp.office365.com`, port 587, plus a username and password means Basic auth.
4. In Exchange Online PowerShell, run `Get-TransportConfig | Format-List SmtpClientAuthenticationDisabled` and `Get-CASMailbox -Identity user@yourdomain.com | Format-List SmtpClientAuthenticationDisabled` to see where SMTP AUTH is switched on today.

**Fix:**
1. Apps and scripts that support OAuth (Microsoft calls it Modern authentication): switch their SMTP login to OAuth. This is the method Microsoft supports long term.
2. Devices that can't do OAuth: use **SMTP relay**. Create an inbound connector in the Exchange admin center that trusts the device's TLS certificate or static public IP, point the device at your MX host on port 25, and add that IP to your SPF record (SPF generator (https://mailpassed.com/spf-generator)). Relay doesn't work from third-party clouds such as Azure.
3. Mail that only goes to people inside your own organization: Microsoft's High Volume Email (HVE) accepts Basic auth or OAuth, but it can't send to outside recipients.
4. Stop-gap only: Microsoft says admins can turn Basic auth back on after the default change (`Set-TransportConfig -SmtpClientAuthenticationDisabled $false`, or per mailbox with `Set-CASMailbox -Identity user@yourdomain.com -SmtpClientAuthenticationDisabled $false`). It doesn't work while security defaults are on, and it ends when Microsoft sets the final removal date.
5. Re-test each device with its "send test email" button, then check the SMTP AUTH clients report again until no Basic auth rows are left.

**Sources:**
- Microsoft Exchange Team: Updated Exchange Online SMTP AUTH Basic Authentication deprecation timeline (27 Jan 2026): https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- Microsoft Exchange Team: Exchange Online to retire Basic auth for Client Submission (SMTP AUTH), with the 550 5.7.30 response: https://techcommunity.microsoft.com/blog/exchange/exchange-online-to-retire-basic-auth-for-client-submission-smtp-auth/4114750
- Microsoft Learn: Enable or disable SMTP AUTH in Exchange Online: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- Microsoft Learn: How to set up a multifunction device or application to send email using Microsoft 365: https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365
- Microsoft Q&A: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled: https://learn.microsoft.com/en-us/answers/questions/5429662/smtp-server-authentication-error
- Zabbix issue ZBX-26652: 535 5.7.139 SmtpClientAuthentication is disabled for the Tenant: https://support.zabbix.com/browse/ZBX-26652

Full page: https://mailpassed.com/changes/microsoft-365-smtp-auth-basic-auth-retirement

## MailerSend Retires DKIM TXT Records on 1 February 2027: Move to the New CNAME Records

**Status:** confirmed · **Who:** MailerSend

MailerSend fully retires DKIM TXT records on 1 February 2027. If your domain still uses the old TXT key, add the two new CNAME records from your MailerSend dashboard now so your mail keeps passing DKIM. The new keys are 2048-bit and MailerSend rotates them without DNS changes.

**Dates:**
- 2027-02-01: MailerSend stops supporting DKIM TXT records (announced 8 Sep 2026)

**Affected:** MailerSend customers whose sending domain was set up with the older DKIM TXT record. Domains already on the two CNAME records need no change.

**What you see:**
- Expected: Authentication-Results headers show dkim=fail or dkim=none for your domain on mail sent through MailerSend.
- Expected at Gmail if DMARC then fails: 550-5.7.26 Unauthenticated email from yourdomain.com is not accepted due to domain's DMARC policy.
- Expected: more mail in spam at Gmail, which requires bulk senders to set up both SPF and DKIM.

**How to check:**
1. In MailerSend, go to Email → Domains and click **Manage** next to your domain. See whether it asks for one DKIM TXT record (old) or two CNAME records (new).
2. Look up the DKIM name shown there, for example `nslookup -type=TXT name._domainkey.yourdomain.com`. A TXT answer with a key and no CNAME means you still have to migrate.
3. Run our checker (https://mailpassed.com/) to confirm SPF and DMARC. DKIM selector names differ per account, so take the name from the MailerSend screen.

**Fix:**
1. In MailerSend, go to Email → Domains and click **Manage** next to the domain.
2. At your DNS host, create the two new CNAME records exactly as MailerSend shows them.
3. Back in MailerSend, tick **I have added DNS records** and click **Re-check now**.
4. Send a test to a Gmail address and open **Show original**: DKIM should say PASS for your domain.
5. That's it: MailerSend rotates the 2048-bit keys itself from then on.

**Sources:**
- MailerSend newsroom: New DKIM keys (8 Sep 2026): https://www.mailersend.com/newsroom/new-dkim-keys
- Google: Email sender guidelines: https://support.google.com/a/answer/81126
- Google: Gmail SMTP errors and codes: https://knowledge.workspace.google.com/admin/support/troubleshooting/gmail-smtp-errors-and-codes

Full page: https://mailpassed.com/changes/mailersend-dkim-txt-to-cname-feb-2027

## Shorter TLS Certificates for Mail Servers: Let's Encrypt 64 Days (Feb 2027), 100-Day Maximum (Mar 2027)

**Status:** confirmed · **Who:** CA/Browser Forum and Let's Encrypt

Let's Encrypt's default certificates drop from 90 to 64 days on 10 Feb 2027, and from 15 Mar 2027 no public certificate may last over 100 days (47 days from 2029). Mail servers with hand-copied certificates, or no reload after renewal, will serve expired ones. Automate renewal and the reload.

**Dates:**
- 2027-02-10: Let's Encrypt default (classic) profile: 64-day certificates, 10-day authorization reuse
- 2027-03-15: CA/Browser Forum: maximum public TLS certificate validity 100 days (it has been 200 days since 15 Mar 2026)
- 2028-02-16: Let's Encrypt: 45-day certificates, 7-hour authorization reuse
- 2029-03-15: CA/Browser Forum: maximum validity 47 days

**Affected:** Anyone who runs a mail server (Postfix, Exim, Dovecot, Exchange Server or an appliance) with a public certificate on ports 25, 465, 587, 993 or 995, especially if the certificate is copied in by hand or renewed without restarting the mail services. Inbound mail from senders that enforce MTA-STS or DANE stops when the certificate expires.

**What you see:**
- Exchange Online sending to a DANE domain: 4/5.7.322 certificate-expired: Destination mail server's certificate is expired.
- Expected: senders that enforce MTA-STS defer and then bounce mail to you, while senders using plain opportunistic TLS keep delivering.
- Expected: Outlook, Thunderbird and phone mail apps warn that the server certificate has expired when they connect over IMAP or SMTP.
- Expected: your website shows the new certificate, but the mail ports still serve the old one because Postfix or Dovecot was never reloaded.

**How to check:**
1. See what your mail server serves today: `openssl s_client -starttls smtp -connect mail.example.com:25 -servername mail.example.com /dev/null | openssl x509 -noout -dates -issuer`. Repeat with port 587, and for IMAP use `-connect mail.example.com:993` without `-starttls smtp`.
2. Look at `notBefore` and `notAfter`. If nobody can say which tool renews this certificate, it will break once lifetimes shrink.
3. Find the renewal job. Does your ACME client (certbot, acme.sh, lego, win-acme) reload the mail services after renewal? A renewal fixed at every 60 days is not enough for 64-day and 45-day certificates.
4. Run our checker (https://mailpassed.com/): if MTA-STS is on, every MX host name in your policy must be on the certificate. If you publish DANE TLSA records, note whether they pin the certificate (`3 0 1`), the key (`3 1 1`) or a CA (`2 x x`).

**Fix:**
1. Automate issuance for every mail host name with an ACME client. Let's Encrypt advises against manual renewal and recommends ACME Renewal Information (ARI), or renewing at about two-thirds of the lifetime.
2. Reload the mail services after each renewal: with certbot, add `--deploy-hook "systemctl reload postfix dovecot"` or put a script in `/etc/letsencrypt/renewal-hooks/deploy/`. Deploy hooks run only after a successful renewal.
3. DANE: publish `3 1 1` (hash of the public key) and keep the same key on renewal (`certbot --reuse-key`), so the record survives every renewal. The alternative is a usage 2 record for your CA (`2 1 1` on its key; RFC 7672's second choice is `2 0 1`), which must cover every CA certificate that can sign yours.
4. MTA-STS: keep each MX host name on the certificate and serve the full chain. Generate or fix the policy with our MTA-STS generator (https://mailpassed.com/mta-sts-generator).
5. Monitor expiry so a failed renewal shows up within days, not on the expiry date.

**Sources:**
- CA/Browser Forum Baseline Requirements v2.3.1, section 1.2.2 (200, 100 and 47-day dates): https://cabforum.org/working-groups/server/baseline-requirements/requirements/
- Let's Encrypt: Decreasing certificate lifetimes to 45 days: https://letsencrypt.org/2025/12/02/from-90-to-45/
- Certbot user guide: --deploy-hook and --reuse-key: https://eff-certbot.readthedocs.io/en/stable/using.html
- Microsoft Learn: How SMTP DANE works (4/5.7.322 certificate-expired): https://learn.microsoft.com/en-us/purview/how-smtp-dane-works
- RFC 7672: SMTP security via opportunistic DANE TLS (recommended TLSA records): https://www.rfc-editor.org/rfc/rfc7672.html
- RFC 8461: SMTP MTA Strict Transport Security (MTA-STS), section 4.2: https://www.rfc-editor.org/rfc/rfc8461.html#section-4.2

Full page: https://mailpassed.com/changes/mail-server-certificates-shorter-lifetime

## April 15, 2027 Root CA Distrust: Will Your Mail Server's Certificate Still Be Trusted?

**Status:** confirmed · **Who:** Mozilla, Chrome and the CAs

On 15 Apr 2027 Mozilla removes TLS trust from 19 roots with keys over 15 years old, including USERTrust RSA/ECC, COMODO RSA/ECC, Go Daddy Root G2, Starfield G2 and GlobalSign R3. Mail servers whose chain ends only at one of them will fail strict certificate checks. Check your chain and relays now.

**Dates:**
- 2027-04-15: Mozilla removes the websites (TLS) trust bit from roots with 2008–2009 key material; Sectigo lists Chrome and Mozilla distrust of its COMODO and USERTrust roots on the same day

**Affected:** Mail servers and SMTP relays whose certificate chain ends only at one of the 2027 roots, as seen by clients that use Mozilla-based trust stores (Thunderbird, curl's CA bundle and the many tools built on it) and by senders that enforce MTA-STS. Our check on 25 Oct 2026 found two big providers in that position, so this is a risk to watch, not a certain outage.

**What you see:**
- Expected (OpenSSL, as in the April 2026 DigiCert case on Outlook.com MX): verify error:num=20:unable to get local issuer certificate ... Verify return code: 20 (unable to get local issuer certificate)
- Expected (Python apps sending through a relay): [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate
- Expected (Thunderbird on IMAP or SMTP): SEC_ERROR_UNKNOWN_ISSUER
- Expected: senders that enforce MTA-STS stop delivering to an MX whose certificate no longer validates.

**How to check:**
1. Show the chain a server sends: `openssl s_client -starttls smtp -connect smtp.example.com:587 -servername smtp.example.com -showcerts </dev/null`. Use port 25 for an MX; for IMAP use `-connect imap.example.com:993` without `-starttls smtp`.
2. Read the `i:` (issuer) line of the last certificate. If it names USERTrust RSA or ECC Certification Authority, COMODO RSA or ECC Certification Authority, Go Daddy Root Certificate Authority - G2, Starfield Root Certificate Authority - G2, Starfield Services Root Certificate Authority - G2 or GlobalSign Root CA - R3, the chain depends on a root on Mozilla's 2027 list (19 roots in total).
3. Check the relays you send through, not just your own MX. On 25 Oct 2026 `smtp.sendgrid.net:587` ended at Go Daddy Root G2 (its certificate expires 19 Dec 2026), and `smtp.mail.me.com:587` and `mx01.mail.icloud.com:25` ended at USERTrust RSA.
4. If your own domain uses MTA-STS (our checker (https://mailpassed.com/) shows it), run the openssl check on every MX host.

**Fix:**
1. Your own server: reissue the certificate from a current hierarchy. Sectigo tells its customers to move to its R46 and E46 roots; other CAs have their own newer roots.
2. Install the full chain your CA gives you, including any cross-certificate. GoDaddy, for example, chains its new R1 certificates through G2 while R1 trust spreads.
3. Relays you don't control (SendGrid, iCloud and others): re-run the check in early 2027. If a relay still ends at a 2027 root in March, ask its support for their plan before 15 April.
4. Keep the CA bundles on your own servers and apps up to date, so newer roots are known as soon as root programs accept them. GoDaddy says its R1 root is still being added to browser and OS trust stores.
5. Don't re-add a distrusted root to your trust store as a fix. It hides the problem for you while other senders still fail.

**Sources:**
- Mozilla wiki: Root CA lifecycles (roots losing the websites trust bit on April 15, 2027): https://wiki.mozilla.org/CA/Root_CA_Lifecycles
- Sectigo: Changes to root CA hierarchies and trust status: https://www.sectigo.com/resource-library/changes-to-root-ca-hierarchies-and-trust-status
- GoDaddy Help: clientAuth EKU removal and the R1 root hierarchy (G2 ends April 2027): https://www.godaddy.com/en/help/why-is-godaddy-removing-clientauth-eku-and-transitioning-to-the-r1-root-hierarchy-for-dv-tls-issuance-42783
- curl: CA certificates extracted from Mozilla: https://curl.se/docs/caextract.html
- Microsoft Q&A: Untrusted root certificate for Outlook.com MX servers (April 2026 precedent): https://learn.microsoft.com/en-us/answers/questions/5925427/untrusted-root-certificate-for-outlook-com-mx-serv
- RFC 8461: MTA-STS, section 4.2 (certificate validation): https://www.rfc-editor.org/rfc/rfc8461.html#section-4.2

Full page: https://mailpassed.com/changes/root-ca-distrust-april-2027-mail-servers
