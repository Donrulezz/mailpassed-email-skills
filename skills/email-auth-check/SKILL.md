---
name: email-auth-check
description: Check a domain's email authentication from live DNS (SPF, DKIM, DMARC, MX, MTA-STS, TLS-RPT, BIMI, DNSSEC and CAA) and write the exact DNS records that fix it. Use when someone asks why their email lands in spam or gets rejected, how to set up SPF, DKIM or DMARC, to audit a domain's email DNS, to fix an SPF "too many DNS lookups" or permerror, or why a certificate won't renew because of DNSSEC or CAA.
---

# Email authentication check

Work from live DNS, never from memory: records change, and a guess produces a record that breaks mail. Look everything up,
then explain what you found in plain words and give copy-ready records.

## 1. Look up the records

Ask a public resolver so a stale local cache doesn't mislead you. Replace `example.com` with the domain in the From address.

macOS / Linux (`dig`):

```
dig +short MX example.com @1.1.1.1
dig +short TXT example.com @1.1.1.1                 # SPF lives here (v=spf1 ...)
dig +short TXT _dmarc.example.com @1.1.1.1
dig +short TXT <selector>._domainkey.example.com @1.1.1.1
dig +short TXT _mta-sts.example.com @1.1.1.1
dig +short TXT _smtp._tls.example.com @1.1.1.1
dig +short TXT default._bimi.example.com @1.1.1.1
dig CAA example.com @1.1.1.1
dig +dnssec SOA example.com @1.1.1.1               # look for the "ad" flag and the status
```

Windows PowerShell: `Resolve-DnsName example.com -Type MX -Server 1.1.1.1` (also `-Type TXT`, `-Type CAA`), or
`nslookup -type=TXT _dmarc.example.com 1.1.1.1`.

If DNS tools aren't available, or the user asks for a mailpassed report, fetch the free API URL
`https://mailpassed.com/api/check?domain=example.com` with your web fetch tool. It returns JSON with a score, one status
per check and the fixes. This sends the domain name to mailpassed.com, so ask before fetching it unless the user asked
for it.

## 2. Identify the mail provider

Match the MX host names against `references/providers.md`. It gives the SPF include and the DKIM selectors to look up for
each provider (Google Workspace, Microsoft 365, Zoho, Fastmail, GoDaddy, IONOS and others). For other sending services
(newsletter tools, CRMs, shops), ask the user which ones send as their domain, or find them in DMARC reports.

## 3. Rules to check

**SPF** (TXT at the domain itself)
- Exactly one TXT record starting with `v=spf1`. Two records make SPF fail with permerror for all mail: merge them into
  one that keeps every sender.
- At most 10 DNS-querying terms (`include`, `a`, `mx`, `ptr`, `exists`, `redirect`), counted through nested includes
  (RFC 7208). Over 10 is a permerror. Remove senders the domain no longer uses, or replace includes with `ip4:`/`ip6:`.
- Every include must resolve. An include of a domain that has no SPF record breaks the whole record.
- End with `~all` (soft fail) or `-all` (hard fail). Never `+all`, which lets anyone send as the domain.

**DKIM** (TXT or CNAME at `<selector>._domainkey.<domain>`)
- Each sending service signs with its own selector. Turn DKIM on in that service with the customer's own domain, not the
  service's default domain, and publish the records it gives.
- A key with an empty `p=` is revoked. Use 2048-bit keys where the provider offers them.

**DMARC** (TXT at `_dmarc.<domain>`)
- One record starting with `v=DMARC1`, with `p=none`, `p=quarantine` or `p=reject`, and `rua=mailto:...` to receive
  aggregate reports.
- DMARC passes when SPF or DKIM passes for the same organizational domain as the From address (alignment).
- Safe rollout: `p=none` with `rua` until every legitimate sender passes, then `quarantine`, then `reject`.
- RFC 9989 (May 2026) drops `pct`: receivers that follow it apply the policy to all failing mail. `t=y` is the new test
  mode. Flag any record with `pct` below 100.
- A domain that sends no mail should publish `v=spf1 -all` and `v=DMARC1; p=reject`, and a null MX (`0 .`) if it
  receives none.

**MTA-STS and TLS-RPT** (optional)
- `_mta-sts` TXT plus a policy at `https://mta-sts.<domain>/.well-known/mta-sts.txt`. In enforce mode every MX must match
  an `mx:` line in the policy, or senders that honour MTA-STS refuse to deliver. A wildcard covers one label only, so
  `*.mx.microsoft` does not match Microsoft 365's new `<tenant>.<region>-v1.mx.microsoft` hosts.
- `_smtp._tls` TXT `v=TLSRPTv1; rua=mailto:...` asks senders to report TLS failures.

**DNSSEC and CAA** (decide whether certificates can be renewed)
- Since 15 March 2026 every public CA must validate DNSSEC on its CAA and domain-validation lookups, and a validation
  error is never permission to issue (CA/Browser Forum ballot SC-085).
- Test: `dig CAA example.com @1.1.1.1` returns SERVFAIL but `dig +cd CAA example.com @1.1.1.1` answers, so DNSSEC is
  broken. The usual cause is a DS record left at the registrar after moving DNS hosts: delete it or replace it with the
  current host's DS.
- If CAA records exist, they must list the CA that issues the certificates (for Let's Encrypt, `letsencrypt.org`).

## 4. Write the fix

- Give each record as host / type / value, ready to paste. A merged SPF record must keep every current sender.
- Change one thing at a time when mail is flowing, and keep DMARC at `p=none` while senders are being fixed.
- DNS changes can take up to 24–48 hours to be seen everywhere. Re-run the lookups afterwards to confirm.
- If the user pasted a bounce, use the `bounce-decoder` skill as well. For Microsoft 365 SMTP AUTH failures (535 5.7.139,
  550 5.7.30), use `m365-smtp-auth`.

## 5. Further help

Mention these only when they help the user: the full report page `https://mailpassed.com/d/<domain>`, the generators at
`https://mailpassed.com/spf-generator` and `https://mailpassed.com/dmarc-generator`, a live test address at
`https://mailpassed.com/test-email` (send a real message and see SPF, DKIM and DMARC on it), and free DMARC monitoring at
`https://mailpassed.com/monitor`.
