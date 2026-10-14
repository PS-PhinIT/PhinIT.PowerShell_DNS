# Audit easyDNS – Version 0.2.27 → 0.3.0

**Datum:** 14.10.2026
**Umfang:** `easyDNS_V0.2.27.ps1` (jetzt `src/easyDNS.ps1`) – Sicherheit, Performance, Funktionen, Verweise, GUI, Befehle, Repository-Struktur.
**Methode:** Code-Review aller Funktionen und Event-Handler, PowerShell-Parser, PSScriptAnalyzer 1.25, XAML-Wohlgeformtheit, Abgleich XAML-Controls ↔ Control-Liste ↔ Verwendung im Code.

Status: ✅ behoben · ⚠️ dokumentiertes Restrisiko

## 1. Sicherheit

| # | Befund | Schwere | Status |
|---|--------|---------|--------|
| S1 | Server-, Zonen-, Record-Namen und IPs wurden ungeprüft an DnsServer-Cmdlets und `dnscmd.exe` übergeben | Hoch | ✅ `Test-DnsServerName`, `Test-DnsName`, `Test-RecordName`, `TryParse` für IPs; `dnscmd` nur mit validiertem Zonennamen + Exit-Code-Prüfung |
| S2 | Log-Injection: Zeilenumbrüche in Meldungen erzeugten gefälschte Logzeilen | Mittel | ✅ `Write-Log` entfernt CR/LF |
| S3 | Import ohne Größenlimit, ohne Validierung je Datensatz | Mittel | ✅ 100-MB-Limit, `-LiteralPath`, Validierung, Zähler für übersprungene/fehlerhafte Datensätze |
| S4 | XML-Import über `Import-Clixml` deserialisiert beliebige Objekte | Mittel | ✅ Eigener XML-Parser im selben Skript (`Import-SafeXml`/`Export-SafeXml`): DTD verboten, kein Resolver, nur Textwerte; alte Clixml-Exporte bleiben lesbar |
| S5 | DNSSEC bot 1024-Bit-RSA an | Mittel | ✅ entfernt, 3072 Bit und ECDSA P-384 ergänzt |
| S6 | Ereignis-Suche: Wildcards aus Benutzereingabe nicht maskiert | Niedrig | ✅ `[WildcardPattern]::Escape` |
| S7 | Website-Link öffnete beliebige URL | Niedrig | ✅ nur `https://` |
| S8 | Authenticode-Signatur nach Codeänderungen ungültig | Info | ✅ entfernt; Anleitung zum Neusignieren in README |
| S9 | Keine STA-Prüfung (WPF stürzt in MTA ab) | Niedrig | ✅ Prüfung beim Start |

## 2. Performance

| # | Befund | Status |
|---|--------|--------|
| P1 | Arrays mit `+=` in Schleifen (Zonen, Records, Export, Events) – O(n²) | ✅ generische Listen |
| P2 | `Show-LoadingStatus` blockierte den UI-Thread mit `Start-Sleep` | ✅ entfernt |
| P3 | `Test-NetConnection` (langsam, lange Timeouts) für Port-Tests | ✅ `Test-TcpPort` mit Timeout |
| P4 | `Get-Counter` – langsam und Zählernamen lokalisiert (deutsches Windows schlägt fehl) | ✅ `Win32_Processor.LoadPercentage` / `Get-DnsServerStatistics` |
| P5 | `Get-WinEvent` ohne Limit | ✅ `Get-DnsWinEvent` mit `MaxEvents` und Filter-Hashtable |
| P6 | Auto-Refresh lud DNSSEC-Panel neu (viele RPC-Aufrufe, Auswahl ging verloren) | ✅ DNSSEC aus Auto-Refresh entfernt |
| P7 | Traceroute mit DNS-Rückwärtsauflösung je Hop | ✅ `tracert -d` |

## 3. Funktionen

