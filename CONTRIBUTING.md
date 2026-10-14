# Contributing / Mitwirken

Thanks for helping improve easyDNS!

## Workflow

1. Fork the repository and create a feature branch.
2. Change `src/easyDNS.ps1` – there is only **one** script; do not add versioned copies (`easyDNS_Vx.y.z.ps1`).
3. Run the checks below.
4. Add an entry to [CHANGELOG.md](CHANGELOG.md) under a **concrete version and date** (`## x.y.z - DD.MM.YYYY`). There is no "Unreleased" section.
5. Update `ScriptVersion`/`LastUpdate` in the script and the version in README.md.
6. Open a pull request with a clear description.

## Checks

```powershell
# Syntax check
$errors = $null
[void][System.Management.Automation.Language.Parser]::ParseFile("$PWD/src/easyDNS.ps1", [ref]$null, [ref]$errors)
$errors

# Static analysis
Install-Module PSScriptAnalyzer -Scope CurrentUser
Invoke-ScriptAnalyzer -Path ./src/easyDNS.ps1 -Severity Error,Warning
```

Functional tests need a Windows machine with the DNS Server role or RSAT DNS tools.

## Coding rules

- Encoding: **UTF-8 with BOM**, needed by Windows PowerShell 5.1.
- Validate every user input before it reaches a DnsServer cmdlet or external program.
- Use `$global:Controls.<Name>`; new XAML controls with `Name="..."` must be added to the control list.
- Use generic lists instead of `+=` in loops.
- Do not block the UI thread with `Start-Sleep`.
- UI and log messages in English; comments may be German.
- Screenshots go into `assets/screenshots/`.
