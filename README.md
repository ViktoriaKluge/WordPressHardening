# WordPress Server Hardening

Dieses Protokoll ist im Rahmen der Lehrveranstaltung System Hardening entstanden und gibt einen Einblick in meine Arbeits- und Vorgangsweise. Es handelt sich um eine isolierte Laborumgebung mit VirtualBox und Host-only-Netzwerk, also kein Produktivsystem. Da die Lehrveranstaltung auf Englisch gehalten und durchgeführt wurde, ist 
auch das Protokoll auf Englisch.

## Protocol Contents
- Abstract
- Procedure and Set-Up
- Analysis Part
- Conclusions
- References
- Appendix

## Setup

- Debian 12 (Netinstall, nur SSH-Server und Standard-Systemwerkzeuge)
- Nginx, PHP-FPM 8.2, MariaDB, WordPress
- Zweite VM (Kali) als Testmaschine für Scans und einfache Angriffssimulationen
- Tools: Lynis, nmap, WPScan, sslscan, curl, ApacheBench

## Vorgehen

1. **Basiswerte ermitteln:** Der Ausgangszustand wird dokumentiert bevor etwas geändert wird, damit die Wirkung der Maßnahmen auch messbar ist.
2. **Schicht für Schicht härten:** Betriebssystem (SSH, Firewall, Updates) → WordPress → Webserver → PHP-FPM / MariaDB → AppArmor, auditd, Backups.
3. **Maßnahmen gleich überprüfen:** Jede Änderung wird von der Kali-VM aus getestet.
4. **Begründung:** Für jede Maßnahme wird erklärt, warum sie durchgeführt oder bewusst nicht durchgeführt wird.

## Ausgewählte Ergebnisse

| Bereich | Vorher | Nachher |
|---|---|---|
| Nginx-Version | sichtbar | versteckt |
| Security-Header | keine | HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy |
| TLS | nicht aktiv | nur TLS 1.2 / 1.3, HTTP/2 |
| XML-RPC, wp-cron.php, readme.html, wp-config.php | erreichbar | 403 |
| Login-Brute-Force | unbegrenzt | Rate Limit 5 Anfragen/min, 16 von 20 Testanfragen blockiert |
| `/wp-admin/` | uneingeschränkt | auf das lokale Netz beschränkt |
| Dateirechte | wp-config.php gehörte root | 755 / 644 / 640, keine world-writable Dateien |
| Audit-Logging | keines | auditd-Regeln für wp-config.php, sshd_config, Nginx-Konfiguration |
| Backups | keine | DB-Dump und wp-content-Archiv mit Prüfsummen, Restore getestet |
| Lynis Hardening Index | 61 (frische Installation), 69 (Baseline) | 71 |

## Was ich gelernt habe

- **Testen deckt Fehler auf.** Erst der Test hat gezeigt, dass meine erste Regel zum Blockieren von PHP-Ausführung im Uploads-Ordner nicht funktioniert hat. Durch die Verschiebung der Prüfung in den `.php`-Location-Block wurde die Testdatei nicht mehr ausgeführt. (siehe Protokoll, Webserver Hardening)
- **Ein Tool-Score zeigt nicht das ganze Bild.** Der Lynis-Index hat sich am Ende kaum verändert, weil die meisten WordPress-spezifischen Maßnahmen, also Header, Rate Limiting, gesperrte Pfade usw. von Lynis gar nicht erfasst werden. Trotzdem hat sich die Sicherheit klar verbessert. Es sollte also immer beachtet werden, wie aussagekräftig eine Messung überhaupt ist.
- **Sicherheit und Funktionalität passen nicht immer zusammen.** Die WordPress-Version in den CSS/JS-Query-Parametern habe ich bewusst sichtbar gelassen, weil das Verstecken die Seite beschädigen könnte und die Version aktuell war. Solche bewusst akzeptierten Risiken stehen im Fazit.

## Bekannte Einschränkungen

- Selbstsigniertes Zertifikat (keine echte Domain, daher kein Let's Encrypt)
- Keine AppArmor-Profile für Nginx und MariaDB (keine vorhanden)
- Kein Malware-Scanner, kein Remote-Logging
- Die Einschränkung von `/wp-admin/` konnte nicht von außerhalb des lokalen Netzes getestet werden
