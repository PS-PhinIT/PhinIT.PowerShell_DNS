# Changelog

All notable changes to easyDNS are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project uses [Semantic Versioning](https://semver.org/).
Every entry belongs to a released version with a date (`DD.MM.YYYY`).

## 0.4.0 - 15.10.2026

### Added
- Audit & Logs → **Query Clients**: Live DNS Monitoring (option "DNS Queries") records incoming DNS queries per client IP – one row per IP with the number of queries, first/last seen and up to five queried names, instead of one entry per query. Optional name filter (host names/FQDNs, wildcards `*`/`?`, a short name also matches `name.<suffix>`), e.g. to find devices that still use an old Exchange server for SMTP (25) or POP3S (995). CSV export (sorted by query count) and Clear. Source is the DNS debug log (local or via the administrative share).
- Configuration keys `QueryClientMaxClients` (5000) and `QueryClientMaxReadMB` (4).

### Changed
- Resource use during long recordings: only the newly written part of the DNS debug log is read per 30 s cycle (byte offset, max. 4 MB, backlog is carried over, truncated logs are detected), the file is only open while reading, memory grows with the number of client IPs and not with the number of queries, the table is only redrawn when it changed and the panel is visible.
- DNS event log grid has a maximum height so its rows are virtualized.

## 0.3.0 - 14.10.2026

Audit release. Full report: [docs/AUDIT.md](docs/AUDIT.md).

### Repository
- Single script at `src/easyDNS.ps1`; the old versioned scripts (`easyDNS_V0.0.14.ps1`, `easyDNS_V0.1.11.ps1`, `easyDNS_V0.2.27.ps1`) and all screenshots removed.
- New folder structure: `src/`, `docs/`, `assets/screenshots/`.
- Added `CHANGELOG.md`, `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `docs/AUDIT.md`; README rewritten.
- `.gitignore` now covers all runtime folders (`Logs/`, `Export/`, `Import/`, `Temp/`).
- Script saved as UTF-8 with BOM so Windows PowerShell 5.1 shows umlauts correctly.
- Authenticode signature removed (it became invalid when the code changed). See README for re-signing.

### Security
- Input validation for server names, zone names, record names, IP addresses, network IDs/prefixes and forwarder IPs before any cmdlet is called.
- `Write-Log` strips CR/LF from messages (log injection).
- Footer website link only opens `https://` URLs.
- Import: 100 MB file size limit, `-LiteralPath`, per-record validation.
- XML import/export no longer uses `Import-Clixml`/`Export-Clixml` (deserialization of arbitrary objects). New built-in functions `Export-SafeXml`/`Import-SafeXml` in the same script: plain XML, DTDs prohibited, no external resolver, text values only. XML exports of older versions remain importable. Event log XML export uses the same format.
- `dnscmd.exe` is only called with a validated zone name; the exit code is checked.
- DNSSEC key length 1024 bit removed (3072 bit added); ECDSA P-384 added.
- Event log search escapes wildcard characters.
- STA apartment check on start.

### Fixed
- Connect button: the server name was lost inside a deferred dispatcher call, so the tool always connected to `localhost`.
- Automatic local connection did not set the connection cache, so the status bar showed "not connected".
- DNS records: delete removed every record with the same name and type instead of only the selected one; MX/SRV data split; `@` and wildcard names allowed.
- `A` record validation accepted incomplete addresses.
- Zone list returned a single object instead of an array when only one zone existed.
- DNSSEC: signing, unsigning, key generation, key export, validation and forced rollover rewritten with the real DnsServer cmdlets (`Invoke-DnsServerZoneSign`, `Invoke-DnsServerZoneUnsign`, `Add-DnsServerSigningKey`, `Invoke-DnsServerSigningKeyRollover`, `Resolve-DnsName -DnssecOk`). Statistics compare "Yes" correctly.
- Export: Forward/Reverse/DNSSEC checkboxes are now honoured.
- Import: zones are created when missing, `Overwrite existing` works, skipped and failed records are counted; A/AAAA/CNAME/PTR/TXT/MX/SRV supported.
- Monitoring: real query counter deltas, DNS Server event log and DNS audit log instead of placeholder data; `Add-MonitoringEvent` parameter conflict fixed.
- Diagnostics: zone refresh/transfer use the correct operation per zone type; server statistics and diagnostics settings show real properties; debug logging enable/disable uses valid parameters; server cache view uses `Show-DnsServerCache`.
- Service start/stop/restart and status now work against remote servers.
- Security events now read the `Microsoft-Windows-DNSServer/Audit` log.
- Dashboard: CPU, disk, uptime and connection status are read from the connected server (remote CIM) instead of the local machine.
- Benchmark/latency: failed queries no longer distort averages; ranking by success rate, then response time.
- Leak test: IPv4 and IPv6, private range detection.
- Traceroute accepts IP addresses and falls back to AAAA.
- Realtime analysis: date range and throughput calculations no longer fail on Windows PowerShell 5.1; top queries are parsed from the real DNS debug log.
- Auto-refresh no longer reloads the DNSSEC panel (it reset the zone selection).
- Closing the window stops all monitoring timers without dialogs.

### Changed
- Ping and port tests use shared helpers (`Invoke-Ping`, `Test-TcpPort`) that behave the same on PowerShell 5.1 and 7 and use short timeouts.
- CPU load via `Win32_Processor.LoadPercentage` instead of localized performance counters.
- Large lists are built with generic lists instead of `+=` arrays.
- Loading indicator no longer blocks the UI thread with `Start-Sleep`.
- All UI and log messages in English; bilingual help texts kept.
- "File (non-AD)" replication option for forward and reverse zones.
- Footer: website link points to this repository, copyright 2025-2026.

### Removed
- Unused functions `Get-SafeString`, `Run-DNSSECValidation`, `Export-AppConfiguration`, `Import-AppConfiguration`.

## 0.2.27 - 26.05.2025

### Added
- Complete WPF rewrite: Windows 11 style interface with panel navigation (8 main sections).
- Real-time monitoring with performance analytics and pattern analysis.
- Advanced diagnostics: DNS benchmarking, leak testing, traceroute analysis.
- DNSSEC management with key generation, validation and rollover.
- Automatic detection of local DNS server roles.
- Response time analysis, throughput monitoring and health report.
- Logging with filtering, export and rotation.
- Inline configuration (no external INI file).
- Import/Export in CSV, XML and JSON.
- Audit and event log integration.

## 0.1.11 - 11.05.2025

### Added
- Performance counters and connection status caching.
- Component-based logging with log rotation.
- Configurable auto-refresh (5 minutes).
- Auto-connect for local DNS servers.
- Retry mechanism for DNS operations.
- Record types A, AAAA, CNAME, MX, PTR, TXT, SRV, NS, SOA.
- Input validation, temporary file handling, caching.

### Changed
- Improved Windows 11 design and responsiveness.

## 0.0.14 - 21.03.2025

### Added
- First WPF version.
- Zone and record management (forward and reverse).
- CSV, XML and JSON export/import.
- Detection of local DNS server installations.
- Logging with file rotation.
- Inline configuration and basic error handling.
