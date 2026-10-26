---
name: bounce-decoder
description: Explain an email bounce, non-delivery report (NDR) or SMTP error and give the fix, using a reference of real Gmail, Outlook/Microsoft 365, Yahoo, Postfix and certificate-authority errors such as 550 5.7.26, 421 4.7.26, 550 5.7.515, 535 5.7.139, 550 5.7.30, 554 5.7.9, S3150 and "SERVFAIL looking up CAA". Use when someone pastes a bounce message or asks what an SMTP status code means.
---

# Bounce decoder

## 1. Find the entry

1. Pull the enhanced status code out of the bounce (three numbers like `5.7.26`) and any distinctive phrase.
2. Search `references/errors.md` for the code, then for the phrase. The **Markers** line of each entry lists phrases that
   identify it even when the code is generic (for example `4.7.1` appears in many unrelated bounces). Prefer a marker match
   over a code match.
3. Read the whole entry: short answer, meaning and fix steps.

## 2. Confirm before you explain

Many bounces are authentication failures. When the entry is about SPF, DKIM or DMARC, look up the sender's records (see
the `email-auth-check` skill) so the answer is about this domain, not a generic one. Identify which domain the receiver
judged: the From address domain, not the mailbox the user logs in with.

## 3. Answer

- One or two sentences on what happened, in plain words. Say whether it is permanent (5.x.x: resending the same way will
  fail again) or temporary (4.x.x: the sender retries, but it becomes a bounce if nothing changes).
- The fix steps from the entry, adapted to what you found in DNS, with exact records where a record is needed.
- The entry's full page link, once, for screenshots and details.

## 4. Codes that aren't in the reference

Say so, then reason from the code's structure (RFC 3463) and the receiver's wording, and label it as general guidance:
`x.1.x` addressing (unknown user or domain), `x.2.x` mailbox (full, disabled), `x.4.x` network or routing (DNS, loops,
timeouts), `x.5.x` protocol, `x.7.x` security or policy (authentication, reputation, blocklists, TLS). Don't invent a
provider-specific meaning you can't source.

## 5. Microsoft 365 sending errors

`535 5.7.139` and `550 5.7.30` from `smtp.office365.com` mean SMTP AUTH with a password was refused. Microsoft turns Basic
authentication for SMTP AUTH off by default at the end of December 2026, so these will spread: switch to the
`m365-smtp-auth` skill for the device-by-device fix.
