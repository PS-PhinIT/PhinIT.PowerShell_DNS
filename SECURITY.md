# Security Policy / Sicherheitsrichtlinie

## Supported versions

| Version | Supported |
|---------|-----------|
| 0.3.x   | Yes |
| < 0.3   | No (removed from the repository) |

## Reporting a vulnerability / Schwachstelle melden

Please do **not** open a public issue. Use GitHub's private vulnerability reporting
(*Security* → *Report a vulnerability*) in this repository and include:

- affected version and file
- steps to reproduce
- impact

You will receive a response as soon as possible; fixes are documented in [CHANGELOG.md](CHANGELOG.md).

## Security notes for operators / Hinweise für Betreiber

- easyDNS requires administrator rights and changes DNS server configuration. Run it only on trusted admin workstations or the DNS server itself.
- **XML import:** easyDNS does not use `Import-Clixml`. XML files are read by a built-in parser with DTD processing prohibited and no external resolver; only text values are read, no .NET objects are created. Older Clixml exports are read the same way.
- Import files are limited to 100 MB and every record is validated before it is written.
- The script is not signed as of 0.3.0. Verify the source and sign it yourself if your policy requires `AllSigned` (see README).
- Log files (`src/Logs/`) can contain server and zone names. Protect the folder accordingly.
- DNSSEC: 1024-bit RSA keys are no longer offered; use 2048 bit or more, or ECDSA.
