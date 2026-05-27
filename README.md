# Mein Enterprise Homelab – Infrastruktur & Automatisierung

Dieses Repository dokumentiert den schrittweisen Aufbau meines privaten IT-Infrastruktur-Labors. Ziel des Projekts ist es, eine realistische Unternehmensnetzwerk-Struktur (KMU) zu simulieren, abzusichern und zu automatisieren.

---

## Phase 1: Hypervisor & Virtualisierungs-Infrastruktur

Als Basis für das gesamte Labor dient ein natives Linux-System mit einem Typ-1-nahen Hypervisor (KVM/QEMU). Dies sorgt für maximale Performance auf der Hardware und entspricht modernen Enterprise-Standards im Open-Source-Bereich.

### Architektur des Hypervisors:
* **Host-Betriebssystem:** Ubuntu Linux
* **Hypervisor:** KVM (Kernel-based Virtual Machine) & QEMU
* **Management-Tools:** Libvirt, Virt-Manager (CLI & GUI)
* **Standard-Netzwerk:** NAT-Netzwerk (192.168.122.0/24) via `dnsmasq`

### Durchgeführte Schritte & Fehlerbehebung (Troubleshooting):
1. Validierung der CPU-Virtualisierungskompatibilität (`vmx`/`svm`) über `/proc/cpuinfo`.
2. Installation der Enterprise-Kernpakete (`qemu-system-x86`, `libvirt-daemon-system`, `virt-manager`).
3. Berechtigungssteuerung: Zuweisung des lokalen Systembenutzers zu den privilegierten Gruppen `libvirt` und `kvm`.
4. **Troubleshooting:** Behebung eines Systemd-Initialisierungsfehlers des Monolithic Daemons (`libvirtd.service` schlug fehl aufgrund von `User record for user 'libvirt-qemu' was not found`). Der Fehler wurde durch manuelles Nachpflegen des Systembenutzers `libvirt-qemu` im OS gelöst, gefolgt von einer persistenten Aktivierung des Dienstes via `systemctl enable --now libvirtd`.

---

## Phase 2: Linux Core-Netzwerk & Zentrale Dienste

In dieser Phase wurde das logische Kernnetzwerk des Labors aufgesetzt. Alle Server werden als minimale Instanzen ohne grafische Oberfläche (Headless) betrieben, um die Administration ausschließlich auf CLI-Ebene zu trainieren.

### VM 1: Core-Server (`srv-core-01`)
* **Betriebssystem:** Debian GNU/Linux 13 (Trixie) – Minimal Installation (CLI)
* **Ressourcen:** 2 vCPUs, 2 GB RAM, 20 GB vDisk
* **Statische IP-Konfiguration:** `192.168.122.200/24` | Gateway: `192.168.122.1`
* **Bereitgestellte Dienste:** * **DNS-Server (BIND9):** Aktiv (Authoritative für `homelab.local` / Forwarding für externes Routing)

### Durchgeführte Schritte & Troubleshooting (Phase 2):
1. **Speicherpool-Bereitstellung:** Behebung von *Permission denied*-Restriktionen des Systembenutzers (`libvirt-qemu`) durch Verschieben des Debian-ISOs in den globalen libvirt-Speicherpool (`/var/lib/libvirt/images/`) und Anpassung der Besitzrechte via `chown`.
2. **Netzwerk-Härtung (`/etc/network/interfaces`):** Umstellung des primären Interfaces (`enp1s0`) auf eine statische Konfiguration. Behebung von Syntax-Konflikten über die serielle Virt-Manager-Konsole.
3. **DNS-Infrastruktur:** Konfiguration einer Forward-Zone (`db.homelab.local`) mit korrekten SOA- und A-Records. Aktivierung von DNS-Forwarding in `named.conf.options`.

---

## Phase 3: Dynamische Adressvergabe (Kea DHCP)

In dieser Phase wurde die automatische Netzwerkkonfiguration innerhalb des Homelabs realisiert, um neue Instanzen vollautomatisch mit passenden IP-Parametern zu versorgen.

### Bereitgestellter Dienst:
* **DHCP-Server:** Kea DHCPv4 Server (Modern Enterprise Standard)
* **Zugeordnetes Interface:** `enp1s0`

### Konfigurations-Parameter (Scope):
* **Subnetz:** `192.168.122.0/24`
* **Dynamischer IP-Pool:** `192.168.122.50` bis `192.168.122.150`
* **Ausgegebene Optionen an Clients:**
  * **Standard-Gateway (Routers):** `192.168.122.1`
  * **Zentraler DNS-Server:** `192.168.122.200` (Verweis auf `srv-core-01`)
  * **Domain-Name:** `homelab.local`

### Durchgeführte Schritte & Troubleshooting (Phase 3):
1. **Paket-Installation:** Einspielen des `kea-dhcp4-server`-Pakets aus den Debian-Repositories.
2. **JSON-Strukturierung & Fehlerbehebung:** * Behebung eines *Parser-Fehlers* durch Definition der zwingend erforderlichen Subnetz-`id` im JSON-Objekt.
   * Eliminierung eines *Systemd-Startkonflikts* (`exit-code 1`), verursacht durch Inkompatibilität des Parameters `"socket-type": "stdout"`. Die Steuerung wurde erfolgreich auf das native systemd-journal-basierte Logging umgestellt.
3. **Rechte-Härtung:** Anpassung der Besitzer- und Verzeichnisrechte der Konfigurationsdatei (`/etc/kea/kea-dhcp4.conf`) auf den Debian-spezifischen Systembenutzer `_kea`.

---
*Nächster Meilenstein: Phase 4 – Aufbau eines zentralen Verzeichnisdienstes (OpenLDAP/Samba AD) zur Benutzer- und Rechteverwaltung.*
