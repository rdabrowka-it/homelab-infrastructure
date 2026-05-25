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
---

## Phase 2: Linux Core-Netzwerk & Zentrale Dienste

In dieser Phase wird das logische Kernnetzwerk des Labors aufgebaut. Alle Server werden als minimale Instanzen ohne grafische Oberfläche (Headless) betrieben, um maximale Performance zu gewährleisten und die Administration ausschließlich auf CLI-Ebene (SSH) zu trainieren.

### VM 1: Core-Server (`srv-core-01`)
* **Betriebssystem:** Debian GNU/Linux 13 (Trixie) – Minimal Installation (CLI)
* **Ressourcen:** 2 vCPUs, 2 GB RAM, 20 GB vDisk
* **Statische IP-Konfiguration:** `192.168.122.200/24` | Gateway: `192.168.122.1`
* **Bereitgestellte Dienste:** * **DNS-Server (BIND9):** Aktiv (Authoritative für `homelab.local` / Caching & Forwarding für externes Routing)
  * **DHCP-Server (Kea DHCP):** *In Vorbereitung*

### Durchgeführte Schritte:

1. **Speicherpool-Bereitstellung:** Aufgrund von Restriktionen des Hypervisor-Systembenutzers (`libvirt-qemu`) beim Zugriff auf externe Mount-Pfade (*Permission denied*), wurde das Debian-ISO-Abbild nach Best-Practice-Standard direkt in den globalen libvirt-Speicherpool (`/var/lib/libvirt/images/`) verschoben. Die Dateibesitzrechte wurden angepasst (`chown libvirt-qemu:kvm`).
2. **OS-Deployment:** Geführte Installation des Linux-Kernels über den KVM-Virtualisierungs-Assistenten.
3. **Software-Härtung:** Bewusste Abwahl aller X11/Desktop-Umgebungen (GNOME/XFCE). Es wurden ausschließlich die Pakete `SSH server` und `Standard-Systemwerkzeuge` installiert.
4. **Netzwerk-Härtung (`/etc/network/interfaces`):** Umstellung des primären Interfaces (`enp1s0`) von dynamischem DHCP auf eine feste, statische IP-Konfiguration. Erfolgreiches Troubleshooting bei Syntax-Konflikten (Typo-Korrektur der `address`-Direktive) direkt über die serielle Virt-Manager-Konsole.
5. **DNS-Infrastruktur (BIND9):** * Einrichtung als *Authoritative Nameserver* für die lokale Labor-Domain `homelab.local` mit Forward-Zonendatei (`db.homelab.local`).
   * Konfiguration von DNS-Forwarding in den globalen Optionen zur nahtlosen Weiterleitung externer Internet-Anfragen über das KVM-Gateway.
6. **Lokale Resolver-Anpassung (`/etc/resolv.conf`):** Systemweite DNS-Priorisierung auf den lokalen Loopback (`127.0.0.1`) umgestellt und Such-Domain (`search homelab.local`) für Kurznamen-Auflösung implementiert.

### Validierung & Funktionstests:
* Syntax-Prüfung via `named-checkconf` und `named-checkzone` fehlerfrei durchgeführt.
* Lokale Namensauflösung des FQDN sowie des Kurznamens (`srv-core-01`) via `nslookup` erfolgreich verifiziert.
* Externes Routing und Caching mittels `ping google.com` erfolgreich bestätigt.

---
*Nächster Meilenstein: Implementierung von Kea DHCP zur Automatisierung der IP-Adressvergabe innerhalb des Labor-Subnetzes.*

