# easyDNS v0.4.0 – DNS Server Management Tool

PowerShell-based management tool for the Windows DNS Server role with a Windows 11 style WPF interface.
PowerShell-basiertes Verwaltungstool für die Windows-DNS-Server-Rolle mit WPF-Oberfläche im Windows-11-Stil.

- **Changelog:** [CHANGELOG.md](CHANGELOG.md)
- **Audit report / Audit-Bericht:** [docs/AUDIT.md](docs/AUDIT.md)
- **Security / Sicherheit:** [SECURITY.md](SECURITY.md)
- **Contributing / Mitwirken:** [CONTRIBUTING.md](CONTRIBUTING.md)

---

## Features

| Area | Functions |
|------|-----------|
| Dashboard | OS, user, DNS service, CPU, RAM, disk and uptime of the connected server; zone/record statistics |
| Forward zones | List, create (AD forest / AD domain / legacy / file), delete |
| Reverse zones | IPv4 /8, /16, /24 reverse zones, create, delete |
| DNS records | A, AAAA, CNAME, MX, PTR, TXT, SRV – create, filter, delete exactly the selected record |
| Import / Export | CSV, JSON, XML; forward/reverse/DNSSEC selection; missing zones are created on import |
| DNSSEC | Sign/unsign zones, generate KSK/ZSK (RSA/SHA-256, RSA/SHA-512, ECDSA P-256/P-384), export DNSKEY, validate, force rollover |
| Diagnostics | Ping, nslookup, resolve test, connection test, cache, service control (also remote), server configuration, forwarders, zone refresh/transfer, event logs, debug logging |
| Advanced tools | DNS benchmark, latency test, leak test, traceroute, health report |
| Monitoring | Query/error/security monitoring from counters and event logs, real-time monitor, top queries (DNS debug log), response time and throughput analysis |
| Audit & logs | Event list with filter/search, export; **query clients**: one row per client IP with query count, first/last seen and queried names (name filter, CSV export) |

## Requirements / Voraussetzungen

- Windows 10/11 or Windows Server 2016 – 2025
- Windows PowerShell 5.1 or PowerShell 7.x (Windows), STA mode (default)
- Module `DnsServer` (RSAT DNS tools or DNS Server role)
- Administrator rights (the script uses `#requires -RunAsAdministrator`)
- For remote servers: RPC/WinRM/CIM access and DNS admin permissions on the target

## Repository structure / Ordnerstruktur

```
.
├── src/
│   └── easyDNS.ps1          # application (single file)
├── docs/
│   └── AUDIT.md             # audit report 0.3.0
├── assets/
│   └── screenshots/         # place for current screenshots
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── SECURITY.md
└── README.md
```

At runtime the folders `Logs/`, `Export/`, `Import/` and `Temp/` are created next to the script (`src/`). They are excluded via `.gitignore`.

## Installation & start / Start

```powershell
# Downloaded files are marked as coming from the internet – unblock first
Unblock-File -Path .\src\easyDNS.ps1

# Start from an elevated PowerShell
powershell.exe -ExecutionPolicy Bypass -File .\src\easyDNS.ps1
```

If a local DNS server is detected, easyDNS connects automatically. For a remote server enter the name or IP and click **Connect**.

### Code signing

Version 0.4.0 is **not signed** (the previous signature became invalid when the code changed). With `AllSigned` policies sign the script yourself:

```powershell
$cert = Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert | Select-Object -First 1
Set-AuthenticodeSignature -FilePath .\src\easyDNS.ps1 -Certificate $cert -TimestampServer 'http://timestamp.digicert.com'
```

## Configuration / Konfiguration

Configuration is inline at the top of `src/easyDNS.ps1` (`$global:AppConfig`):

| Key | Default | Meaning |
|-----|---------|---------|
| `AutoRefreshInterval` | `300` | Auto-refresh in seconds |
| `MaxLogEntries` | `10000` | Max. entries in the in-memory log |
| `DefaultTTL` | `3600` | Default TTL for new records |
| `DebugMode` | `$false` | Additional debug logging |
| `QueryClientMaxClients` | `5000` | Max. number of client IPs in the query client list |
| `QueryClientMaxReadMB` | `4` | Max. MB of the DNS debug log read per monitoring cycle (the rest follows in the next cycle) |

## Query clients – which devices still use a server? / Welche Geräte nutzen einen Server noch?

Example: an old Exchange server is to be replaced and you need to know which devices still send mail through it (SMTP 25, POP3S 995).
Beispiel: Ein alter Exchange-Server soll abgelöst werden – welche Geräte nutzen ihn noch für den Mailversand?

1. *Diagnostics → Enable Debug Logging* on the DNS server (queries and received packets are logged to the DNS debug log).
2. *Audit & Logs → Query Clients*: enter the names of the old server in **Name filter**, e.g. `exch01, mail.contoso.com, smtp.contoso.com` (comma-separated, wildcards `*` and `?`; a short name also matches `exch01.<suffix>`). Leave it empty to record all queries.
3. Keep **DNS Queries** checked and click **Start** in *Live DNS Monitoring*.
4. Every client IP appears **once** with the number of queries, first/last seen and the queried names. Sort by any column, **Export CSV** for the list.

Notes:

- DNS cannot see ports. A device that resolves the server name is a candidate; devices configured with the server's IP address do not appear. Clients cache answers for the record TTL, so the count is the number of DNS queries, not of connections.
- Built for long recordings: every 30 s only the newly written part of the debug log is read (max. `QueryClientMaxReadMB`), memory grows with the number of IPs, not with the number of queries (limit `QueryClientMaxClients`, max. 5 names per IP). The table is only redrawn when the panel is visible.
- Stop/Start continues the list if server and name filter are unchanged; otherwise a new recording starts. **Clear** empties the list.
- Disable debug logging after the analysis – it costs DNS performance and the log contains client IP addresses.

## Logging

- File: `src/Logs/easyDNS_YYYYMMDD_<user>.log`
- Levels: `DEBUG`, `INFO`, `SUCCESS`, `WARN`, `ERROR`
- Line breaks in messages are removed (protection against log injection)

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `Get-DnsServerZone` not found | Install RSAT DNS tools / `DnsServer` module |
| Connection failed | Check DNS service, firewall (RPC, TCP 135 + dynamic ports) and permissions |
| Script will not start | `Unblock-File`, check execution policy, start elevated |
| Umlauts garbled | Make sure the file is saved as UTF-8 **with** BOM |
| Top queries / query clients empty | Enable DNS debug logging (Diagnostics → Debug logging) |

## Known limitations / Bekannte Einschränkungen

- XML import/export uses a built-in safe XML format (no `Import-Clixml`, no DTDs). XML exports of older versions (Clixml) can still be imported.
- Tested with the PowerShell parser, PSScriptAnalyzer and XAML checks; functional tests require a Windows DNS server.
- No license file is included in the repository.

## Author

- **Author:** Andreas Hepp – PHscripts.de
- **Repository:** <https://github.com/PS-PhinIT/PhinIT.PowerShell_DNS>
- **Version:** 0.4.0 (15.10.2026)
