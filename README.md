# MTA-STS policy host

The one static file other mail servers read before delivering to `fortunapace.com` and `fortunapace.co.uk`:
`.well-known/mta-sts.txt` (RFC 8461). Both domains' mail is Microsoft 365, whose inbound servers are
`*.mail.protection.outlook.com`, so one policy serves both. Hosted on Vercel as the project `mta-sts`, reached as
`mta-sts.fortunapace.com` and `mta-sts.fortunapace.co.uk`.

## The records (123-Reg DNS, both domains)

| Record | Name | Value |
|---|---|---|
| CNAME | `mta-sts` | the target Vercel shows for the domain (`cname.vercel-dns.com.`) |
| TXT | `_mta-sts` | `v=STSv1; id=<the id in the table below>` |
| TXT | `_smtp._tls` | `v=TLSRPTv1; rua=mailto:reports@fortunapace.com` (already published) |

The `id` must change whenever the policy file changes; senders re-fetch the file only when they see a new id.

| Policy version | mode | max_age | id |
|---|---|---|---|
| 22 Sep 2026 | testing | 86400 (1 day) | `20260922a` |

## Going to enforce

1. Leave `mode: testing` until the TLS reports (they arrive daily at reports@fortunapace.com; the cockpit's *TLS
   reports* check reads them) show no failures for a couple of weeks.
2. Change the file to `mode: enforce` and `max_age: 604800` (a week) or longer, deploy, then change both `_mta-sts`
   TXT ids. Never enforce while failures are being reported: that mail would be refused instead of delivered.
3. To withdraw: `mode: none` in the file and a new id; senders drop the policy as they re-fetch.

## Rules

* The file must be served over HTTPS with a certificate valid for the exact host name, as `text/plain`, with no
  redirects. Vercel issues the certificate from Let's Encrypt, which both domains' CAA records allow.
* Nothing may sit in front of it that challenges non-browser clients (a bot-protection page is a failed fetch).
