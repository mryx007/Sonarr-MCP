# Sonarr-MCP

Lokaler MCP-Server für **Sonarr 4 unter Windows**. Ein MCP-Client wie Codex kann
damit Qualitätsprofile, eigene Formate und weitere Sonarr-Einstellungen lesen
und auf Auftrag bearbeiten.

- Python-Standardbibliothek, keine zusätzlichen Pakete.
- Lokale STDIO-Verbindung, kein zusätzlicher Webserver.
- Vorschau vor Änderungen und verschlüsselte Sicherung vor Schreibzugriffen.
- Profile und Formate anlegen, kopieren und bearbeiten.
- Allgemeiner Zugriff auf die Sonarr-JSON-API.

Unabhängiges Community-Projekt; kein offizielles Produkt von Sonarr oder OpenAI.

## Voraussetzungen

Windows, Python 3.10 oder neuer, Sonarr 4 mit erreichbarer API und ein
MCP-Client mit Unterstützung für lokale STDIO-Server.
Schreibzugriffe benötigen Windows DPAPI für die verschlüsselten Sicherungen.
Sonarr selbst darf auf einem anderen Rechner laufen.

## Einrichtung

~~~powershell
git clone https://github.com/mryx007/Sonarr-MCP.git
cd Sonarr-MCP
~~~

Bei lokaler Sonarr-Installation unter C:\ProgramData\Sonarr liest der Server
Port, URL-Basis und API-Schlüssel automatisch aus der config.xml.
Der Schlüssel wird nicht ins Projekt kopiert.

Bei anderer Installation settings.example.json nach settings.json kopieren
und config_path anpassen. Ein optionales Feld url überschreibt die
automatisch ermittelte Basisadresse.

Alternativ werden folgende Umgebungsvariablen unterstützt:

| Variable | Bedeutung |
| --- | --- |
| SONARR_CONFIG_PATH | Pfad zu Sonarrs config.xml |
| SONARR_URL | Basisadresse einschließlich optionaler URL-Basis |
| SONARR_API_KEY | Sonarr-API-Schlüssel |

Zugangsdaten nur lokal in der Umgebung des MCP-Clients bereitstellen.

### Codex

Im Projektordner ausführen, wenn Python und Codex installiert sind:

~~~powershell
$serverPath = Join-Path (Get-Location) "server.py"
$pythonPath = (Get-Command python -ErrorAction Stop).Source
codex mcp add sonarr -- $pythonPath -u $serverPath
~~~

Anschließend Codex neu starten. Andere MCP-Clients können den absoluten
Python-Pfad mit den Argumenten -u und dem absoluten Pfad zu server.py starten.

### Genehmigungen

Ohne weitere Einstellungen gelten die Freigaberegeln deines MCP-Clients.

**Optional:** Wer nach bewusster Zustimmung alle Sonarr-Schreibaktionen
dauerhaft freigeben möchte, kann in seiner eigenen Codex-Konfiguration
(~/.codex/config.toml) ergänzen:

~~~toml
[mcp_servers.sonarr.tools.sonarr_apply_change]
approval_mode = "approve"
~~~

Das umfasst **auch Löschungen und Befehle**, nicht nur Profiländerungen.
Der Server setzt diese Berechtigung nicht selbst. Vorschauen und Sicherungen
bleiben bestehen. Danach Codex neu starten. Verwaltete Clients können weitere
Vorgaben haben; andere Clients verwalten Freigaben auf ihre eigene Weise.

