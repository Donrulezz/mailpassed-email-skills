---
name: upcoming-email-changes
description: Dated changes in 2026–2027 that break email or certificates, with status, who is affected, how to check and how to fix, from primary sources. Covers Microsoft 365 SMTP AUTH Basic auth off by default (end of December 2026), DMARCbis RFC 9989 dropping pct, Gmail's POP fetch and Gmailify ending, shorter certificate lifetimes (100 days from March 2027), root CA distrust, Microsoft 365's new mx.microsoft MX hosts, public certificates losing clientAuth, and CAs validating DNSSEC. Use when someone asks what is changing in email, about a deadline, or whether a change affects their domain.
---

# Upcoming email and certificate changes

`references/changes.md` lists each change with its status (confirmed, expected or reported), dates, who is affected, what
breaks, how to check, how to fix, and the primary sources (Microsoft, Google, IETF, CA/Browser Forum, Let's Encrypt and
others).

## How to answer

1. Find the change in `references/changes.md` by product, deadline or error text.
2. Give the date and the status exactly as the file states them. Don't turn "expected" into "confirmed" and don't move a
   date. If something may have changed since the file was generated, say so and point to the change's primary source.
3. Say who is affected and who isn't, so people who are safe don't make changes they don't need.
4. If the user names a domain, run the "How to check" steps against live DNS (see the `email-auth-check` skill) and tell
   them whether the change applies to them.
5. Give the fix steps that fit their setup, and the full page link once.

When asked "what should I fix first", sort by date and by how much breaks: a confirmed change that stops mail or
certificate renewal comes before an expected one that only changes reporting.
