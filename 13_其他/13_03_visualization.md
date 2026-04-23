

# 1 Virtualisierung in der IT

verschiedene unabhängige Begriffsverwendungen
• Webserver (virtual host): ein Serverprozess realisiert mehrere unabhängige Websites, unterschieden durch den Host:-Header
    • Ziel: gemeinsame Nutzung von Hardware und IP-Adressen

• Programmiersprachen (virtual machine): z.B. Java; Software emuliert einen fiktiven Mikroprozessor (Java Bytecode)
    • Ziel: write once, run anywhere
    - 像是 jvm, java 运行在一个 process 中 

• Computer (virtual machine): Software (Hypervisor) erzeugt die Illusion von mehreren Computersystemen auf Basis eines Computers
    • Ziel: gemeinsame Nutzung von Hardware, Isolation
    • Zentrale Begriffe: Hostsystem ( physische Hardware), ==Gastsystem (virtualler PC)==


# 2 Emulation vs. Simulation

• Emulation: Gastsystem hat eigenen Mikroprozessor
    • Prozessor des Hostsystems muss jeden Befehl emulieren, durch eine Reihe
• Simulation: Gastsystem verwendet den gleichen Prozessor wie Hostsystem
    • Vorteil: höhere Performance - jeder Befehl des Gastsystems benötigt (i.d.R.) nur einen Befehl des Hostsystems
    • Nachteil: beide Prozessoren müssen übereinstimmen
        • z.B. keine virtuelle MS-Windows-Installation auf einem ARM-Prozessor


# 3 Herausforderungen

• Hardwarezugriff
    • Gastsystem führt Befehle aus, um auf die Hardware zuzugreifen; das soll aber i.d.R. nur virtuelle Hardware sein (virtuelle Festplatte, virtuelle Grafikkarte, virtuelle Netzwerkkarte)
    • erfordert i.d.R. Emulation
• privilegierte Anweisungen: Betriebssystem des Gastes möchte in privilegierten Modus wechseln (z.B. Systemrufe, Interrupts); das würde die Integrität des Hostsystems gefährden
    • privilegierte Anweisungen im Gast lösen einen Trap im Host aus; Hypervisor muss Anweisung emulieren
    - das würde die Integrität des Hostsystems gefährden:  所以这是不被允许的 
• virtueller Speicher: Gastsystem möchte page tables umkonfigurieren für jeden Prozess im Gastsystem; würde ebenfalls Integrität gefährden


# 4 Paravirtualisierung

• Gastsystem "weiß", dass es in einer virtuellen Maschine läuft
• Hypervisor stellt "hyper calls" zur Verfügung: spezielle privilegierte Befehle für das Gastsystem
    • Gastsystem testet beim Booten, 0b es in einem bekannten Hypervisor läuft
• Performanter als Emulation eines Hardware-Controllers
• Verwendet für Festplatte, Grafikkarte, Netzwerkkarte, Uhrensynchronisation,



# 5 Prozessorunterstützung

• Manche privilegierte Operationen können nur sehr ineffizient emuliert werden
    • z.B. Manipulation der page tables
• Hardwareunterstützung im Prozessor für Performance-Verbesserung
    • VM Control Structure (VMCS), zum schnellen Wechsel zwischen VMS 
    • Nested paging (z.B. Intel VT-x "extended page tables")
    • selektiv direkte "Freischaltung" von Hardware für eine VM
        -  Diese virtuelle Maschine kriegt eine Freischaltung für den und den Bereich.
    • Nested virtualization (VMS in VMs)


# 6 Hypervisor-Produkte

• VMWare ESXi (Broadcom)
• "bare metal" (Hypervisor ist das Host-Betriebssystem)
• Hyper-V (Microsoft)
• Kernel Virtual Machines (KVM, Linux), Proxmox
• xen, Oracle VM, Citrix
• VirtualBox
• Parallels (macOS)
• QEMU: kann auch Prozessoren emulieren
- qemu 

# 7 Zusatzfunktionen
• Management: VM life cycle
    • ggf. Web-basiert mit Konsole im Webbrowser
    • Snapshots
    • Resourcenzuordnung
• Cluster Management: mehrere Hosts betreiben kollektiv VMS
- Migration einer VM von einem Host zu einem anderen



# 8 Container

• für viele Anwendungen ist virtuelle Hardware nicht relevant
    • Ziel: Abschottung von Anwendungen gegeneinander
    • Ziel: separate IP-Adressen
    • Ziel: Ressourcenbeschränkung (Hauptspeicher, CPU, Festplatte)
• Container: Isolationskonzept ohne Virtualisierung von Hardware
    • alle Prozesse greifen auf denselben Kern zu (kein Gastsystem)
    • Kern stellt separaten Dateibaum und Netzzugang zur Verfügung
• Managementsoftware: Docker, Kubernetes, ...




# 9 问答

1 

noch mal zu bare metal: bedeutet, dass ich da gar nicht erst ein system installieren muss? Was läuft dann? eine bestimmte eigen linux distro von vmware?

proxmox z.B. ist hypervisor, aber die basis ist einfach ein debian
Ja, das VE ist das Debian, was direkt auf dem Metall läuft.
dann läuft da auch nur softwar drauf, die dafür compiliert wurde?

ESXi ist wie ein DOS auf dem Metall. Man administriert das dann meist mit einer vCenter Appliance.
Der ESXi Host hat auch eine Web-Appliance auf dem System. Die ist aber stark abgespeckt und nicht Host-übergreifend.


Das Produkt ESXi und vCenter hieß mal vSphere.













