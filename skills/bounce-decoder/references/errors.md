# Bounce and SMTP error reference

Generated from https://mailpassed.com/errors. Search this file for the status code (for example `5.7.26`) or for a phrase from the bounce (the **Markers** lines list the phrases that identify each entry).

## Gmail 550 5.7.26: “sender is unauthenticated” / DMARC policy

**Provider:** Gmail · **Codes:** 550 5.7.26

**Short answer:** Gmail rejected the message because neither SPF nor DKIM passed for the domain in your From address. Fix: add the sending service to your one SPF record, turn on DKIM with your own domain, and keep DMARC at `p=none` until every sender passes.

**Looks like:**
    550-5.7.26 This email has been blocked because the sender is unauthenticated.
    550-5.7.26 Unauthenticated email from yourdomain.com is not accepted due to domain's DMARC policy.
    550-5.7.26 … has an SPF record with a hard fail policy (-all) but it fails to pass SPF checks

**Meaning:** Gmail could not authenticate the message: neither SPF nor DKIM passed for your domain, or they passed for a different domain than the one in the From address, and your DMARC policy (or Gmail's own sender rules) says to reject it. This is a permanent bounce – resending the same way will fail again.

**Fix:**
1. Run the checker below and look at SPF, DKIM and DMARC for the domain in your From address.
2. Make sure the service that sent the bounced message (your mailbox, CRM, newsletter or store app) is in your SPF record – build one with the SPF generator (https://mailpassed.com/spf-generator). Keep exactly one SPF record.
3. Turn on DKIM signing in that service with your own domain (not the service's default domain) and publish the CNAME/TXT records it gives you.
4. If the third bounce line appears, your SPF ends in `-all` and the sending IP is missing from it: add the sender, or use `~all` until the list is complete.
5. If you just moved DMARC to `p=reject` or `p=quarantine`, step back to `p=none` until every sender passes.

Full page: https://mailpassed.com/errors/gmail-550-5-7-26

## Gmail 421 4.7.26: “rate limited because it is unauthenticated”

**Provider:** Gmail · **Codes:** 421 4.7.26

**Short answer:** Gmail is temporarily deferring your mail because it is not authenticated with SPF or DKIM; it will bounce for good if nothing changes. Fix: add the sender to SPF, enable DKIM with your own domain and publish a DMARC record.

**Looks like:**
    421-4.7.26 This email has been rate limited because it is unauthenticated.
    421-4.7.26 Unauthenticated email from yourdomain.com is not accepted due to domain's DMARC policy, but temporary DNS failures …

**Meaning:** The temporary version of 5.7.26: Gmail is slowing down and deferring your mail because it is not authenticated with SPF or DKIM. Your server will retry, but if nothing changes the messages bounce for good after a few days. The second variant means Gmail could not even read your DNS records – check that your DNS host answers reliably.

**Fix:**
1. Check the domain below; fix every red SPF or DKIM line first.
2. Add the sending service to SPF (SPF generator (https://mailpassed.com/spf-generator)) and enable DKIM with your own domain.
3. Publish a DMARC record (DMARC generator (https://mailpassed.com/dmarc-generator)) – start with `p=none`.
4. If the error mentions DNS failures, test that your nameservers respond (the checker shows “unknown” when DNS does not answer).

Full page: https://mailpassed.com/errors/gmail-421-4-7-26

## Gmail 550 5.7.27: message didn't pass SPF authentication

**Provider:** Gmail · **Codes:** 550 5.7.27 / 421 4.7.27

**Short answer:** The sending server is not allowed by the SPF record of your envelope (Return-Path) domain, or that SPF record is broken (syntax error or more than 10 DNS lookups). Fix: add the server or service to a single, valid SPF record.

**Looks like:**
    550-5.7.27 This message was blocked because it didn't pass SPF authentication.
    421-4.7.27 Your email has been rate limited because SPF authentication didn't pass for this message.

**Meaning:** Gmail requires bulk senders to pass SPF. The server that sent this message is not listed in the SPF record of the envelope (Return-Path) domain, the record has a syntax error, or it needs more than 10 DNS lookups and is treated as broken.

**Fix:**
1. Run the checker: it counts SPF DNS lookups including nested includes.
2. Add the missing sender with the SPF generator (https://mailpassed.com/spf-generator) and replace (not add) your SPF record – two SPF records also fail.
3. Over 10 lookups? Remove services you no longer use or replace broad includes with the provider's narrower one.
4. If you send through a service that uses its own Return-Path domain, set up its custom return-path/bounce domain so SPF passes for your domain.

Full page: https://mailpassed.com/errors/gmail-550-5-7-27

## Gmail 550 5.7.30: message didn't pass DKIM authentication

**Provider:** Gmail · **Codes:** 550 5.7.30 / 421 4.7.30

**Short answer:** The message had no valid DKIM signature for your domain. Fix: enable DKIM in the sending service, publish its key exactly as given, and make sure nothing changes the message after it is signed.

**Looks like:**
    550-5.7.30 This message was blocked because it didn't pass DKIM authentication.
    421-4.7.30 Your email has been rate limited because DKIM authentication didn't pass for this message.

**Meaning:** The message had no valid DKIM signature for your domain: DKIM is not enabled in the sending service, the public key was never published (or was copied with a typo), or the message was changed after signing (for example by a forwarder or footer tool).

**Fix:**
1. Run the checker – it looks for DKIM keys on 40+ common selectors and flags weak 1024-bit keys.
2. In the sending service, open its domain authentication / DKIM page and publish exactly the records it shows (CNAME or TXT, host name included).
3. Click “verify” or “start authentication” in that service – many (Google Workspace, Microsoft 365) do not sign until you do.
4. Send a test to a Gmail address, open “Show original” and confirm `DKIM: PASS` with your domain.

Full page: https://mailpassed.com/errors/gmail-550-5-7-30

## Gmail 550 5.7.32: From header not aligned with SPF or DKIM

**Provider:** Gmail · **Codes:** 550 5.7.32 / 421 4.7.32

**Short answer:** SPF or DKIM passed, but for the email service's domain instead of the domain in your From address, so DMARC alignment failed. Fix: set up DKIM (or a custom Return-Path) with your own domain in that service.

**Looks like:**
    550-5.7.32 Your email was blocked because the From: header (RFC5322) in this message isn't aligned with either the authenticated SPF or DKIM organizational domain.

**Meaning:** SPF or DKIM passed – but for someone else's domain (usually the email service's), not for the domain in your From address. DMARC needs at least one of them to pass for your own domain. Typical case: a newsletter or CRM tool signs with its own default DKIM domain.

**Fix:**
1. Run the checker for the domain in your From address.
2. In the sending service, set up DKIM with your domain (custom/branded DKIM) instead of the shared default.
3. Optionally set a custom return-path (bounce) domain on your domain so SPF aligns too.
4. Keep DMARC alignment relaxed (default) unless you know every sender signs with the exact same domain.

Full page: https://mailpassed.com/errors/gmail-550-5-7-32

## Gmail 550 5.7.40: no DMARC record for the sending domain

**Provider:** Gmail · **Codes:** 550 5.7.40 / 421 4.7.40

**Short answer:** Your domain has no valid DMARC record, which Gmail requires from bulk senders. Fix: publish one TXT record at `_dmarc.yourdomain.com` such as `v=DMARC1; p=none; rua=mailto:you@yourdomain.com`.

**Looks like:**
    550-5.7.40 Your message was blocked because the sending domain doesn't have a DMARC record or the DMARC record doesn't specify a DMARC policy.

**Meaning:** Gmail requires bulk senders to publish DMARC. Your domain has no TXT record at `_dmarc.yourdomain.com`, or the record is missing the `p=` tag (or is broken, e.g. two DMARC records).

**Fix:**
1. Create the record with the DMARC generator (https://mailpassed.com/dmarc-generator): host `_dmarc`, value starting with `v=DMARC1; p=none`.
2. Add a report address (`rua=`) so you can see every service that sends as your domain.
3. Make sure there is only one TXT record at `_dmarc`, then run the checker below.

Full page: https://mailpassed.com/errors/gmail-550-5-7-40

## Gmail 550 5.7.24: suspicious entries in your SPF record

**Provider:** Gmail · **Codes:** 550 5.7.24 / 421 4.7.24

**Short answer:** Your SPF record allows too much (for example `+all`, `?all`, huge IP ranges or `ptr`), so Gmail doesn't trust it. Fix: list only your real senders and end the record with `~all` or `-all`.

**Looks like:**
    550-5.7.24 The SPF record of the sending domain has one or more suspicious entries.

**Meaning:** Gmail does not trust what your SPF record allows – typically `+all` or `?all` (anyone may send), huge IP ranges, or `ptr` mechanisms. Records like that make spoofing easy, so Gmail limits or blocks the mail.

**Fix:**
1. Run the checker – it flags `+all`, `?all`, `ptr` and lookup problems.
2. Rebuild the record with only your real senders using the SPF generator (https://mailpassed.com/spf-generator), ending in `~all` or `-all`.
3. Replace very wide ranges (like a whole /8 or /16) with the exact IPs or the provider's include.

Full page: https://mailpassed.com/errors/gmail-550-5-7-24

## Gmail 550 5.7.25: sending IP has no PTR (reverse DNS) record

**Provider:** Gmail · **Codes:** 550 5.7.25 / 421 4.7.23

**Short answer:** Your mail server's IP has no reverse DNS (PTR) name, or that name doesn't point back to the same IP. Fix: set a PTR record for the IP at your hosting provider and make the name resolve to that IP.

**Looks like:**
    550-5.7.25 This message was blocked because the sending IP address doesn't have a PTR record, or the forwarding DNS entry of the PTR record doesn't match the sending IP address.

**Meaning:** This is about your mail server's IP, not your domain's TXT records: the IP has no reverse DNS name, or that name does not resolve back to the same IP. It usually hits self-hosted servers and VPSs (Postfix, cPanel, WordPress SMTP on a VPS).

**Fix:**
1. Set the PTR (reverse DNS) of the sending IP to a host name like `mail.yourdomain.com` – this is done at the VPS/hosting provider, not in your DNS zone.
2. Create an A (and AAAA for IPv6) record for that host name pointing to the same IP.
3. Use the same host name as your mail server's HELO/EHLO name.
4. Simpler option: send through your mailbox provider or a sending service instead of the VPS directly.

Full page: https://mailpassed.com/errors/gmail-550-5-7-25

## Gmail 550 5.7.29: message not sent over a TLS connection

**Provider:** Gmail · **Codes:** 550 5.7.29 / 421 4.7.29

**Short answer:** Your server sent to Gmail without TLS encryption, which bulk senders must use. Fix: enable STARTTLS on your mail server or relay and renew any expired or invalid certificate.

**Looks like:**
    550-5.7.29 This message was blocked because it wasn't sent over a TLS connection.

**Meaning:** Bulk senders must deliver to Gmail over an encrypted (STARTTLS) connection. Your mail server or SMTP relay sent in plain text – usually an old server, a disabled TLS setting, or an expired/invalid certificate that made the server fall back.

**Fix:**
1. Enable opportunistic TLS for outgoing mail (Postfix: `smtp_tls_security_level = may`).
2. Install a valid certificate for the server's host name and keep it renewed.
3. If an app sends through an SMTP relay, use the relay's TLS port (587 with STARTTLS or 465).

Full page: https://mailpassed.com/errors/gmail-550-5-7-29

## Outlook 550 5.7.515: domain does not meet the required authentication level

**Provider:** Outlook / Hotmail · **Codes:** 550 5.7.515

**Short answer:** Outlook.com rejects high-volume senders (5,000+ messages a day) unless SPF and DKIM pass and DMARC is published and aligned. Fix: pass SPF and DKIM for your own domain and publish at least `v=DMARC1; p=none`.

**Looks like:**
    550 5.7.515 Access denied, sending domain yourdomain.com does not meet the required authentication level.

**Meaning:** Since May 2025 Outlook.com, Hotmail and Live addresses reject mail from domains that send them 5,000+ messages a day unless SPF passes, DKIM passes and DMARC is published (at least `p=none`) and aligned with SPF or DKIM.

**Fix:**
1. Run the checker for the domain in your From address – SPF, DKIM and DMARC all need to be green.
2. Add every sending service to SPF (SPF generator (https://mailpassed.com/spf-generator)) and enable DKIM with your own domain in each of them.
3. Publish DMARC (DMARC generator (https://mailpassed.com/dmarc-generator)) with a report address, then watch the reports for senders you forgot.
4. Forwarded mail can break DKIM: if the bounce comes after a forward, fix authentication at the original sender.

Full page: https://mailpassed.com/errors/outlook-550-5-7-515

## Outlook 550 5.7.1 S3150 / S3140: “part of their network is on our block list”

**Provider:** Outlook / Hotmail · **Codes:** 550 5.7.1 S3150 / S3140

**Short answer:** Outlook.com, Hotmail and Live have blocked the IP address (or the whole IP range) of your mail server, so nothing you send from it reaches them. Fix: check authentication and blocklists, then ask Microsoft to unblock the IP with the Outlook.com delisting form.

**Looks like:**
    550 5.7.1 Unfortunately, messages from [203.0.113.10] weren't sent. Please contact your Internet service provider since part of their network is on our block list (S3150).
    … (S3140). You can also refer your provider to http://mail.live.com/mail/troubleshooting.aspx#errors.

**Markers:** S3150, S3140

**Meaning:** Microsoft's consumer mail service (outlook.com, hotmail.com, live.com, msn.com) has put the IP address of your mail server on its block list. It is very common on new VPS and cloud servers, because the IP or its range was used for spam before you got it. Microsoft doesn't publish what the codes mean; from what hosting providers observe, **S3150** often lifts by itself after a few days without mail to Outlook.com, while **S3140** stays until Microsoft removes it. Microsoft 365 business mailboxes use a separate list (see 550 5.7.606 below).

**Fix:**
1. Check the domain you send from with the checker below: SPF, DKIM and DMARC should all pass. Outlook.com requires them from senders of 5,000+ mails a day and penalises missing ones for everyone.
2. Make sure the server's reverse DNS (PTR) is set to its mail host name, and that the IP is not on Spamhaus or other DNS blocklists – Microsoft uses Spamhaus data too.
3. Rule out spam from the server itself: an open relay, a hacked WordPress or contact form, or a mailing list without opt-in will get the IP blocked again right after delisting.
4. Ask for delisting with the Outlook.com delisting form (https://support.microsoft.com/supportrequestform/8ad563e3-288e-2a61-8122-3ba03d6b8d75): give the IP and paste the full bounce. The first reply is often an automatic “not eligible”: reply to that email with the details it asks for so a person reviews it.
5. If the IP is new to you, ask your hosting provider for a short note confirming when it was assigned to you; providers such as Hetzner supply one for exactly this case.
6. Sign up for Microsoft SNDS (https://substrate.office.com/ip-domain-management-snds/SNDS) (IP reputation data) and JMRP (https://substrate.office.com/ip-domain-management-snds/SNDS/Jmrp) (complaint reports). They don't delist you, but they show problems early.
7. If you only send a few hundred mails a day, sending through an established service (Amazon SES, Postmark, Mailgun, Brevo or your mailbox provider's SMTP) avoids IP reputation problems altogether.

Full page: https://mailpassed.com/errors/outlook-550-5-7-1-s3150

## Microsoft 365 550 5.7.606 / 5.7.511: “Access denied, banned sending IP”

**Provider:** Microsoft 365 · **Codes:** 550 5.7.606 / 5.7.511

**Short answer:** Microsoft 365 has put your sending IP on its blocked senders list, so mail to business mailboxes hosted at Microsoft bounces. Fix: request removal at sender.office.com (5.7.606–649), or email the bounce to delist@microsoft.com (5.7.511).

**Looks like:**
    550 5.7.606-649 Access denied, banned sending IP [203.0.113.10]: To request removal from this list please visit https://sender.office.com/ and follow the directions.
    550 5.7.511 Access denied, banned sender[203.0.113.10]. To request removal from this list, forward this message to delist@microsoft.com.

**Meaning:** The recipient's company email runs on Microsoft 365 (Exchange Online), and Microsoft has added your server's IP to its blocked senders list after seeing spam, spoofing or phishing from it or its range. Every message from that IP to any Microsoft 365 organisation bounces until it is removed. This is a different list from the Outlook.com consumer block (S3150/S3140).

**Fix:**
1. **5.7.606 to 5.7.649:** open the Office 365 Anti-Spam IP Delist Portal (https://sender.office.com/), enter the email address that got the bounce and the IP from the error (one of each per visit), confirm the link Microsoft emails you, then click **Delist IP**. Removal can take up to 24 hours or longer.
2. **5.7.511:** the portal won't work for this one. Forward the full bounce, including the IP, to `delist@microsoft.com`; Microsoft says it replies within 48 hours with next steps.
3. Before you delist, fix the cause: authentication (check your domain below), a compromised account or website sending spam, or bulk mail without opt-in. Messages still have to pass Microsoft's filters afterwards, and the IP can be blocked again.
4. If Microsoft refuses and the IP belongs to your hosting provider, ask the provider to contact Microsoft, or move mail to a reputable sending service.

Full page: https://mailpassed.com/errors/microsoft365-550-5-7-606

## Microsoft 550 5.7.509: “sending domain does not pass DMARC verification”

**Provider:** Microsoft 365 / Outlook · **Codes:** 550 5.7.509

**Short answer:** Microsoft rejected the message because it failed DMARC and your domain publishes `p=reject`: neither SPF nor DKIM passed for the domain in your From address. Fix: make the sending service DKIM-sign with your domain (or add it to SPF), then check the result in the message headers.

**Looks like:**
    550 5.7.509 Access denied, sending domain [YOURDOMAIN.COM] does not pass DMARC verification and has a DMARC policy of reject.

**Markers:** 5.7.509

**Meaning:** Microsoft 365 and Outlook.com honour the sender's DMARC policy: when a message fails DMARC and the From domain says `p=reject`, it is rejected (with `p=quarantine` it goes to junk instead). DMARC passes only when SPF or DKIM passes **and** the passing domain matches the From address. The fix is always on the sending side; the recipient can't repair your DNS.

**Fix:**
1. Check the From domain with the checker below and look at DKIM first: most failures come from a tool (newsletter, CRM, helpdesk, website form) that signs with its own domain instead of yours.
2. In that tool, turn on domain authentication / custom DKIM so it signs as your domain (see the setup guides (https://mailpassed.com/guides)). Adding its SPF include helps only if it also uses your domain as the return path.
3. Can't get a tool aligned? Send its mail from a subdomain (for example `news.yourdomain.com`), set up SPF and DKIM there, and keep relaxed alignment (`adkim=r; aspf=r`) in your DMARC record.
4. Forwarding breaks SPF: if the bounce follows a forward, DKIM is what has to survive. Check that the original message was DKIM-signed with your domain.
5. Need mail flowing while you fix it? Switching DMARC to `p=quarantine` or `p=none` for a few days stops the rejections, but it also lets spoofed mail through. Move back once every sender passes.
6. Confirm the fix in Outlook: open a test message, view the message source and check `Authentication-Results` for `dmarc=pass`.

Full page: https://mailpassed.com/errors/microsoft-550-5-7-509

## Outlook 451 4.7.650: “temporarily rate limited due to IP reputation” (S775)

**Provider:** Outlook / Hotmail · **Codes:** 451 4.7.650 (S775)

**Short answer:** Outlook.com is slowing down mail from your server's IP because of its reputation. It is a temporary deferral, not a block: messages are retried, and the limit lifts once Microsoft sees clean traffic. Fix what hurts the IP's reputation and, if it lasts for days, ask Microsoft sender support to review it.

**Looks like:**
    451 4.7.650 The mail server [203.0.113.10] has been temporarily rate limited due to IP reputation. For e-mail delivery information, see https://postmaster.live.com (S775)

**Markers:** 4.7.650, S775

**Meaning:** Outlook.com, Hotmail and Live accept only part of your mail and defer the rest because the sending IP has a weak or unknown reputation: a new server, a sudden jump in volume, spam complaints or spam from a compromised account. A 4xx code means "try again later", so your server keeps retrying. It turns into a bounce only when the retries run out. A hard block shows up as 550 5.7.606 (https://mailpassed.com/errors/microsoft365-550-5-7-606) or 550 5.7.1 S3150 (https://mailpassed.com/errors/outlook-550-5-7-1-s3150) instead.

**Fix:**
1. Check the domain below: SPF, DKIM and DMARC should all pass. Microsoft weighs authentication heavily for new IPs.
2. Look at your outgoing queue and logs for spam: a hacked mailbox, website form or script sending in bulk is the most common cause of a sudden S775.
3. Slow down: send less mail to Outlook.com addresses per hour and grow volume gradually; drop old, unengaged and bouncing addresses.
4. Make sure the server's reverse DNS (PTR) matches its host name and the IP is not on Spamhaus.
5. Sign up for Microsoft SNDS (IP reputation data) and JMRP (complaint reports) to see what Microsoft sees.
6. If it doesn't clear within a few days, open a request with Outlook.com sender support and include the full bounce. Owners report that the first reply is often an automatic "no issue detected"; replying to ask for escalation is what got the limit raised.

Full page: https://mailpassed.com/errors/outlook-451-4-7-650

## Yahoo 421 4.7.0 [TSS04]: “temporarily deferred due to unexpected volume or user complaints”

**Provider:** Yahoo / AOL · **Codes:** 421 4.7.0 [TSS04]

**Short answer:** Yahoo (and AOL) are temporarily deferring mail from your IP because of complaints, spam-like content, poor IP reputation or unusual volume. Your server retries automatically. Fix the cause and keep volume steady, and if it persists, send a Sender Support Request to Yahoo with the full error.

**Looks like:**
    421 4.7.0 [TSS04] Messages from 203.0.113.10 temporarily deferred due to unexpected volume or user complaints - 4.16.55.1; see https://postmaster.yahooinc.com/error-codes

**Markers:** TSS04

**Meaning:** Yahoo's TS codes are temporary deferrals. Yahoo lists the usual causes as complaints, spam-like content, poor IP reputation and unusual traffic. It is common after a big campaign, on a new IP, or when many recipients mark the mail as spam. Mail is retried, so most of it still arrives late. If it keeps happening, it means Yahoo trusts your traffic less.

**Fix:**
1. Meet Yahoo's sender requirements first: SPF or DKIM for everyone, and SPF, DKIM and DMARC for bulk senders. Check your domain below.
2. Make unsubscribing easy (one-click List-Unsubscribe for marketing mail) and keep the spam complaint rate under 0.3%.
3. Send to people who asked for your mail: remove unengaged and bouncing addresses and never send to bought lists.
4. Spread large sends over hours instead of minutes, especially from a new IP.
5. Check outgoing queues for spam from a compromised account or form.
6. If the deferral continues, submit a Sender Support Request to Yahoo with the error text and the diagnostic codes from your logs.

Full page: https://mailpassed.com/errors/yahoo-421-4-7-0-tss04

## Yahoo 553 5.7.1 [BL21]: “connections will not be accepted … ip is in Spamhaus's list”

**Provider:** Yahoo / AOL · **Codes:** 553 5.7.1 [BL21] / [BL23]

**Short answer:** Yahoo refuses connections from your IP because it is on a Spamhaus blocklist. Find out why the IP was listed, fix it, and request removal from Spamhaus. Yahoo starts accepting mail again once the listing is gone.

**Looks like:**
    553 5.7.1 [BL21] Connections will not be accepted from 203.0.113.10, because the ip is in Spamhaus's list; see https://postmaster.yahooinc.com/error-codes

**Markers:** BL21, BL23

**Meaning:** A 553 code is permanent: Yahoo won't take mail from that IP at all while the listing lasts, and retrying does not help. Yahoo checks Spamhaus, which lists IPs that send spam, are infected, or sit in ranges that should not send mail directly (typical for home connections and some cloud ranges).

**Fix:**
1. Look up the IP from the bounce on the Spamhaus IP and Domain Reputation Checker; it says which list and why.
2. Fix the reason first: a compromised mailbox, an infected PC on the network, an open relay, a website form or script sending spam.
3. Then follow the removal link Spamhaus shows for that listing. Policy lists for dynamic or cloud ranges usually mean you should send through your provider's mail server or a sending service instead.
4. On a VPS, make sure the reverse DNS is set to your mail host name and only your own mail software can send.
5. While you wait, a reputable sending service (your mailbox provider's SMTP or a transactional email service) gets mail out without the listed IP.

Full page: https://mailpassed.com/errors/yahoo-553-5-7-1-bl21

## Yahoo 554 5.7.9: “message not accepted for policy reasons”

**Provider:** Yahoo / AOL · **Codes:** 554 5.7.9

**Short answer:** Yahoo didn't accept the message for policy reasons. Usually it failed authentication (no SPF or DKIM pass, or a DMARC failure) or broke Yahoo's sender rules. Fix SPF, DKIM and DMARC for the domain in your From address, then resend.

**Looks like:**
    554 5.7.9 Message not accepted for policy reasons. See https://postmaster.yahooinc.com/error-codes

**Markers:** 5.7.9 MESSAGE NOT ACCEPTED FOR POLICY REASONS

**Meaning:** 554 is a permanent rejection. Yahoo asks every sender for SPF or DKIM, and bulk senders for SPF, DKIM and a DMARC policy. It rejects messages where the domain requires DKIM but the signature is missing or invalid, or where DMARC fails under a reject policy. Malformed headers can trigger it too (an undecodable From or Subject, a domain without DNS records).

**Fix:**
1. Check the From domain below: SPF and DKIM should pass and DMARC should be published.
2. If the mail came through a newsletter or CRM tool, enable its custom DKIM so it signs with your domain.
3. Sending as someone else's domain (for example a Yahoo or Gmail address from your own server) always fails DMARC: send from your own domain instead.
4. Mailing lists should put the list address in From and the original sender in Reply-To; forwarders should use ARC.
5. Make sure the domain in MAIL FROM and in the From header has A or MX records, and headers are valid UTF-8.

Full page: https://mailpassed.com/errors/yahoo-554-5-7-9

## Gmail 421 4.7.28: “unusual rate of unsolicited email” (rate limited)

**Provider:** Gmail · **Codes:** 421 4.7.28

**Short answer:** Gmail is rate limiting mail from your IP, domain or links because it sees an unusual amount of unsolicited (spam-like) mail from them. It is temporary: mail is retried. Fix: cut spam complaints, authenticate properly and slow down, then watch your spam rate in Google Postmaster Tools.

**Looks like:**
    421-4.7.28 Gmail has detected an unusual rate of unsolicited email originating from your IP address. To protect our users from spam, mail sent from your IP address has been temporarily rate limited.
    421-4.7.28 Gmail has detected an unusual rate of unsolicited email originating from your DKIM domain yourdomain.com …

**Markers:** 4.7.28, 5.7.28

**Meaning:** Google's list has several 4.7.28 variants, one each for an IP address, an IP netblock, a DKIM domain, an SPF domain, a URL domain in the message, and repeated Message-IDs. The variant tells you **what** is being throttled. The cause is the same: recipients see too much of this mail as spam, or volume jumped suddenly. A 421 is temporary, but if it keeps up, mail starts going to spam or bouncing.

**Fix:**
1. Read which variant you got: IP (server or shared pool), DKIM or SPF domain (your domain's reputation), or URL domain (a link in the mail, often a shortener or tracking domain with a bad reputation).
2. Check the domain below: SPF, DKIM and DMARC must pass, with DKIM signed by your own domain.
3. Sign up for Google Postmaster Tools and keep the user-reported spam rate below 0.3% (Google's bulk-sender rule; under 0.1% is safer).
4. Mail only people who opted in, add one-click unsubscribe to marketing mail, and drop addresses that never open.
5. Spread big sends out and ramp up volume on new IPs or domains gradually.
6. Check for spam leaving your server: a compromised account or website form is a common cause of a sudden 4.7.28.

Full page: https://mailpassed.com/errors/gmail-421-4-7-28

## Microsoft 365 550 5.7.708: “Access denied, traffic not accepted from this IP”

**Provider:** Microsoft 365 · **Codes:** 550 5.7.708

**Short answer:** Exchange Online refused to send your mail because the outgoing IP it would use is treated as low reputation. This mostly hits new or trial Microsoft 365 tenants. Fix: make sure the mailbox has an Exchange Online licence and your domain is verified, then ask Microsoft support for an exception. Nothing in your DNS causes it.

**Looks like:**
    550 5.7.708 Service unavailable. Access denied, traffic not accepted from this IP.

**Markers:** 5.7.708

**Meaning:** The bounce comes from your own Microsoft 365 tenant, not from the recipient. Exchange Online keeps new and trial tenants on a separate, low-reputation sending pool and can stop their outbound mail. It can also happen after the tenant sent suspicious mail, for example from a compromised account.

**Fix:**
1. Check that the sending mailbox has an Exchange Online licence assigned and that your custom domain is verified in the Microsoft 365 admin center (trial tenants are the usual case).
2. Look for compromise: unknown inbox rules, unexpected forwarding or a spike in sent mail. Reset passwords and turn on MFA before anything else.
3. Open a service request in the Microsoft 365 admin center. Include the full bounce, the sending mailbox and how the mail was sent (Outlook, Outlook on the web, a connector or an app), and ask for the IP exception.
4. If the mail is sent by a third-party app through your tenant, that app's provider may need to raise it with Microsoft as well.
5. Use the time to get SPF, DKIM and DMARC right for the domain (check it below), so mail lands in the inbox once Microsoft lifts the block.

Full page: https://mailpassed.com/errors/microsoft365-550-5-7-708

## Microsoft 365 535 5.7.139: “Authentication unsuccessful” – SMTP AUTH and Basic auth disabled

**Provider:** Microsoft 365 · **Codes:** 535 5.7.139 / 530 5.7.57

**Short answer:** Microsoft 365 refused the username-and-password (Basic auth) login that your printer, app or script uses on smtp.office365.com. Fix: switch it to OAuth, or send through a relay connector or an email service. An admin can re-enable SMTP AUTH for one mailbox as a stopgap.

**Looks like:**
    535 5.7.139 Authentication unsuccessful, SmtpClientAuthentication is disabled for the Tenant. Visit https://aka.ms/smtp_auth_disabled for more information.
    5.7.57 Client not authenticated to send mail. Error: 535 5.7.139 Authentication unsuccessful, basic authentication is disabled.
    535 5.7.139 Authentication unsuccessful, the request did not meet the criteria to be authenticated successfully. Contact your administrator.
    530 5.7.57 Client not authenticated to send mail …

**Markers:** 5.7.139, 5.7.57, SMTPCLIENTAUTHENTICATION, 5.7.3 AUTHENTICATION UNSUCCESSFUL, CLIENT SUBMISSION, BASIC AUTHENTICATION IS NOT SUPPORTED

**Meaning:** Your device or app signs in to `smtp.office365.com` (port 587) with a username and password, and Exchange Online refused the login. The text after 5.7.139 says why: SMTP AUTH is turned off for the whole organisation or for that mailbox, or Basic auth is blocked by security defaults, a Conditional Access policy, an authentication policy or MFA on the account. **5.7.57** and **535 5.7.3** are the same failed login as the app reports it. Microsoft's current plan: Basic auth for SMTP AUTH is **switched off by default for existing tenants at the end of December 2026**, admins can still turn it back on, tenants created after that won't have it by default, and the final removal date will be announced in the second half of 2027. Personal Outlook.com and Hotmail accounts lost Basic auth on 16 September 2024; there is no switch to bring it back.

**Fix:**
1. Find what still signs in with a password: in the Exchange admin center go to **Reports → Mail flow → SMTP AUTH clients report**. The **Authentication Protocol** column shows Basic Auth or Modern Auth for each sender.
2. Best fix: switch the app to OAuth 2.0. Many apps, plugins and newer printers have a “Microsoft 365 / Outlook” sign-in option (XOAUTH2). For your own code, register an app in Microsoft Entra with the `SMTP.Send` permission (`SMTP.SendAsApp` for unattended services), or send with Microsoft Graph `sendMail`. SMTP AUTH must still be enabled on the sending mailbox.
3. Device can't do OAuth and has to mail outside addresses? Create an SMTP relay connector: Exchange admin center → **Mail flow → Connectors → Add a connector**, **Connection from: Your organization's email server**, authenticated by a TLS certificate or a static public IP that only you use. Point the device at your MX host (`yourdomain-com.mail.protection.outlook.com`) on port 25 and add the IP to SPF with the SPF generator (https://mailpassed.com/spf-generator).
4. Only mailing colleagues? **High Volume Email** (`smtp.hve.mx.microsoft`, port 587) still accepts an HVE account's username and password but delivers to internal recipients only, and is billed per recipient. Direct send (MX host, no login) is internal-only too; Microsoft recommends it only as a last resort, says it is working on turning it off by default, and tenants with `RejectDirectSend` on bounce it with `550 5.7.68`.
5. Or give the device its own SMTP login at an email sending service (Amazon SES, Postmark, Brevo, Mailgun) and publish that service's SPF and DKIM records – see the setup guides (https://mailpassed.com/guides). Microsoft is retiring Azure Communication Services Email on 30 September 2028, so don't start new setups on it.
6. Stopgap while you migrate: an admin can enable SMTP AUTH for just the device's mailbox – Microsoft 365 admin center → **Users → Active users** → the user → **Mail → Manage email apps → Authenticated SMTP**, or `Set-CASMailbox -Identity scanner@yourdomain.com -SmtpClientAuthenticationDisabled $false`. It doesn't work while security defaults are on or a policy blocks Basic auth, and it will stop working when Microsoft removes Basic auth for good.
7. App passwords don't get around this: they are Basic auth too. Old devices also fail if they can't do TLS 1.2, so update the firmware. Outlook.com and Hotmail users need an app that offers an Outlook.com (OAuth) sign-in.

Full page: https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth

## Microsoft 365 550 5.7.233 / 5.7.232: tenant exceeded its daily limit for external recipients (TERRL)

**Provider:** Microsoft 365 · **Codes:** 550 5.7.233 / 5.7.232

**Short answer:** Your whole Microsoft 365 organisation sent to more outside recipients in the last 24 hours than its quota allows, so external mail is blocked until the count drops. Fix: wait, look for a hacked mailbox or a bulk send, and move newsletters and app mail to an email sending service.

**Looks like:**
    550 5.7.233 Your message can't be sent because your tenant exceeded its daily limit for sending email to external recipients (tenant external recipient rate limit).
    550 5.7.232 Your message can't be sent because your trial tenant has exceeded its daily limit …

**Markers:** 5.7.233, 5.7.232

**Meaning:** Exchange Online caps how many external recipients (addresses outside your own domains) the **whole organisation** can send to in a rolling 24 hours: the Tenant External Recipient Rate Limit (TERRL). Once it is reached, everyone gets this bounce for outside recipients; internal mail keeps working. The quota grows with purchased licences – `500 × licences^0.7 + 9,500`, so 10,000 a day for one licence, about 12,000 for 10 and 22,000 for 100 – and each external member of a distribution group counts separately. Trial tenants get **5.7.232**. In a rollout due to finish by the end of November 2026, Microsoft is cutting the trial quota from 5,000 to 500 a day and giving tenants younger than 31 days 10% of the normal quota (25% up to 60 days).

**Fix:**
1. See where you stand: Exchange admin center → **Reports → Mail flow → Tenant Outbound External Recipients** shows the daily quota, current usage, blocked recipients and whether enforcement is on.
2. Wait: the window is rolling, so external mail flows again by itself once the last 24 hours drop below the quota. Bounced messages are not retried – resend the important ones after that.
3. Rule out a hacked account: a sudden jump usually means a mailbox is sending spam. Check **Restricted entities** and the **Email sending limit exceeded** alert in the Microsoft Defender portal and run a message trace; reset the password and turn on MFA for anything suspicious.
4. Find the big senders: mail merges, newsletters, CRM or app notifications sent from Microsoft 365 mailboxes, and messages to distribution groups with many outside members.
5. Move bulk and automated mail off Exchange Online to an email sending service (Amazon SES, Postmark, Brevo), ideally from a subdomain, and set up its SPF and DKIM with the setup guides (https://mailpassed.com/guides). Microsoft's pages still mention Azure Communication Services Email, but it is being retired on 30 September 2028.
6. Need more headroom on Microsoft 365 itself? The quota only rises with purchased licences; a trial tenant moves to the normal formula once it has paid licences.
7. After moving senders, check your domain below: every new service has to be in your SPF record and sign DKIM with your domain.

Full page: https://mailpassed.com/errors/microsoft365-550-5-7-233

## Microsoft 365 550 5.7.236: daily limit for external recipients from onmicrosoft.com domains

**Provider:** Microsoft 365 · **Codes:** 550 5.7.236

**Short answer:** You are sending from your tenant's free onmicrosoft.com address, and Microsoft allows those only 100 external recipients per organisation in 24 hours. Fix: add your own domain to Microsoft 365, make it the default and every user's primary address, and set up SPF, DKIM and DMARC.

**Looks like:**
    550 5.7.236 Your message can't be sent because your tenant has exceeded its daily limit for sending email to external recipients from your tenant's onmicrosoft.com domains. For more information see https://aka.ms/EXONdrs.

**Markers:** 5.7.236

**Meaning:** Every Microsoft 365 organisation gets a free address like `yourcompany.onmicrosoft.com`. Because spammers abuse these, Microsoft throttles outbound mail from them: at most **100 external recipients per organisation in a rolling 24 hours**, counted after distribution groups are expanded. Mail between colleagues and incoming mail are not affected. The limit is fully rolled out (2026), so organisations that never added their own domain now hit it after a few messages, and waiting only frees up the next 100.

**Fix:**
1. Check the From address of the bounced message. If it ends in `.onmicrosoft.com`, that's the cause. Admins can run a message trace in the Exchange admin center to see which mailboxes, printers and apps still send from it.
2. Add a domain you own: Microsoft 365 admin center → **Settings → Domains → + Add domain**, verify it with the TXT record shown, then add the DNS records it lists. No domain yet? Buy one from any registrar first.
3. Make it the default: on the same **Domains** page select your domain and choose **Set as default**. This only changes new accounts.
4. Move existing users: **Users → Active users** → the user → **Manage username and email**, and pick your domain. Do the same for shared mailboxes, and change the sender address and username on printers and apps.
5. Set up authentication for the new domain so mail reaches the inbox: SPF with `include:spf.protection.outlook.com`, DKIM turned on in Microsoft 365, and a DMARC record. Follow the Microsoft 365 guide (https://mailpassed.com/guides/microsoft-365) and the DMARC generator (https://mailpassed.com/dmarc-generator), then check the domain below.
6. Microsoft's documented fix is a custom domain. Until users send from it, external mail from the onmicrosoft.com address keeps bouncing after every 100 recipients.

Full page: https://mailpassed.com/errors/microsoft365-550-5-7-236

## Microsoft 550 5.7.520: “Your organization does not allow external forwarding” / AS(4810)

**Provider:** Microsoft 365 / Outlook · **Codes:** 550 5.7.520

**Short answer:** Usually your Microsoft 365 organisation blocks automatic forwarding to outside addresses, so a forwarding rule bounced. Fix: an admin sets Automatic forwarding rules to On in an outbound spam policy for those users. With AS(4810), Outlook.com has restricted your account from sending.

**Looks like:**
    550 5.7.520 Access denied, Your organization does not allow external forwarding. Please contact your administrator for further assistance. AS(7555)
    550 5.7.520 Message blocked because it contains content identified as spam. AS(4810)

**Markers:** AS(7555), AS(4810), 5.7.520

**Meaning:** Two different errors share this code. **AS(7555)** is the common one, in Microsoft 365: a mailbox tried to forward mail automatically to an outside address – through an Inbox rule or forwarding set by an admin – and the outbound spam policy blocks external forwarding. Since 2021 the default setting, **Automatic - System-controlled**, means “off” unless your organisation was already relying on it, because attackers use forwarding rules to siphon off mail. Forwarding between colleagues is not affected. **AS(4810)** shows up on personal Outlook.com and Hotmail accounts that Microsoft's outbound filter has restricted from sending. Microsoft publishes no page for it, and only Outlook.com support can lift it.

**Fix:**
1. **AS(7555), admins:** Microsoft Defender portal → **Email & collaboration → Policies & rules → Threat policies → Anti-spam**. Create an outbound policy for just the users who need forwarding (or edit **Anti-spam outbound policy (Default)**) and set **Automatic forwarding rules** to **On - Forwarding is enabled**.
2. Prefer a policy for named users over switching forwarding on for everyone: forwarding to outside addresses is a classic sign of a hacked mailbox.
3. Still bounced? A block elsewhere wins. Check **Remote domains** (automatic forwarding allowed for that domain) and any mail flow rule that blocks auto-forwarded messages in the Exchange admin center.
4. Nobody set up forwarding? Treat it as a compromise: check the user's Inbox rules and mailbox forwarding, reset the password and turn on MFA. Exchange admin center → **Reports → Mail flow → Auto forwarded messages report** shows who forwards where.
5. **AS(4810), personal Outlook.com or Hotmail:** there is no setting to change. Check the account for signs of a hack (mail you didn't send, unknown rules or sign-ins) and secure it, then contact Microsoft support – from another account if you can't sign in – and ask for a review of the outbound sending restriction. Keep the account's phone number and alternate email up to date.
6. If that account sends business mail, move it to your own domain on a business mailbox with SPF, DKIM and DMARC (check it below). Personal accounts have fixed sending limits and no admin who can lift a block.

Full page: https://mailpassed.com/errors/microsoft-550-5-7-520

## Microsoft 365 550 5.1.8: “Access denied, bad outbound sender” AS(42004)

**Provider:** Microsoft 365 · **Codes:** 550 5.1.8

**Short answer:** Microsoft 365 has blocked this mailbox from sending because it sent too much mail or mail that looked like spam, usually after the account was hacked. Fix: an admin secures the account (password, MFA, rules, forwarding), then unblocks it under Restricted entities in the Microsoft Defender portal.

**Looks like:**
    Your message couldn't be delivered because you weren't recognized as a valid sender. The most common reason for this is that your email address is suspected of sending spam and it's no longer allowed to send email. …
    Remote Server returned '550 5.1.8 Access denied, bad outbound sender AS(42004)'

**Markers:** AS(42004), BAD OUTBOUND SENDER

**Meaning:** The mailbox went over Exchange Online's sending limits (10,000 recipients a day, or lower limits set in your outbound spam policy) or sent mail that Microsoft scored as spam, so it is restricted from sending. It still receives mail but can't send anything, to anyone, until the block is lifted. Microsoft treats this as a sign of a **compromised account**; the other common cause is cold outreach or bulk mail sent from a normal mailbox. Once it is on the **Restricted entities** list, only an admin of your Microsoft 365 organisation can remove it.

**Fix:**
1. **Users:** you can't fix this yourself. Tell your Microsoft 365 admin, and look in Sent Items for mail you didn't send – that means the account was hacked.
2. **Admins, secure the account first:** reset the password (don't email the new one to the user), turn on MFA, sign the user out of all sessions, and remove MFA methods, app consents and admin roles you don't recognise.
3. Check rules and forwarding: look for Inbox rules that forward, redirect or delete mail, and for forwarding to outside addresses (`Get-InboxRule -Mailbox user@yourdomain.com -IncludeHidden` and `Get-Mailbox -Identity user@yourdomain.com | Format-List Forwarding*Address,DeliverTo*`). Remove anything the user didn't create.
4. Unblock: Microsoft Defender portal → **Email & collaboration → Review → Restricted entities**, select the mailbox, choose **Unblock**, work through the wizard and **Submit** – or run `Remove-BlockedSenderAddress -SenderAddress user@yourdomain.com`. Restrictions usually lift within an hour, at most 24 hours.
5. Mailbox not on that page? Then it hit a sending limit, and sending resumes when the 24-hour window clears; with the default outbound spam policy action the block lasts until the next day (UTC).
6. Blocked again after cold outreach or a newsletter? Microsoft 365 mailboxes aren't meant for bulk mail. Send campaigns through an email sending service from a separate domain or subdomain with SPF and DKIM set up (setup guides (https://mailpassed.com/guides)).
7. Keep the **User restricted from sending email** alert on (Defender portal → **Email & collaboration → Policies & rules → Alert policy**) so you hear about the next block straight away.

Full page: https://mailpassed.com/errors/microsoft365-550-5-1-8

## Gmail 550 5.7.1: “very low reputation of the sending domain” (or IP)

**Provider:** Gmail · **Codes:** 550 5.7.1 / 421 4.7.0

**Short answer:** Gmail blocked the message because your sending domain (or IP) has a very low reputation with Gmail, usually after spam complaints or past spam. Passing SPF and DKIM doesn't override it. Fix: stop the source, keep the spam rate in Postmaster Tools under 0.3%, clean your list and ramp up slowly.

**Looks like:**
    550-5.7.1 This message is likely suspicious due to the very low reputation of the sending domain. To best protect our users from spam, the message has been blocked.
    550-5.7.1 [203.0.113.10] Gmail has detected that this message is likely suspicious due to the very low reputation of the sending IP address. …
    421-4.7.0 This message is suspicious due to the very low reputation of the sending IP address. To protect our users from spam, email sent from your IP address has been temporarily rate limited.
    550 5.7.350 Remote server returned message detected as spam -> 550 5.7.1 … very low reputation of the sending domain …

**Markers:** VERY LOW REPUTATION, LOW REPUTATION OF THE SENDING

**Meaning:** Gmail keeps a reputation for every sending domain and IP, built from how its users treat your mail: spam reports, mail to people who never signed up, and spam sent in the past (often from a hacked mailbox or website form). At “very low”, Gmail blocks some of your messages outright, even when SPF, DKIM and DMARC pass. The wording says what is at fault: **sending domain** (your domain's own history, which follows you to any server) or **sending IP address** (the server or shared pool you send through). **421 4.7.0** is the temporary version, and your server retries. Microsoft 365 and Outlook.com senders see the same text inside `550 5.7.350 Remote server returned message detected as spam`. If Gmail objects to the message itself instead, see “likely unsolicited email” (https://mailpassed.com/errors/gmail-550-5-7-1-unsolicited).

**Fix:**
1. Find and stop the source first: a hacked mailbox, website form or script sending spam, or a campaign to a bought or old list. Pause bulk sends until it's fixed.
2. Check the domain below: SPF, DKIM (signed with your own domain) and DMARC must pass. That alone won't lift the block, but Google gives no help to senders who miss its requirements.
3. Set up Google Postmaster Tools (`postmaster.google.com`) for your domain and keep the user-reported spam rate below 0.3%; Google recommends under 0.1%. Watch the spam rate and compliance status: Google is retiring the old domain and IP reputation charts.
4. Mail only people who opted in: confirm new sign-ups, never buy lists, and remove recipients who don't open your mail.
5. Give marketing mail one-click unsubscribe (`List-Unsubscribe` and `List-Unsubscribe-Post` headers) and honour unsubscribes within 48 hours.
6. Restart slowly: send to your most engaged recipients first, keep the daily volume steady and raise it gradually. If bounces continue, send less until they stop.
7. IP variant: on a shared IP (newsletter tool, hosting provider), ask the provider about its sending pool; on your own server, check blocklists and reverse DNS (see 5.7.25 (https://mailpassed.com/errors/gmail-550-5-7-25)).
8. There is no form to remove a reputation block; it recovers as Gmail sees clean mail. Once you meet every requirement (bulk senders: spam rate under 0.3% for 7 days in a row), use **Report delivery issue** in Postmaster Tools.

Full page: https://mailpassed.com/errors/gmail-550-5-7-1-low-reputation

## Gmail 550 5.7.1: “This message is likely unsolicited email”

**Provider:** Gmail · **Codes:** 550 5.7.1 / 421 4.7.0

**Short answer:** Gmail's spam filter judged this message to be unwanted mail and refused it. It is about the message and how you send it (content, links, recipients who didn't ask, sudden volume), not one DNS record. Fix: send only to opted-in people, clean up links and content, and pass SPF, DKIM and DMARC.

**Looks like:**
    550-5.7.1 This message is likely unsolicited email. To reduce the amount of spam sent to Gmail, this message has been blocked.
    550-5.7.1 [203.0.113.10] Gmail has detected that this message is likely unsolicited mail. … https://support.google.com/mail/?p=UnsolicitedMessageError
    421-4.7.0 This message is suspicious due to the nature of the content or the links within. To best protect our users from spam, the message has been blocked.

**Markers:** LIKELY UNSOLICITED, UNSOLICITEDMESSAGEERROR, NATURE OF THE CONTENT

**Meaning:** Gmail's filter scored this particular message as spam and rejected it instead of delivering it to the spam folder. Google says it blocks only some mail that looks like spam, judging the wording, the links and the sender's history. Typical triggers: bulk or cold mail to people who never signed up, links to domains with a bad reputation, hidden text, a sudden jump in volume, or a forwarder passing spam along. The **421 4.7.0** “nature of the content or the links” variant points straight at the message body. If the bounce blames your domain or IP instead, see very low reputation (https://mailpassed.com/errors/gmail-550-5-7-1-low-reputation); for “unusual rate of unsolicited email”, see 421 4.7.28 (https://mailpassed.com/errors/gmail-421-4-7-28).

**Fix:**
1. Send only to people who asked for your mail: confirm new sign-ups, never buy lists, and remove recipients who don't open.
2. Check every link: it should be visible and easy to understand, and its domain must not be flagged as unsafe by Google Safe Browsing (`transparencyreport.google.com/safe-browsing`).
3. Don't hide text with HTML or CSS, and don't mix promotions into receipts or other transactional mail.
4. Check the domain below: SPF, DKIM with your own domain and DMARC should pass, and marketing mail needs one-click unsubscribe.
5. Keep volume steady: raise it gradually, and send less while bounces continue.
6. Watch the spam rate in Google Postmaster Tools and keep it under 0.3% (ideally under 0.1%).
7. Forwarding mail into Gmail? Filter spam before it is forwarded: Gmail blocks forwarded spam and holds it against the forwarding domain (see forwarded email rejected (https://mailpassed.com/errors/forwarded-email-rejected)).
8. A 550 rejection is final, so nothing is retried. Resend the important messages once the cause is fixed.

Full page: https://mailpassed.com/errors/gmail-550-5-7-1-unsolicited

## Gmail 550 5.4.5: “Daily user sending limit exceeded”

**Provider:** Gmail / Google Workspace · **Codes:** 550 5.4.5

**Short answer:** Your own Gmail or Google Workspace account hit its daily sending limit, so Google stopped sending for you; no recipient blocked you. Sending resumes within 24 hours. For newsletters or other bulk mail, use a sending service instead of a mailbox.

**Looks like:**
    550 5.4.5 Daily user sending limit exceeded. For more information on Gmail sending limits, go to …
    550 5.4.5 Daily user sending quota exceeded. … - gsmtp
    You have reached a limit for sending mail.
    550 5.4.5 Daily SMTP relay limit exceeded for user.

**Markers:** DAILY USER SENDING, GMAIL SENDING LIMIT, LIMIT FOR SENDING MAIL, DAILY SMTP RELAY

**Meaning:** The block is on your own account: once it passes a daily limit, Google stops it from sending for up to 24 hours, counted over a rolling 24 hours. Personal Gmail: about 500 emails a day, or 500 recipients in one message. Google Workspace: 2,000 messages a day per user (500 on trial accounts, 1,500 with mail merge), 10,000 recipients a day in total, 2,000 different external recipients a day, and 100 recipients per message when an app sends over SMTP. Every To, Cc and Bcc address counts, and so does mail sent from aliases, by delegates or by the vacation responder. Google notes the limits can change without notice. The **SMTP relay** lines refer to the separate limit of Workspace's smtp-relay service.

**Fix:**
1. Wait: sending comes back by itself within 24 hours (Google says 1 to 24 hours for personal Gmail). You can still receive mail meanwhile.
2. Pause scripts, mail merges and apps that send through the account until the limit resets. Then spread the remaining mail over several days.
3. Count what you really send: a message to 50 people uses 50 of the recipient limit, and aliases, delegates and auto-replies share the same limit.
4. Workspace trial? Trial accounts keep the lower limits until the domain has paid at least US$100 in total, which can take up to 75 days.
5. For newsletters, announcements or app notifications, use a sending service instead of a mailbox, for example Amazon SES (https://mailpassed.com/guides/amazon-ses), Postmark (https://mailpassed.com/guides/postmark) or Brevo (https://mailpassed.com/guides/brevo). For a small group of people, Google suggests a Google Group.
6. When you move bulk mail to a service, add it to SPF (SPF generator (https://mailpassed.com/spf-generator)) and turn on its DKIM with your domain, or Gmail recipients will reject it with 5.7.26 (https://mailpassed.com/errors/gmail-550-5-7-26) instead.
7. SMTP relay variant: your Workspace admin can see the relay's sending limits in the SMTP relay service settings.

Full page: https://mailpassed.com/errors/gmail-550-5-4-5-sending-limit

## Forwarded email rejected: why forwarding fails SPF and DMARC

**Provider:** Gmail / Yahoo / iCloud · **Codes:** 550 5.7.26 / 554 5.7.9 / 554 5.7.1 [BS01]

**Short answer:** Forwarding sends the message on from a different server, so SPF fails, and DMARC then passes only if the original DKIM signature survives. Fix: the original sender DKIM-signs with their own domain; the forwarder rewrites the sender (SRS), adds ARC and doesn't change the message.

**Looks like:**
    550-5.7.26 Unauthenticated email from example.com is not accepted due to domain's DMARC policy. …
    550-5.7.26 The MAIL FROM domain [example.com] has an SPF record with a hard fail policy (-all) …
    554 5.7.9 Message not accepted for policy reasons. See https://postmaster.yahooinc.com/error-codes
    554 5.7.1 [BS01] Message rejected due to local policy. Please visit https://support.apple.com/…

**Markers:** [BS01]

**Meaning:** SPF checks the server that delivers the message. After a forward that is the forwarding server, which isn't in the original sender's SPF record, so SPF fails. Google says forwarded messages often fail SPF and aren't automatically trusted because they passed before. DMARC can then pass only through the original DKIM signature, and that breaks when the forwarder changes the message (footer, subject tag, re-encoding) or when the sender never signed with its own domain. If the sender's domain has `p=reject`, Gmail, Yahoo and iCloud reject the forwarded copy. If you forward from Gmail, the failure comes back as a “Message blocked” notice with the other server's error under the technical details. Google, Yahoo and Apple all point forwarders to ARC, which carries the original authentication results along. Apple doesn't publish what **[BS01]** means: it is a “local policy” rejection that can also hit mail that was never forwarded.

**Fix:**
1. **If you own the sending domain:** make every service sign with DKIM for your own domain, because DKIM is what survives a forward. Check it below and look for `DKIM: PASS` with your domain in Gmail's “Show original”.
2. With `p=reject`, every forward that breaks DKIM bounces. Move to reject only once DKIM passes for all your senders (DMARC generator (https://mailpassed.com/dmarc-generator)).
3. **If you run the forwarder** (your own server, cPanel, a domain forwarder): rewrite the envelope sender to your forwarding domain (SRS), include the forwarding servers in your SPF, and add ARC headers if your mail software supports it.
4. Don't change forwarded messages: footers, subject tags and re-encoding break the original DKIM signature.
5. Filter spam before you forward, and don't forward the spam folder. Gmail blocks forwarded spam and counts it against the forwarding domain.
6. **If you just forward a mailbox** to Gmail, Yahoo or iCloud: use a forwarder that rewrites the sender (SRS), such as Cloudflare Email Routing (https://mailpassed.com/guides/cloudflare-email-routing), or add the account to the Gmail app over IMAP instead, as Google suggests now that Gmail's POP import is ending (January 2027 for existing users).
7. **iCloud [BS01] that keeps happening:** email `icloudadmin@apple.com` with your company name, domain, sending IPs, the full SMTP error and when it started.

Full page: https://mailpassed.com/errors/forwarded-email-rejected

## Validity Heatwave blocklist: “synthetic reputation warming observed”

**Provider:** Validity Heatwave · **Codes:** domain blocklist

**Short answer:** Your domain is on Validity's Heatwave blocklist, which flags domains seen in synthetic warm-up networks or cold outreach. Check it at lookup.validity.tools. Listings don't expire and stopping warm-up doesn't remove them: only a review that finds the listing was wrong does.

**Looks like:**
    Listed · Warming: Synthetic reputation warming observed
    DNS answer 127.0.95.2 · listing score 95/100 · does not expire merely because the activity later stops

**Markers:** HEATWAVE, VALIDITY.TOOLS, SYNTHETIC REPUTATION WARMING

**Meaning:** Heatwave is a domain blocklist that Validity launched on 3 September 2026. It lists sending domains, not IP addresses, that Validity's data shows taking part in **synthetic warming**: warm-up tools that trade automated messages and fake engagement between mailboxes they control. **Active outreach** is added when the domain is also seen sending live cold campaigns, and **pre-warming** marks registered domains of the same operator before they send. Validity names Comcast, Proofpoint, Spamhaus, SURBL and some ESPs as users or evaluators, and each decides for itself whether to filter mail from listed domains. So there is no standard bounce: you usually notice it as mail going to spam or an email service questioning your account. A warming listing is a permanent record of what was observed.

**Fix:**
1. Look up the exact domain at `lookup.validity.tools` (free, 100 checks a day). Subdomains are listed separately, and searching your main domain can show related domains tied to it.
2. Stop warm-up services on every domain you send from. Stopping doesn't remove a listing, but domains that keep warming can be listed next.
3. If the listing is wrong (an agency, vendor or ex-employee used the domain without approval, or it was impersonated or compromised), request a review at `lookup.validity.tools/delist`.
4. The request must come from your own named address at the listed domain (not info@, sales@ or support@) and explain why the original listing was wrong. Validity answers in 2–5 business days; if it agrees, the listing is removed.
5. If the listing is accurate, a review won't remove it, and Validity says changing how you send doesn't undo an accurate listing. From now on, send from that domain only to people who asked for your mail.
6. Check the domain below: SPF, DKIM and DMARC still have to pass for any of your mail to reach the inbox.

Full page: https://mailpassed.com/errors/heatwave-blocklist

## Microsoft 365 5.4.8: “MX hosts of the domain failed MTA-STS validation”

**Provider:** Microsoft 365 · **Codes:** 5.4.8

**Short answer:** Microsoft 365 refused to deliver because the recipient domain's MTA-STS policy (mode: enforce) doesn't list the MX host the domain actually uses. The fix is on the recipient's side: add the MX host to the policy's mx: lines (or switch it to testing), then change the policy id.

**Looks like:**
    5.4.8 MX hosts of 'contoso.com' failed MTA-STS validation

**Markers:** MX HOSTS OF

**Meaning:** Exchange Online checks the MTA-STS policy of every domain it sends to. When the policy is in **enforce** mode and the recipient's live MX host isn't covered by any `mx:` line, Microsoft returns this NDR instead of delivering. Typical causes: the domain moved its MX (for example to Microsoft's new `*.mx.microsoft` hosts, or to a new mail filter) without updating the policy, or the policy uses a wildcard that covers the wrong number of labels (`*.mx.microsoft` doesn't match `contoso-com.o-v1.mx.microsoft`).

**Fix:**
1. If it's **your** domain that people can't reach: open `https://mta-sts.yourdomain/.well-known/mta-sts.txt` and compare its `mx:` lines with your MX records – run the checker below, it does this the way senders do.
2. Add every MX host you use (exact name, or a wildcard that covers exactly one label), or set `mode: testing` while you fix it.
3. Change the `id=` in your `_mta-sts` TXT record so senders fetch the new policy; until then they keep the cached one for up to `max_age`.
4. If you're the sender: forward the bounce to the recipient's IT – only they can fix their policy. More: the mx.microsoft trap (https://mailpassed.com/research/microsoft-365-mta-sts-mx-microsoft-trap).

Full page: https://mailpassed.com/errors/microsoft-5-4-8-mta-sts-validation

## Microsoft 365 5.7.5: “Remote certificate failed MTA-STS validation”

**Provider:** Microsoft 365 · **Codes:** 5.7.5

**Short answer:** Microsoft 365 refused to deliver because the recipient's mail server certificate failed the MTA-STS check: it doesn't chain to a trusted root CA, or its name doesn't match the MX host in the policy. The recipient has to fix the certificate (or set the policy to testing).

**Looks like:**
    5.7.5 Remote certificate failed MTA-STS validation. Reason: {validityStatus}

**Markers:** REMOTE CERTIFICATE FAILED MTA-STS VALIDATION

**Meaning:** With an MTA-STS policy in enforce mode, the receiving mail server must present a certificate that chains to a trusted public CA and whose Common Name or Subject Alternative Name matches the MX host named in the policy. Self-signed, expired or wrong-name certificates on the MX make Exchange Online refuse delivery with this NDR.

**Fix:**
1. If it's your domain: check the certificate on each MX host (expiry, chain, the names it covers).
2. Renew or replace it with one from a public CA that covers the exact MX hostname; automate renewal (certificate lifetimes shrink to 200 days in 2026 and 100 days in 2027 – details (https://mailpassed.com/changes/mail-server-certificates-shorter-lifetime)).
3. While you fix it, set the policy to `mode: testing` and change the policy id so mail flows again.
4. If you're the sender: forward the bounce to the recipient's IT.

Full page: https://mailpassed.com/errors/microsoft-5-7-5-mta-sts-certificate

## Microsoft 365 550 5.7.64: “TenantAttribution; Relay Access Denied”

**Provider:** Microsoft 365 · **Codes:** 550 5.7.64 / 4.4.62

**Short answer:** Microsoft 365 couldn't match the server or printer that sent the message to your organization, so it refused to relay it outside. Fix: update your inbound (relay) connector with the device's current public IP or its renewed certificate, and make sure the sender's domain is verified in your tenant.

**Looks like:**
    550 5.7.64 TenantAttribution; Relay Access Denied SMTP.
    4.4.62 Mail sent to the wrong Office 365 region
    550 5.7.64 TenantAttribution; Relay Access Denied … ATT36

**Markers:** TENANTATTRIBUTION; RELAY, WRONG OFFICE 365 REGION

**Meaning:** Your printer, scanner, app or on-premises mail server sends through your Microsoft 365 MX host (`…mail.protection.outlook.com`) without a login and relies on an inbound connector to prove the mail is yours. Exchange Online looked for a connector that matches the sender's TLS certificate or public IP, found none, and won't relay the message to outside recipients. Microsoft names the usual causes: the office's static IP changed (new ISP, new router, failover line), or the certificate was renewed or replaced with a different name. **4.4.62** is the same problem reported as a deferral. An **ATT36** variant means the domain in the certificate or the sender address isn't a verified domain in your tenant.

**Fix:**
1. Find the public IP the device really sends from: on the same network open a site such as `ifconfig.me`, or read the connecting IP from the message trace in the Exchange admin center.
2. Exchange admin center → **Mail flow → Connectors** → open the connector **from Your organization's email server to Office 365**. If it trusts an IP, replace the old address with the new one. If it trusts a certificate, set the domain to the name in the new certificate (Subject or SAN, e.g. `*.contoso.com`). Save and allow some time to apply.
3. Hybrid Exchange after a certificate renewal: re-run the Hybrid Configuration Wizard and pick the new certificate on the transport certificate page (Microsoft's recommended fix); the send connector log shows which certificate is presented.
4. **ATT36**: add and verify the domain from the certificate or the From address under **Settings → Domains** in the Microsoft 365 admin center.
5. No connector at all yet? Follow the relay connector set-up (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup): MX host, port 25, TLS, no login.
6. Put the new IP in your SPF record (one record: `v=spf1 ip4:203.0.113.10 include:spf.protection.outlook.com -all`, built with the SPF generator (https://mailpassed.com/spf-generator)) so relayed mail doesn't land in Junk.
7. Static IP keeps changing, or the device is in Azure or another cloud? Relay isn't supported there. Use OAuth (`smtp.office365.com`, port 587) or an email sending service with its own login instead.

Full page: https://mailpassed.com/errors/microsoft365-550-5-7-64-tenantattribution

## Microsoft 365 5.7.60: “Client does not have permissions to send as this sender”

**Provider:** Microsoft 365 · **Codes:** 5.7.60

**Short answer:** Your printer or app logs in to smtp.office365.com as one mailbox but sends From a different address, and that mailbox has no Send As permission for it. Fix: make the From address match the login, grant Send As, or switch the device to a relay connector.

**Looks like:**
    5.7.60 SMTP; Client does not have permissions to send as this sender
    550 5.7.60 SMTP; Client does not have permissions to send as this sender

**Markers:** SEND AS THIS SENDER

**Meaning:** With SMTP AUTH (client submission) the device signs in to `smtp.office365.com` with one mailbox's username and password, and Microsoft 365 only accepts mail From that same address, or from an address the mailbox has **Send As** permission for. Microsoft's example: the scanner logs in as `sales@contoso.com` but sends as `salesperson1@contoso.com`. Scan-to-email profiles, print management tools, helpdesk tools and report schedulers often put the user's own address or a `noreply@` address in From, which triggers this. For a device that sends as many different addresses, Microsoft says to use SMTP relay instead.

**Fix:**
1. Easiest: in the device or app, set the From (sender) address to exactly the mailbox you log in with. On printers look for **Device email address**, **From** or **Sender** next to the SMTP server settings; turn off options that use the logged-in user's address as From.
2. Need a different From, such as a shared `scanner@` address? Give the login mailbox **Send As** permission: Exchange admin center → **Recipients → Mailboxes** → the From mailbox → **Delegation → Send as → Edit** → add the login mailbox. In PowerShell: `Add-RecipientPermission -Identity scanner@contoso.com -Trustee printer@contoso.com -AccessRights SendAs`. Allow some time to apply.
3. Many different From addresses (each user scans with their own address as sender)? Microsoft's answer is the relay connector (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup): MX host, port 25, no login, any From in your accepted domains.
4. Password (Basic) sign-in for SMTP AUTH is switched off by default at the end of December 2026 (535 5.7.139 (https://mailpassed.com/errors/microsoft365-535-5-7-139-smtp-auth)), so if you're touching this device anyway, move it to OAuth or relay now.
5. Then send a test to an outside address and check SPF, DKIM and DMARC for the From domain below.

Full page: https://mailpassed.com/errors/microsoft365-5-7-60-send-as-permission

## Microsoft 365 550 5.7.68: “Direct Send not allowed for this organization from unauthorized sources”

**Provider:** Microsoft 365 · **Codes:** 550 5.7.68

**Short answer:** The receiving Microsoft 365 organization has Reject Direct Send turned on, so unauthenticated mail sent straight to its MX host with its own domain as sender is refused. Fix: authenticate the printer or service with a connector (IP or certificate), or move it to OAuth.

**Looks like:**
    550 5.7.68 TenantInboundAttribution; Direct Send not allowed for this organization from unauthorized sources

**Markers:** TENANTINBOUNDATTRIBUTION, DIRECT SEND NOT ALLOWED

**Meaning:** **Direct Send** means a printer, scanner, app or third-party service delivers to your MX host (`…mail.protection.outlook.com`) on port 25 with no login, using your own domain as the sender. Spoofers abuse the same path, so Microsoft added the `RejectDirectSend` setting (preview June 2025, worldwide rollout from September 2025). It is off by default; once an admin turns it on, Direct Send mail that isn't authenticated by a connector bounces with 5.7.68, about 30 minutes after the change. Microsoft also says it is working on turning Direct Send off by default, so devices that rely on it should move now.

**Fix:**
1. Check the setting in Exchange Online PowerShell: `Get-OrganizationConfig | Format-List RejectDirectSend`. `True` means Direct Send from unauthenticated sources is blocked.
2. Printers, scanners and apps in your office: create a connector that recognises them by your static public IP or a TLS certificate – the same connector as in the relay connector set-up (https://mailpassed.com/m365-smtp/microsoft-365-smtp-relay-connector-setup). Mail that matches it is authenticated.
3. Third-party services that send as your domain (ticketing, CRM, scanning or signature services): Microsoft's fix is a **partner** mail flow connector that authenticates them by IP or certificate. Or have the service send through its own authenticated route with SPF and DKIM for your domain (see the setup guides (https://mailpassed.com/guides)).
4. Devices that can sign in: use SMTP AUTH with OAuth on `smtp.office365.com`, port 587. Our device pages (https://mailpassed.com/m365-smtp) show which printers and apps support it.
5. Last resort, if you accept the spoofing risk: `Set-OrganizationConfig -RejectDirectSend $false` turns the block off again.
6. Mail forwarded to you by outside systems without SRS can hit this too; a partner connector for that source fixes it.

Full page: https://mailpassed.com/errors/microsoft365-550-5-7-68-direct-send-rejected

## “Unsuitable certificate purpose” / “Client certificate not trusted”: client certificate without clientAuth

**Provider:** Postfix / Exim / OpenSSL · **Codes:** 421 4.7.1

**Short answer:** A mail server that checks client certificates rejected the sender's certificate because its Extended Key Usage has no TLS Client Authentication (clientAuth). Public CAs stopped adding clientAuth to new server certificates, so this usually starts right after a certificate renewal. Fix it on the receiving side: fingerprint matching or a private CA.

**Looks like:**
    421 4.7.1 <host> Error: Client certificate not trusted
    certificate verification failed for <host>[<ip>]: not designated for use as a client certificate
    SSL verify error: depth=0 error=unsuitable certificate purpose cert=<DN>
    verify error:num=26:unsupported certificate purpose

**Markers:** UNSUITABLE CERTIFICATE PURPOSE, UNSUPPORTED CERTIFICATE PURPOSE, NOT DESIGNATED FOR USE AS A CLIENT CERTIFICATE, CLIENT CERTIFICATE NOT TRUSTED

**Meaning:** The receiving server asks the sender for a TLS client certificate and verifies it for the client purpose. OpenSSL rejects a certificate whose EKU list exists but lacks clientAuth (the same check applies to the intermediate CA). Public CAs are dropping clientAuth: Let's Encrypt's last dual-purpose certificates expired around 6 Oct 2026, Sectigo and DigiCert stop in Feb–Mar 2027, and Chrome requires serverAuth-only certificates from 15 Mar 2027. OpenSSL 3.0.8 and later say "unsuitable certificate purpose", older versions "unsupported certificate purpose". Exchange Online is not affected: Microsoft says certificates used to relay to it don't need clientAuth.

**Fix:**
1. Check the sender's certificate with our certificate check (https://mailpassed.com/certificate-check): if the Extended Key Usage shows serverAuth only, this is the cause.
2. Postfix receiver: replace `permit_tls_all_clientcerts` (any publicly trusted certificate) with `check_ccert_access` or `permit_tls_clientcerts` and the sender's certificate fingerprint. Fingerprint matching works without clientAuth; update it at every renewal. `smtpd_tls_req_ccert = yes` also requires a trusted client certificate.
3. Exim receiver: use `tls_try_verify_hosts` instead of `tls_verify_hosts` and decide in an ACL, or trust a private CA with `tls_verify_certificates`.
4. Long term: issue client certificates from a private CA that both sides trust; public server certificates won't carry clientAuth any more.
5. If you're the sender and can't change the receiver: Sectigo (until 10 Feb 2027) and DigiCert (until 1 Mar 2027) still add clientAuth on request. That buys at most 200 days. Timeline and sources: public certificates lose clientAuth (https://mailpassed.com/changes/tls-client-auth-eku-removed-mail-servers).

Full page: https://mailpassed.com/errors/smtp-unsuitable-certificate-purpose-client-cert

## “SERVFAIL looking up CAA” / “DNSSEC: DNSKEY Missing”: certificate renewal blocked by broken DNSSEC

**Provider:** Let's Encrypt / DigiCert / any public CA · **Codes:** SERVFAIL (CAA / DNSSEC)

**Short answer:** Your certificate authority could not read your domain's DNS because DNSSEC validation failed, or because your DNS host returns an error for CAA queries. Since 15 March 2026 every public CA must treat that as a refusal, so renewal fails at Let's Encrypt, DigiCert and every other CA alike. Fix the DS record at your registrar (or the CAA answer), then retry.

**Looks like:**
    DNS problem: SERVFAIL looking up CAA for example.com - the domain's nameservers may be malfunctioning
    DNS problem: looking up A for www.example.com: DNSSEC: DNSKEY Missing
    DNS problem: looking up TXT for _acme-challenge.example.com: DNSSEC: Signature Expired
    CAA record for example.com prevents issuance
    We detected a DNSSEC validation issue on your domain. Please contact your DNS provider to review the configuration.

**Markers:** SERVFAIL LOOKING UP CAA, WHILE PROCESSING CAA FOR, PREVENTS ISSUANCE, DNSSEC: BOGUS, DNSSEC: DNSKEY MISSING, DNSSEC: SIGNATURE EXPIRED, DNSSEC: SIGNATURE NOT YET VALID, DNSSEC: RRSIGS MISSING, DNSSEC: NSEC MISSING, DNSSEC: INDETERMINATE, DNSSEC: UNSUPPORTED DNSKEY ALGORITHM, DNSSEC: UNSUPPORTED DS DIGEST TYPE, DNSSEC: NO ZONE KEY BIT SET, DNS_SEC_VALIDATION_ERROR, DNSSEC VALIDATION ISSUE ON YOUR DOMAIN

**Meaning:** CA/Browser Forum ballot SC-085 makes every public CA validate DNSSEC back to the IANA root on all CAA and domain-validation lookups from 15 March 2026, and the Baseline Requirements (section 4.2.2.2.5) say a validation error such as SERVFAIL "MUST NOT be treated as permission to issue". Let's Encrypt has validated DNSSEC for years; DigiCert started on 3 March 2026. Domains without DNSSEC (no DS record at the registrar) are not affected. The usual cause is a DS record left at the registrar after moving to a new DNS host, which Cloudflare and Google both name; others are signatures that stopped being refreshed, a key rollover gone wrong, or a DNS host that errors on the CAA record type. Let's Encrypt shows the Extended DNS Error ("DNSSEC: DNSKEY Missing", "Signature Expired"…) when the resolver gives one, and the generic "SERVFAIL … nameservers may be malfunctioning" when it doesn't. Switching CA won't help, and validating resolvers also refuse your records, so mail to the domain is deferred too.

**Fix:**
1. Run our checker (https://mailpassed.com/) on the domain: the DNSSEC and CAA rows show whether the chain is broken and why, and when the DS doesn't match the key your DNS host signs with, the fix shows the exact DS record to publish instead. By hand: `dig CAA example.com @1.1.1.1` returns SERVFAIL and `dig +cd CAA example.com @1.1.1.1` answers? Then DNSSEC is the cause (Cloudflare's test).
2. Moved DNS hosts? At the registrar, delete the old DS record or replace it with your new host's DS. Cloudflare's safe order for a move: remove the DS, wait for its TTL to expire (often 24–48 hours), change the nameservers, then turn DNSSEC back on and add the new DS.
3. "Signature Expired" or "RRSIGs Missing": your DNS host stopped signing. Ask them to re-sign the zone, or delete the DS to go unsigned until they do. An unsigned domain can get certificates again right away.
4. Zone signed with algorithm 5 or 7 (RSASHA1)? RFC 9905 (November 2025) says not to sign with SHA-1 any more, and resolvers that follow it treat such zones as unsigned. Roll to algorithm 13 (ECDSAP256SHA256) or 8.
5. DNSSEC fine but CAA still fails: your DNS host returns an error for CAA queries. Ask them to answer (an empty answer is fine) or move DNS. "CAA record … prevents issuance" means your CAA records don't list this CA: add the identifier your CA documents (for Let's Encrypt, `letsencrypt.org`).
6. Retry the renewal, then reload the services that use the certificate (web server, Postfix, Dovecot): see shorter certificate lifetimes (https://mailpassed.com/changes/mail-server-certificates-shorter-lifetime). Timeline and sources: CAs validate DNSSEC (https://mailpassed.com/changes/cas-validate-dnssec-certificate-renewal).

Full page: https://mailpassed.com/errors/certificate-renewal-dnssec-caa-servfail
