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
*Nächster Schritt: Bereitstellung der ersten Core-Server-VM (Headless) für zentrale Netzwerkdienste (DNS/DHCP).*
