# mailpassed: email authentication and SMTP fixes for Claude

Free skills that let Claude diagnose and fix email problems from live DNS instead of guesswork: SPF, DKIM and DMARC
records, bounce messages, certificate renewals blocked by DNSSEC or CAA, and the Microsoft 365 SMTP AUTH change that
turns Basic authentication off by default at the end of December 2026.

The knowledge comes from [mailpassed.com](https://mailpassed.com), where every page is checked against primary sources
(Microsoft, Google, IETF RFCs, the CA/Browser Forum, Let's Encrypt and device vendors).

## Skills

| Skill | Ask Claude things like |
|---|---|
| `email-auth-check` | "Why does mail from example.com go to spam?", "Fix my SPF, it says too many DNS lookups", "Set up DMARC for my domain" |
| `bounce-decoder` | "What does this bounce mean?" (paste it), "Gmail says 550 5.7.26", "Outlook returns 550 5.7.515" |
| `m365-smtp-auth` | "Our scanner can't send through Office 365 any more", "How do I move our HP printers off SMTP AUTH?", "535 5.7.139" |
| `upcoming-email-changes` | "What email changes should I prepare for?", "Does the DMARC pct change affect us?", "Will my certificate renewals break?" |

Claude picks the right skill from your question. You can also call one directly, for example
`/mailpassed:email-auth-check example.com`.

## What it runs, sends and fetches

- **DNS lookups on your computer.** The skills ask Claude to run `dig`, `nslookup` or PowerShell `Resolve-DnsName`
  against a public resolver (1.1.1.1). These are ordinary DNS queries for the domain you ask about.
- **Optional mailpassed.com API.** Only when you ask for a mailpassed report, or when no DNS tool is available and you
  agree, Claude calls `https://mailpassed.com/api/check?domain=<domain>`. That sends the domain name, nothing else. See
  the [privacy policy](https://mailpassed.com/privacy).
- **Optional PowerShell script.** `skills/m365-smtp-auth/scripts/Find-SmtpAuthSenders.ps1` is a read-only inventory of
  Microsoft 365 SMTP AUTH senders. You run it yourself and sign in with your own admin account. It reads Exchange Online
  settings and, if you pass the switches, Microsoft Graph usage reports and Entra ID sign-in logs, then writes a CSV and an
  HTML report to your disk. It sends nothing anywhere else and changes nothing in your tenant. Source and tests:
  [Donrulezz/Find-SmtpAuthSenders](https://github.com/Donrulezz/Find-SmtpAuthSenders).
- **Links.** Answers may include links to the matching page on mailpassed.com for details and screenshots.
- No credentials are needed or stored, and the skills contain no hidden or remote instructions: everything Claude reads
  is in this repository.

## Install

In Claude Code:

```
/plugin marketplace add Donrulezz/mailpassed-email-skills
/plugin install mailpassed@mailpassed
```

## Keeping it current

The reference files in `skills/*/references/` are generated from mailpassed.com's data, so they change when a provider or
Microsoft changes something. Check the date of the last commit, and the primary source linked in each entry, for anything
time-sensitive.

## Paid help (optional)

The skills are complete on their own. If you'd rather not do it yourself, mailpassed.com links to a $9 per-domain fix plan,
printable fix kits and done-for-you setup. None of it is required.

## License

MIT. Issues and corrections: [open an issue](https://github.com/Donrulezz/mailpassed-email-skills/issues) or email
hello@mailpassed.com.
