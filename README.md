# Projekt: Netzwerkweiter Werbeblocker (Pi-hole) 

## 1. Ausgangslage & Zielsetzung
Ziel dieses Projekts war die Implementierung eines lokalen DNS-Servers (Pi-hole) zur netzwerkweiten Blockierung von Werbe- und Tracking-Domains. Anstatt auf jedem Endgerät einzeln Werbeblocker zu installieren, filtert diese Lösung unerwünschte Anfragen direkt auf Netzwerkebene heraus, spart Bandbreite und schützt die Privatsphäre.

## 2. Systemumgebung
* **Hardware:** Raspberry Pi Zero W, 64GB MicroSD-Karte
* **Betriebssystem:** Raspberry Pi OS Lite (32-bit) – Headless-Setup (ohne grafische Oberfläche)
* **Software:** Pi-hole (DNS-Sinkhole)
* **Netzwerkinfrastruktur:** Fritz!Box (als Router & DHCP-Server), lokaler Windows-PC zur Remote-Administration

---

## 3. Umsetzung & Meilensteine

### Phase 1: Headless-Provisionierung
Um den Server ressourcenschonend und ohne Monitor/Tastatur in Betrieb zu nehmen, wurde das Betriebssystem per Raspberry Pi Imager geflasht und vorkonfiguriert.
* Auswahl von Raspberry Pi OS Lite (32-bit).
* Vorkonfiguration via Imager: Vergabe eines Hostnamens, Anlage des Administrator-Benutzers, Konfiguration der WLAN-Zugangsdaten und Aktivierung der SSH-Authentifizierung.

### Phase 2: Remote-Zugriff & Fehleranalyse
Nach dem Boot-Vorgang wurde die IP-Adresse des Raspberry Pi über das Web-Interface des Routers ermittelt. Bei der ersten Verbindungsaufnahme traten typische Netzwerk-Herausforderungen auf, die systematisch analysiert und gelöst wurden:

* **Problem 1: `Connection timed out`**
  * *Analyse:* Ping-Test ergab keine Antwort des Hosts.
  * *Lösung:* Überprüfung der Netzwerkverbindung und IP-Zuweisung.
* **Problem 2: `Connection refused` auf Port 22**
  * *Analyse:* Der SSH-Daemon startete nicht, da die Aktivierung im Imager nicht griff.
  * *Lösung:* Manuelles Eingreifen in die Boot-Partition unter Windows. Da Windows Dateiendungen oft versteckt, wurden diese eingeblendet, um eine komplett endungsfreie Datei namens `ssh` im `bootfs`-Verzeichnis anzulegen. Dies zwang das Linux-System beim nächsten Start, den SSH-Port freizugeben.

Der folgende Screenshot zeigt den Troubleshooting-Prozess im Terminal bis zur finalen, erfolgreichen SSH-Authentifizierung (Trust on first use):

![SSH Fehleranalyse und Login](<versuch 1 (3).png>)

### Phase 3: Installation & Konfiguration von Pi-hole
Nach erfolgreichem SSH-Login wurde die Installation von Pi-hole über das offizielle Shell-Skript gestartet:

`curl -sSL https://install.pi-hole.net | bash`

![Start der Pi-hole Installation](<Screenshot 2026-07-19 195514_2.png>)

Während des Setups wurden über die textbasierte Benutzeroberfläche (ncurses) folgende Kern-Konfigurationen vorgenommen:

**1. Zuweisung des Upstream DNS-Providers:**
![Upstream DNS Auswahl](<Screenshot 2026-07-19 195838_2.png>)

**2. Integration der Standard-Blocklisten:**
![Blocklisten Integration](<Screenshot 2026-07-19 200250_2.png>)

**3. Aktivierung des Query Loggings (Privacy Level 0), um geblockte Anfragen für Troubleshooting-Zwecke einsehen zu können:**
![Privacy Level Auswahl](<Screenshot 2026-07-19 200517_2.png>)

Die Installation wurde erfolgreich abgeschlossen und das Webinterface bereitgestellt:

![Installation abgeschlossen](<Screenshot 2026-07-19 200734_2.png>)

### Phase 4: Netzwerkintegration (DHCP-Konfiguration)
Damit nicht jedes Endgerät manuell konfiguriert werden muss, wurde der DHCP-Server des Routers (Fritz!Box) umgestellt. Folgende Schritte wurden durchgeführt:

1. **Auf der Fritz!Box einloggen:** Zugriff auf das Kommandozentrum des Heimnetzwerks über `http://fritz.box`.
2. **Erweiterte Ansicht aktivieren:** Notwendig, um tiefere Netzwerkeinstellungen sichtbar zu machen.
3. **IPv4-Einstellungen öffnen:** Navigation über `Heimnetz` -> `Netzwerk` -> `Netzwerkeinstellungen` zum DHCP-Server.
4. **Den Pi-hole als DNS eintragen:** Austausch des standardmäßigen lokalen DNS-Servers (IP der Fritz!Box) durch die statische IP des Raspberry Pi (`192.168.178.108`).
5. **Speichern und bestätigen:** Übernahme der Einstellungen im Router.

![FritzBox DNS Konfiguration](<Screenshot 2026-07-19 205356.png>)

Nach Erneuerung des DHCP-Leases (Re-Connect der Endgeräte) liefen alle DNS-Anfragen erfolgreich über den Pi-hole.

---

## 4. Learnings & Ausblick

Neben der grundlegenden Erfahrung in der Linux-Kommandozeile (Bash) und der Fehlersuche bei SSH-Verbindungen, war die wichtigste Erkenntnis die Architektur-Planung:

**Ausfallsicherheit (Redundanz):** 
Da der Pi-hole nun als einziger DNS-Server im Netzwerk fungiert (Single Point of Failure), führt ein Hardware- oder Stromausfall des Raspberry Pi unweigerlich zum Ausfall der Internetauflösung für alle Geräte im Netzwerk. In einem produktiven Firmennetzwerk würde ich zur Absicherung (Failover) zwingend einen zweiten DNS-Server (z.B. einen weiteren Pi-hole) aufsetzen, um Hochverfügbarkeit (High Availability) zu garantieren.
