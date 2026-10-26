# Mailbox providers: how to recognise them and what to publish

Match a fragment against the domain's MX host names. The SPF include and DKIM selectors are what https://mailpassed.com's checker uses for that provider.

| MX contains | Provider | SPF include | DKIM selectors to look up |
|---|---|---|---|
| `google.com` | Google Workspace | `include:_spf.google.com` | `google` |
| `googlemail.com` | Google Workspace | `include:_spf.google.com` | `google` |
| `outlook.com` | Microsoft 365 | `include:spf.protection.outlook.com` | `selector1`, `selector2` |
| `zoho.in` | Zoho Mail (IN) | `include:zohomail.in` | `zmail` |
| `zoho.eu` | Zoho Mail (EU) | `include:zohomail.eu` | `zmail` |
| `zoho.com` | Zoho Mail | `include:zohomail.com` | `zmail` |
| `protonmail.ch` | Proton Mail | `include:_spf.protonmail.ch` | `protonmail`, `protonmail2`, `protonmail3` |
| `titan.email` | Titan | `include:spf.titan.email` | `titan1` |
| `hostinger.com` | Hostinger Mail | `include:_spf.mail.hostinger.com` | `hostingermail-a`, `hostingermail-b`, `hostingermail-c` |
| `secureserver.net` | GoDaddy | `include:secureserver.net` | `secureserver1`, `secureserver2` |
| `privateemail.com` | Namecheap Private Email | `include:spf.privateemail.com` | `privateemail`, `default` |
| `ionos.com` | IONOS (US) | `include:_spf-us.ionos.com` | `s1-ionos`, `s2-ionos` |
| `ionos.` | IONOS (EU) | `include:_spf-eu.ionos.com` | `s1-ionos`, `s2-ionos` |
| `messagingengine.com` | Fastmail | `include:spf.messagingengine.com` | `fm1`, `fm2`, `fm3` |
| `yandex` | Yandex 360 | `include:_spf.yandex.net` | `mail` |
| `mx.cloudflare.net` | Cloudflare Email Routing | `include:_spf.mx.cloudflare.net` | `cf2024-1` |
