# Zabbix Template: Auerswald COMpact 4000/5000 by HTTP

> **🇩🇪 [Deutsche Version](#deutsche-version)** | **🇬🇧 [English Version](#english-version)**

---

## Deutsche Version

Zabbix 7.4 Monitoring-Template für Auerswald TK-Anlagen der COMpact/COMmander-Serie über die ICT System API (HTTPS/JSON mit Digest Auth).

**Autor:** Alexander Fox | PlaNet Fox

### Unterstützte Geräte

| Modell | pbxType | Getestet |
|--------|---------|----------|
| COMpact 4000 | 20 | ✅ FW 8.6C |
| COMpact 5000 | 12 | Kompatibel |
| COMpact 5000R | 13 | Kompatibel |
| COMpact 5200 | 33 | Kompatibel |
| COMpact 5200R | 34 | Kompatibel |
| COMpact 5500R | 35 | Kompatibel |
| COMmander 6000 | 10 | Kompatibel |
| COMmander 6000R/RX | 11 | Kompatibel |

> **Hinweis:** COMpact 4000 unterstützt weniger API-Endpunkte als die 5000er-Serie. Das Template erkennt 404-Antworten automatisch und ignoriert fehlende Endpunkte — ein Template für alle Modelle.

### Features

#### Überwachung (66 Items)
- **System:** Firmware, Seriennummer, MAC, Datum, PBX-Typ, Anlagenname
- **Netzwerk:** ICMP Ping/Latenz/Paketverlust, HTTPS-Status
- **API:** Authentifizierungs-Check, Account-Lock-Erkennung (423)
- **Amtsleitungen:** Anzahl gesamt, VoIP, ISDN, Namen
- **Externe Kanäle:** VoIP/ISDN getrennt — Gesamt, Aktiv, Gesperrt
- **MSN/DDI:** Rufnummern-Liste, Anzahl (gesamt/unique), aktive Umleitungen
- **SIP-Proxy:** DNS-Auflösung, Ping, Latenz, Port 5060+5061
- **Voicemail/Fax:** Speicher, Boxen, Nachrichten (auf 5000er)
- **Syslog:** SIP-Auth-Fehler, Errors, Neustarts (optional, deaktiviert)

#### Trigger (28 Stück)

| Priorität | Trigger |
|-----------|---------|
| DISASTER | Ping fehlgeschlagen |
| DISASTER | Alle VoIP-Kanäle gesperrt (SIP-Registrierung!) |
| HIGH | API Auth Fehler / Account gesperrt (423) |
| HIGH | Alle externen Kanäle belegt |
| HIGH | SIP-Proxy nicht erreichbar / DNS fehlgeschlagen |
| HIGH | Datum der Anlage stimmt nicht |
| WARNING | Hohe Latenz Anlage / SIP-Proxy |
| WARNING | Paketverlust >10% |
| WARNING | VoIP/ISDN-Kanäle gesperrt |
| WARNING | >80% Kanäle belegt |
| WARNING | VM-Speicher Warnung |
| INFO | Firmware geändert / Update verfügbar |
| INFO | Amtsleitungen geändert |

#### Graphen (5 Stück)
- Externe Kanäle (Gesamt/Aktiv/Gesperrt)
- VoIP-Kanäle
- Netzwerk Anlage (Latenz + Paketverlust)
- SIP-Proxy Latenz
- Amtsleitungen

### Voraussetzungen

- **Zabbix:** 7.0+ (getestet mit 7.4)
- **Firmware:** 8.x empfohlen (API Version 7)
- **Zugang:** Sub-Admin-Teilnehmer mit Web-/API-Passwort
- **HTTPS:** Anlage über HTTPS erreichbar (selbstsigniertes Cert OK)

### Installation

#### 1. Sub-Admin auf der Anlage einrichten
In der Auerswald Web-Oberfläche:

1. **Administration → Teilnehmer** → neuen Teilnehmer anlegen oder bestehenden wählen
2. **Berechtigungen** → Rolle auf **Sub-Admin** setzen
3. **Web-Passwort** vergeben (unter Passwörter → Web-Zugang)
4. Die **interne Rufnummer** dieses Teilnehmers notieren (z.B. `50`) — das ist der API-Benutzername

> **Tipp:** Einen dedizierten Teilnehmer für Zabbix anlegen (z.B. Nr. 50, Name "Zabbix"), damit API-Zugriffe im Syslog klar zugeordnet werden können.

#### 2. Template importieren
**Data collection → Templates → Import** → YAML-Datei hochladen

#### 3. Host anlegen

| Feld | Wert |
|------|------|
| Host name | z.B. `COMpact 4000 Büro` |
| Templates | `Auerswald COMpact 5000 by HTTP` |
| Host groups | z.B. `Telephony` |
| Agent interface | IP der Anlage, Port 10050 |

> **Wichtig:** Host benötigt ein Agent-Interface mit der IP der Anlage für ICMP-Checks. Zabbix Agent muss dort nicht laufen.

#### 4. Macros am Host setzen

| Macro | Wert | Beschreibung |
|-------|------|-------------|
| `{$AUERSWALD.IP}` | `192.168.1.240` | IP der Anlage |
| `{$AUERSWALD.USER}` | `<Nr>` | Sub-Admin interne Rufnummer |
| `{$AUERSWALD.PASSWORD}` | *(Secret)* | Web-/API-Passwort (Typ: Secret) |
| `{$AUERSWALD.SIP.PROXY}` | `s<Serial>.pbx.auerproxy.de` | SIP-Proxy Hostname |

#### 5. Optional: Syslog aktivieren
Für erweiterte Fehler-Erkennung (SIP-Auth-Fehler, Neustarts):

**Auf dem Zabbix Proxy/Server:**
```bash
# /etc/rsyslog.d/auerswald.conf
if $fromhost-ip == '192.168.1.240' then /var/log/auerswald.log
& stop
```
```bash
systemctl restart rsyslog
```

**Auf der Anlage:** Administration → Netzwerk → Syslog-Server → IP des Proxy/Servers eintragen.

**In Zabbix:** Die 5 Syslog-Items im Template aktivieren (standardmäßig deaktiviert). UDP-Empfang auf Port 514 muss in `/etc/rsyslog.conf` aktiviert sein:
```bash
module(load="imudp")
input(type="imudp" port="514")
```

### Alle Macros

| Macro | Default | Beschreibung |
|-------|---------|-------------|
| `{$AUERSWALD.IP}` | `192.168.1.240` | IP-Adresse der Anlage |
| `{$AUERSWALD.PORT}` | `443` | HTTPS-Port |
| `{$AUERSWALD.USER}` | *(leer)* | Sub-Admin Nr (interne Rufnummer) |
| `{$AUERSWALD.PASSWORD}` | *(Secret)* | Web-/API-Passwort |
| `{$AUERSWALD.PING.WARN}` | `0.1` | Ping-Warnschwelle in Sekunden |
| `{$AUERSWALD.VMF_STORAGE.WARN}` | `80` | VM-Speicher Warnung (%) |
| `{$AUERSWALD.VMF_STORAGE.CRIT}` | `95` | VM-Speicher Kritisch (%) |
| `{$AUERSWALD.SIP.PROXY}` | *(leer)* | SIP-Proxy FQDN |
| `{$AUERSWALD.SIP.PORT}` | `5060` | SIP-Port |
| `{$AUERSWALD.SYSLOG.PATH}` | `/var/log/auerswald.log` | Pfad zur Syslog-Datei |
| `{$AUERSWALD.FIRMWARE.URL}` | `https://www.auerswald.de/de/support/produkt/compact-4000/support` | URL der Auerswald-Support-Seite für Firmware-Check. **Pro Modell anpassen!** z.B. `compact-5000`, `compact-5200`, `compact-5500r`, `commander-6000` |

### API-Endpunkte

| Endpunkt | Auth | 4000 | 5000+ | Beschreibung |
|----------|------|------|-------|-------------|
| `/app_about` | Nein | ✅ | ✅ | Systeminfo |
| `/app_amt_list` | Digest | ✅ | ✅ | Amtsleitungen |
| `/app_ext_ports_status` | Digest | ✅ | ✅ | Externe Ports/Kanäle |
| `/app_msn_aws_list` | Digest | ✅ | ✅ | MSN/DDI-Rufnummern |
| `/app_call_list` | Digest | ✅ | ✅ | Anrufliste |
| `/app_alarm_list` | Digest | ✅ | ✅ | Alarme |
| `/app_relais_list` | Digest | ✅ | ✅ | Relais |
| `/app_vmf_box_list` | Digest | ✅ | ✅ | Voicemail/Fax-Boxen |
| `/app_vmf_msg_list` | Digest | ✅ | ✅ | VM/Fax-Nachrichten |
| `/app_conversation_data` | Digest | ❌ | ✅ | Aktive Gespräche |
| `/app_tln_props` | Digest | ❌ | ✅ | Teilnehmer |
| `/app_grp_state` | Digest | ❌ | ✅ | Gruppen |
| `/app_config_list` | Digest | ❌ | ✅ | Konfigurationen |
| `/app_get_alarms` | Digest | ❌ | ✅ | Aktive Alarme |
| `/app_relais_status` | Digest | ❌ | ✅ | Relais-Status |
| `/app_vmf_storage_overview` | Digest | ❌ | ✅ | VM-Speicher |
| `/app_telefonbuch_cat` | Digest | ❌ | ✅ | Telefonbuch |

### Troubleshooting

#### 423 - Account gesperrt
Die Anlage sperrt nach zu vielen fehlgeschlagenen Auth-Versuchen. Jeder HTTP-Request mit Digest Auth startet mit einem 401 (normal für Digest), aber die Anlage zählt das als Fehlversuch. Das Template behandelt 423 automatisch — bei Ersteinrichtung den Host deaktiviert lassen bis Credentials geprüft sind.

**Manuell testen:**
```bash
curl -k --digest -u <Nr>:<PW> https://<IP>/app_amt_list
```

#### 404 - Endpoint nicht verfügbar
COMpact 4000 unterstützt weniger Endpunkte. Template akzeptiert 404 automatisch — kein Eingriff nötig.

#### 401 - Auth fehlgeschlagen
- Benutzername ist die **interne Rufnummer** (nicht der Anzeigename)
- Passwort ist das **Web-Passwort** (nicht das Telefon-Passwort)
- Sub-Admin benötigt Berechtigung für API-Zugriff

#### Kein Ping
Host braucht ein Agent-Interface mit der IP der Anlage. Agent muss nicht laufen.

### Tags

**Template:** `class=telephony`, `target=auerswald`, `type=pbx`

**Items:** `component` = system, network, api, channels, sip-proxy, msn, voicemail, exchanges, syslog, ...

**Trigger:** `scope` = availability, performance, security, capacity, notice + `component`

### Kompatibilität

| Zabbix Version | Status |
|---------------|--------|
| 7.4 | ✅ Getestet |
| 7.0 LTS | Sollte funktionieren |
| 6.x | ❌ Inkompatibel |

---

## English Version

Zabbix 7.4 monitoring template for Auerswald PBX systems (COMpact/COMmander series) via ICT System API (HTTPS/JSON with Digest Auth).

**Author:** Alexander Fox | PlaNet Fox

### Supported Devices

| Model | pbxType | Tested |
|-------|---------|--------|
| COMpact 4000 | 20 | ✅ FW 8.6C |
| COMpact 5000 | 12 | Compatible |
| COMpact 5000R | 13 | Compatible |
| COMpact 5200 | 33 | Compatible |
| COMpact 5200R | 34 | Compatible |
| COMpact 5500R | 35 | Compatible |
| COMmander 6000 | 10 | Compatible |
| COMmander 6000R/RX | 11 | Compatible |

> **Note:** COMpact 4000 supports fewer API endpoints than the 5000 series. The template automatically detects 404 responses and gracefully ignores missing endpoints — one template for all models.

### Features

#### Monitoring (66 Items)
- **System:** Firmware, serial number, MAC, date, PBX type, device name
- **Network:** ICMP ping/latency/packet loss, HTTPS status
- **API:** Authentication check, account lock detection (423)
- **Exchanges:** Total count, VoIP, ISDN, names
- **External Channels:** VoIP/ISDN separate — Total, Active, Blocked
- **MSN/DDI:** Phone number list, count (total/unique), active call forwarding
- **SIP Proxy:** DNS resolution, ping, latency, port 5060+5061
- **Voicemail/Fax:** Storage, boxes, messages (5000 series only)
- **Syslog:** SIP auth failures, errors, reboots (optional, disabled by default)

#### Triggers (28)

| Severity | Trigger |
|----------|---------|
| DISASTER | Ping failed |
| DISASTER | All VoIP channels blocked (SIP registration failure!) |
| HIGH | API auth failure / Account locked (423) |
| HIGH | All external channels busy |
| HIGH | SIP proxy unreachable / DNS failed |
| HIGH | PBX date incorrect |
| WARNING | High latency (PBX / SIP proxy) |
| WARNING | Packet loss >10% |
| WARNING | VoIP/ISDN channels blocked |
| WARNING | >80% channels busy |
| WARNING | VM storage warning |
| INFO | Firmware changed / update available |
| INFO | Exchange count changed |

#### Graphs (5)
- External Channels (Total/Active/Blocked)
- VoIP Channels
- Network PBX (Latency + Packet Loss)
- SIP Proxy Latency
- Exchanges

### Requirements

- **Zabbix:** 7.0+ (tested with 7.4)
- **Firmware:** 8.x recommended (API version 7)
- **Access:** Sub-admin subscriber with web/API password
- **HTTPS:** PBX reachable via HTTPS (self-signed cert OK)

### Installation

#### 1. Create Sub-Admin on the PBX
In the Auerswald web interface:

1. **Administration → Subscribers** → create a new subscriber or select an existing one
2. **Permissions** → set role to **Sub-Admin**
3. **Set a web password** (under Passwords → Web Access)
4. Note the **internal extension number** of this subscriber (e.g. `50`) — this is the API username

> **Tip:** Create a dedicated subscriber for Zabbix (e.g. ext. 50, name "Zabbix") so API access is clearly identifiable in the syslog.

#### 2. Import Template
**Data collection → Templates → Import** → Upload YAML file

#### 3. Create Host

| Field | Value |
|-------|-------|
| Host name | e.g. `COMpact 4000 Office` |
| Templates | `Auerswald COMpact 5000 by HTTP` |
| Host groups | e.g. `Telephony` |
| Agent interface | PBX IP address, port 10050 |

> **Important:** Host requires an Agent interface with the PBX IP for ICMP checks. Zabbix Agent does not need to be running on the device.

#### 4. Set Host Macros

| Macro | Value | Description |
|-------|-------|-------------|
| `{$AUERSWALD.IP}` | `192.168.1.240` | PBX IP address |
| `{$AUERSWALD.USER}` | `<Nr>` | Sub-admin extension number |
| `{$AUERSWALD.PASSWORD}` | *(Secret)* | Web/API password (Type: Secret) |
| `{$AUERSWALD.SIP.PROXY}` | `s<Serial>.pbx.auerproxy.de` | SIP proxy hostname |

#### 5. Optional: Enable Syslog
For advanced error detection (SIP auth failures, reboots):

**On the Zabbix Proxy/Server:**
```bash
# /etc/rsyslog.d/auerswald.conf
if $fromhost-ip == '192.168.1.240' then /var/log/auerswald.log
& stop
```
```bash
systemctl restart rsyslog
```

**On the PBX:** Administration → Network → Syslog Server → Enter proxy/server IP.

**In Zabbix:** Enable the 5 syslog items in the template (disabled by default). UDP reception on port 514 must be enabled in `/etc/rsyslog.conf`:
```bash
module(load="imudp")
input(type="imudp" port="514")
```

### All Macros

| Macro | Default | Description |
|-------|---------|-------------|
| `{$AUERSWALD.IP}` | `192.168.1.240` | PBX IP address |
| `{$AUERSWALD.PORT}` | `443` | HTTPS port |
| `{$AUERSWALD.USER}` | *(empty)* | Sub-admin extension number |
| `{$AUERSWALD.PASSWORD}` | *(Secret)* | Web/API password |
| `{$AUERSWALD.PING.WARN}` | `0.1` | Ping warning threshold (seconds) |
| `{$AUERSWALD.VMF_STORAGE.WARN}` | `80` | VM storage warning (%) |
| `{$AUERSWALD.VMF_STORAGE.CRIT}` | `95` | VM storage critical (%) |
| `{$AUERSWALD.SIP.PROXY}` | *(empty)* | SIP proxy FQDN |
| `{$AUERSWALD.SIP.PORT}` | `5060` | SIP port |
| `{$AUERSWALD.SYSLOG.PATH}` | `/var/log/auerswald.log` | Syslog file path |
| `{$AUERSWALD.FIRMWARE.URL}` | `https://www.auerswald.de/de/support/produkt/compact-4000/support` | URL of the Auerswald support page for firmware check. **Adjust per model!** e.g. `compact-5000`, `compact-5200`, `compact-5500r`, `commander-6000` |

### API Endpoints

| Endpoint | Auth | 4000 | 5000+ | Description |
|----------|------|------|-------|-------------|
| `/app_about` | No | ✅ | ✅ | System info |
| `/app_amt_list` | Digest | ✅ | ✅ | Exchanges |
| `/app_ext_ports_status` | Digest | ✅ | ✅ | External ports/channels |
| `/app_msn_aws_list` | Digest | ✅ | ✅ | MSN/DDI numbers |
| `/app_call_list` | Digest | ✅ | ✅ | Call list |
| `/app_alarm_list` | Digest | ✅ | ✅ | Alarms |
| `/app_relais_list` | Digest | ✅ | ✅ | Relays |
| `/app_vmf_box_list` | Digest | ✅ | ✅ | Voicemail/fax boxes |
| `/app_vmf_msg_list` | Digest | ✅ | ✅ | VM/fax messages |
| `/app_conversation_data` | Digest | ❌ | ✅ | Active conversations |
| `/app_tln_props` | Digest | ❌ | ✅ | Subscribers |
| `/app_grp_state` | Digest | ❌ | ✅ | Groups |
| `/app_config_list` | Digest | ❌ | ✅ | Configurations |
| `/app_get_alarms` | Digest | ❌ | ✅ | Active alarms |
| `/app_relais_status` | Digest | ❌ | ✅ | Relay status |
| `/app_vmf_storage_overview` | Digest | ❌ | ✅ | VM storage |
| `/app_telefonbuch_cat` | Digest | ❌ | ✅ | Phonebook |

### Troubleshooting

#### 423 - Account Locked
The PBX locks accounts after too many failed auth attempts. Each Digest Auth request starts with a 401 challenge (normal for Digest), but the PBX counts this as a failed attempt. The template handles 423 automatically — keep the host disabled until credentials are verified.

**Manual test:**
```bash
curl -k --digest -u <ext>:<pw> https://<IP>/app_amt_list
```

#### 404 - Endpoint Not Available
COMpact 4000 supports fewer endpoints. Template accepts 404 automatically — no action needed.

#### 401 - Auth Failed
- Username is the **internal extension number** (not the display name)
- Password is the **web password** (not the phone password)
- Sub-admin requires API access authorization

#### No Ping
Host needs an Agent interface with the PBX IP. Agent does not need to be running.

### Tags

**Template:** `class=telephony`, `target=auerswald`, `type=pbx`

**Items:** `component` = system, network, api, channels, sip-proxy, msn, voicemail, exchanges, syslog, ...

**Triggers:** `scope` = availability, performance, security, capacity, notice + `component`

### Compatibility

| Zabbix Version | Status |
|---------------|--------|
| 7.4 | ✅ Tested |
| 7.0 LTS | Should work |
| 6.x | ❌ Incompatible |

---

## Changelog

### v3.1 (2026-04-23)

**Bugfixes**
- JSONPath Typ-Konsistenz: `/app_ext_ports_status` liefert laut API-Doku **Strings** (`anschlArt="2"`, `bKanalStatus="0"` etc.). Alle VoIP/ISDN-Filter nutzen jetzt durchgängig String-Vergleich — vorher waren die VoIP/ISDN-Subcounts durch Integer-Vergleich dauerhaft `0`, wodurch u.a. der DISASTER-Trigger „ALLE VoIP-Kanäle gesperrt" nie feuern konnte
- VM-Speicher Trigger-Dependency gedreht: WARNING hängt jetzt von KRITISCH ab (vorher umgekehrt → KRITISCH wurde von WARNING unterdrückt)
- HIGH-Trigger „API Auth nicht verfügbar" hängt jetzt zusätzlich vom DISASTER-Lock-Trigger ab — keine parallelen Alarme mehr bei 423
- `change()`-Trigger (Amtsleitungen, Firmware) gegen API-Aussetzer abgesichert: Raw-Items werfen bei Blocked/Parse-Fehler eine Exception, `DISCARD_VALUE` greift, Dependents behalten letzten gültigen Wert
- `system.date.ok`: robustere JS mit try/catch, matcht auch wenn API zusätzlich eine Uhrzeit mitliefert
- MSN-Items: `Array.isArray`-Guard, `Number()`-Coercion bei Typenvergleichen
- Firmware-Regex akzeptiert jetzt auch 3-stellige Versionen und Suffixes (`8.6.1`, `8.6C-SP1`)

**Features**
- Neues Macro `{$AUERSWALD.FIRMWARE.URL}` — Firmware-Check-URL pro Modell konfigurierbar (vorher hardcoded auf COMpact 4000)
- Ping-Dependency bei Latenz-Triggern (Anlage + SIP-Proxy) gegen Recovery-Rauschen
- `value_type: UNSIGNED` explizit bei Zähler-Items für Klarheit

### v3.0 (2026-02-25)
- Zabbix 7.4 format (UUIDv4, master_item, authtype, template_groups)
- Single template for COMpact 4000/5000/5200/5500R (404 graceful handling)
- API auth + account lock detection (423)
- External ports split by VoIP/ISDN
- MSN/DDI phone number list, unique count, active forwarding detection
- SIP proxy monitoring (DNS/ping/ports)
- PBX date drift detection
- Channel overload warnings
- Firmware update check
- Syslog integration (optional)
- Comprehensive tags (scope + component)
- 5 graphs

### v2.0 (2026-02-24)
- Initial template with all API endpoints

## License

MIT

## Links

- [Auerswald ICT System API](https://wiki.auerswald.de/compact-4000-5x00-commander-6000/developer_documentation/ict_system_api/)
- [Auerswald Node-RED Integration](https://github.com/Auerswald-GmbH/node-red-contrib-auerswald)