| # | Befund | Status |
|---|--------|--------|
| F1 | **Verbinden** verband immer mit `localhost` (Variable im verzögerten Dispatcher-Aufruf verloren) | ✅ synchron nach Render |
| F2 | Auto-Connect setzte Verbindungs-Cache nicht | ✅ `Test-DNSServerConnection -ForceCheck` |
| F3 | Record löschen entfernte alle Records gleichen Namens/Typs | ✅ nur passende RecordData |
| F4 | MX/SRV-Daten falsch zerlegt; `@`/Wildcard-Namen abgelehnt | ✅ |
| F5 | A-Record-Validierung akzeptierte unvollständige IPs | ✅ |
| F6 | Bei genau einer Zone kein Array (`.Count` falsch) | ✅ |
| F7 | DNSSEC: Signieren/Schlüssel/Rollover/Export/Validierung waren Platzhalter oder falsche Cmdlets | ✅ echte DnsServer-Cmdlets |
| F8 | DNSSEC-Statistik verglich `IsSigned` mit `$true` statt "Yes" | ✅ |
| F9 | Export ignorierte Forward/Reverse/DNSSEC-Checkboxen | ✅ |
| F10 | Import legte fehlende Zonen nicht an, „Überschreiben" ohne Wirkung | ✅ |
| F11 | Monitoring zeigte Zufalls-/Platzhalterdaten | ✅ Zähler-Deltas, DNS-Server- und Audit-Log |
| F12 | `Add-MonitoringEvent -Event` kollidierte mit automatischer Variable | ✅ `-EventType` |
| F13 | Zonen-Refresh/-Transfer ohne Unterscheidung des Zonentyps | ✅ `Invoke-DnsZoneSync` |
| F14 | Dienststeuerung und -status nur lokal | ✅ remote über CIM |
| F15 | Sicherheitsereignisse lasen das Windows-Security-Log | ✅ `Microsoft-Windows-DNSServer/Audit` |
| F16 | Debug-Logging mit ungültigen Parametern | ✅ |
| F17 | Benchmark/Latenz: fehlgeschlagene Abfragen im Durchschnitt | ✅ |
| F18 | Echtzeit-Analyse: `Measure-Object` auf `DateTime` scheitert in PS 5.1 | ✅ |
| F19 | Top-Abfragen waren simuliert | ✅ Auswertung des DNS-Debug-Logs |
| F20 | Ping-Ergebnisse unterscheiden sich zwischen PS 5.1 und 7 | ✅ `Invoke-Ping` |
| F21 | Tote Funktionen (`Get-SafeString`, `Run-DNSSECValidation`, `Export-/Import-AppConfiguration`) | ✅ entfernt |
| F22 | Beim Schließen liefen Monitoring-Timer weiter bzw. zeigten Dialoge | ✅ |

## 4. Verweise

| # | Befund | Status |
|---|--------|--------|
| V1 | Website/GitHub-Links zeigten auf `PS-easyIT` statt auf dieses Repository | ✅ |
| V2 | README verwies auf `easyDNS_V0.2.27.ps1` | ✅ `src/easyDNS.ps1` |
| V3 | Changelog nur im README, kein CHANGELOG.md | ✅ |
| V4 | Fehlende SECURITY.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md | ✅ |
| V5 | Keine LICENSE-Datei | ⚠️ muss vom Inhaber festgelegt werden |

## 5. GUI

| # | Befund | Status |
|---|--------|--------|
| G1 | Gemischte deutsch/englische Texte | ✅ UI/Logs Englisch; zweisprachige Hilfetexte bleiben |
| G2 | Dashboard zeigte CPU/RAM/Disk des lokalen PCs bei Remote-Verbindung | ✅ Remote-CIM |
| G3 | Keine Replikationsoption für Server ohne AD | ✅ „File (non-AD)" |
| G4 | Footer-Website nicht klickbar, Copyright veraltet | ✅ |
| G5 | XAML-Controls ↔ Control-Liste ↔ Code | ✅ geprüft, keine fehlenden Namen |

## 6. Repository

| # | Befund | Status |
|---|--------|--------|
| R1 | Drei Skriptversionen parallel im Root | ✅ nur `src/easyDNS.ps1` |
| R2 | Veraltete Screenshots | ✅ gelöscht, Ordner `assets/screenshots/` |
| R3 | Encoding UTF-8 ohne BOM (Umlaute in PS 5.1 kaputt) | ✅ UTF-8 mit BOM |
| R4 | `.gitignore` deckte `Import/` und `Temp/` nicht ab | ✅ |

## Prüfergebnisse 0.3.0

- PowerShell-Parser: 0 Fehler
- PSScriptAnalyzer (Error/Warning): keine Fehler; verbleibende Warnungen sind Stilregeln (globale Variablen, Verben, Singular-Nomen, ShouldProcess) bzw. False Positives bei Variablen in Scriptblöcken.
- XAML: wohlgeformt; alle referenzierten Controls vorhanden.
- **Nicht durchgeführt:** Funktionstest gegen einen echten Windows-DNS-Server (Testumgebung war Linux). Vor dem produktiven Einsatz in einer Testumgebung prüfen.