[Offizielle Codex-Anleitung](https://learn.chatgpt.com/docs/extend/mcp)

## Werkzeuge

| Werkzeug | Zweck |
| --- | --- |
| sonarr_overview | Status, Warnungen, Profile, Formate, Qualitätsdefinitionen |
| sonarr_api_schema | Mitgelieferte Sonarr-4-API-Beschreibung durchsuchen |
| sonarr_read | Lesender Aufruf eines /api/v3/...-Endpunkts |
| sonarr_prepare_profile | Profil/Format anlegen, kopieren oder teilweise ändern |
| sonarr_prepare_change | Vorschau für POST, PUT, PATCH oder DELETE |
| sonarr_apply_change | Genau einen vorbereiteten Plan übernehmen |
| sonarr_list_backups | Verschlüsselte Sicherungen auflisten |
| sonarr_prepare_restore | Einzelnes geändertes Objekt zurücksetzen vorbereiten |

Bei allgemeinen PUT-Aufrufen das vollständige aktuelle Objekt verwenden.
Listen werden bei Profiländerungen vollständig ersetzt.

**Auch GET-Aufrufe können Sonarr-Aktionen auslösen:** /api/v3/release kann
beispielsweise Indexer abfragen. Für reine Namensprüfungen /api/v3/parse
verwenden. Der Client muss die Bedeutung des Endpunkts beachten.

Die laufende Sonarr-Version entscheidet über gültige Felder und Methoden.
Die mitgelieferte API-Beschreibung ist eine Referenz für Sonarr 4.
Multipart-Dateiuploads, etwa für Backup-ZIPs, sind nicht implementiert.
Antworten sind auf 16 MiB begrenzt; große Listen paginieren.

## Vorschauen und Sicherungen

Pläne gelten 30 Minuten, nur im laufenden Serverprozess und können nur einmal
angewendet werden. Nach einem Neustart neu vorbereiten.

Vor dem Schreiben wird der vorherige API-Zustand mit Windows DPAPI
verschlüsselt unter backups/ gespeichert. Die Dateien sind an das aktuelle
Windows-Benutzerkonto gebunden und können Zugangsdaten enthalten.

Ändert sich das gesicherte Objekt seit der Vorschau, wird der Plan abgelehnt.
Schlägt die Sicherung fehl, wird nicht geschrieben. Bei Aktionsendpunkten
ohne lesbaren Objektzustand wird nur der Systemstatus gesichert, erkennbar
an resource_snapshot_available=false. Dort ist kein zuverlässiger Vergleich
des eigentlichen Zielobjekts möglich.

**Dies sind keine vollständigen Datenbank- oder Mediendatei-Backups.**
Downloads, gelöschte Medien und externe Aktionen lassen sich damit nicht
rückgängig machen. Automatische Wiederherstellung ist nur für PUT-/PATCH-
Änderungen einzelner vorhandener Objekte vorgesehen. Für vollständige
Wiederherstellung Sonarrs eingebaute Backups verwenden.

Nach Verbindungsfehlern beim Schreiben zuerst den aktuellen Zustand lesen:
Die Aktion könnte bereits ausgeführt worden sein. Niemals blind wiederholen.

## Datenschutz und Grenzen

- Keine Telemetrie und keine eigenen OpenAI-API-Aufrufe.
- Anfragen gehen an die konfigurierte Sonarr-Adresse. Sonarr kann je nach
  aufgerufenem Endpunkt selbst externe Dienste kontaktieren.
- Bekannte Passwort-/Schlüsselfelder und vertrauliche URL-Parameter werden
  maskiert. Beliebige Geheimnisse in Freitext werden nicht garantiert erkannt.
- Der API-Schlüssel hat die Berechtigungen deiner Sonarr-Installation;
  die Erweiterung erstellt keine zusätzlichen Sonarr-Rollen.
- HTTP-Weiterleitungen werden nicht verfolgt.
- Zugangsdaten, lokale Einstellungen, Sicherungen und Arbeitsdateien gehören
  nicht ins Repository.
- Formatpunkte sind Präferenzen, keine Messung der Bildqualität. Sonarr
  berücksichtigt die Dateigröße nach den Formatpunkten und weiteren Kriterien.

## Lizenz

GPL-3.0-only; siehe [LICENSE](LICENSE).

Die unverändert mitgelieferte Datei openapi.json stammt von Sonarr und steht
unter GPLv3. Siehe [SONARR-LICENSE](SONARR-LICENSE) und
[THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt).

Quellen:
- https://github.com/Sonarr/Sonarr
- https://modelcontextprotocol.io/specification/2025-06-18/basic/transports
