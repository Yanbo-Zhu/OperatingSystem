
in linux, systemd  übernimmt jetzt auch logging

`systemd` is a system and service manager for Linux operating systems. It provides an efficient way to manage services, processes, and dependencies during the system's boot and runtime. It has largely replaced traditional init systems like SysVinit and Upstart.


# 1 **Key Features of `systemd`:**
1. **Unit Files:**
    - `systemd` manages services, sockets, devices, timers, and more using **unit files**. These files define the behavior and configuration for each managed resource.
    - Common types of units:
        - **Service units (`.service`)**: Define how services start, stop, and restart.
        - **Timer units (`.timer`)**: Schedule tasks, replacing tools like `cron`.
        - **Mount units (`.mount`)**: Handle filesystem mounts.
2. **Parallel Initialization:**
    - `systemd` starts services in parallel, based on their dependencies, for faster boot times.
3. **Dependency Management:**
    - Services and processes are started or stopped in a specific order, as defined by their dependencies.
4. **Journal:**
    - Provides **`journald`**, a logging system that centralizes and manages log files.
5. **Dynamic Control:**
    - Tools like `systemctl` allow you to manage services dynamically.

---

Common Commands for systemd:
**Service Management:**
`sudo systemctl start/stop/restart/enable/disable <service>.service`

Status and Logs:
Check the status of a service:  `sudo systemctl status <service>.service`
View logs `sudo journalctl -u <service>.service`

System Operations:  
Reboot the system:
`sudo systemctl reboot`

Shut down the system
`sudo systemctl poweroff`




# 2 Daemons

> systemd ist ein Daemons 

• Traditioneller Name für Hintergrundaktivitäten und Dienste in Linux
    • Wortspiel mit demon
• Programmnamen enden mit d
    • crond: regelmäßige Ausführung von Programmen
    • Ipd: Printer-Daemon
    • sshd: remote login
• Starten i.d.R bei Systemstart


# 3 Linux-Boot-Prozess auf PCs
• Power On: BIOS, UEFI (Firmware)
    • Bootkonfiguration: Master Boot Record Oder UEFl-Partition
• BIOS lädt Bootloader des Betriebssystems
    • Linux: i.d.R. grub
• OS-Bootloader lädt Kernel
    • Linux: vmlinuz, bzw. entsprechend grub-Konfiguration
• Linux-Kern startet Initialprozess: /sbin/init
    • Implementiert als Teil von systemd


## 3.1 Implementierung von /sbin/init
• Traditionell: /sbin/init liest /etc/inittab
    • Konzept: Runlevel (single user, multi-user, network, ...) 
    • Shell-Skripts zum Start und Stop von Diensten
• systemd
• Upstart



# 4 systemd
• "Ganzheitliche" Konfiguration des Systems
    • Boot-Prozess
    • Daemons
    • Dateisysteme
    • Logging
• Abhängigkeiten zwischen Diensten
    • ermöglicht paralleles Starten von Diensten während des Boot-Prozesses


## 4.1 systemd 文件位置 

1 
/lib/systemd/system/ 

![](images/Pasted%20image%2020250120205501.png)



2 
/etc/systemd/system/ 

![](images/Pasted%20image%2020250120205049.png)

## 4.2 systemd-Konzepte

• Units: Systemkomponenten, die von systemd verwaltet werden
    • .service: Dienstdateien
    • .mount: zu mountende Dateisysteme (ersetzt/ergänzt /etc/fstab)
    • .target: Zielkonfiguration des Systems (entspricht Runleveln). the target under which this service is enabled.
• Managementtool systemctl(8)
    • systemctl list-units
    • systemctl list-dependencies
    • `systemctl show <unit>`
        • z.B. systemctl show multi-user.target

system status ssh 


### 4.2.1 systemctl list-units

alle units, das systemctl kennt 

![](images/Pasted%20image%2020250120201854.png)

![](images/Pasted%20image%2020250120202004.png)


![](images/Pasted%20image%2020250120202119.png)


## 4.3 systemctl show xx


### 4.3.1 systemctl show ssh.service 

konfigurations file von service 

![](images/Pasted%20image%2020250120202135.png)


![](images/Pasted%20image%2020250120202144.png)


![](images/Pasted%20image%2020250120202233.png)

![](images/Pasted%20image%2020250120202302.png)


![](images/Pasted%20image%2020250120202322.png)


### 4.3.2 systemctl show multi-user.target

Zielkonfiguration von multiuser 

![](images/Pasted%20image%2020250120202354.png)

![](images/Pasted%20image%2020250120202432.png)

## 4.4 systemctl status 


systemctl status ssh 

![](images/Pasted%20image%2020250120202646.png)

## 4.5 systemctl list-dependecies 

service 和 target 之间的依存关系 

![](images/Pasted%20image%2020250120203003.png)


## 4.6 Unit-Definitionen

• Ini-Style Schlüssel-Wert-Paare


A **unit file** in systemd describes how a service, socket, timer, or other resource should be managed. These unit files are written in **INI-style format**, which uses key-value pairs. 
Unit files like this allow system administrators to easily manage and configure services in a consistent way using systemd.


### 4.6.1 Beispiel: ssh.service 

• z.B. /lib/systemd/system/ssh.service (gekürzt)
```
[Unit]
Description=OpenBSD Secure Shell server
After-network. target auditd.service
ConditionPathExists= ! / etc/ssh/sshd_not_to_be_run
[Service]
EnvironmentFi1e=-/etc/defau1t/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
Restart=on-failure
RestartPreventExitStatus=255
[Install]
WantedBy=mu1ti-user . target
Alias=sshd . service
```

Unit files like this allow system administrators to easily manage and configure services in a consistent way using systemd.

This unit file defines the SSH server's behavior, including:

1. When it starts and under what conditions.
2. Pre-start checks and the main execution command.
3. Restart policies.
4. Installation targets for enabling the service.


**[Unit]** – General configuration
- **`Description`**: A human-readable description of the service.  
    - Example: `Description=OpenBSD Secure Shell server`
- **`After`**: Specifies that this service should start after certain targets or services are active.
    - `After=network.target auditd.service`
    - This ensures that the SSH server starts only after the network and audit daemon services are up.
- **`ConditionPathExists`**: A condition that must be met for the unit to start.
    - `ConditionPathExists=!/etc/ssh/sshd_not_to_be_run`
    -  The service will not start if the specified path exists.

**[Service]** – Configuration specific to the service
- **`EnvironmentFile`**: Specifies a file containing environment variables. The `-` prefix means it's optional.
    - `EnvironmentFile=-/etc/default/ssh`
- **`ExecStartPre`**: Commands to run before starting the service.
    - `ExecStartPre=/usr/sbin/sshd -t`
    - Runs a configuration test for SSH before starting.
- **`ExecStart`**: The main command to start the service.
    - `ExecStart=/usr/sbin/sshd -D $SSHD_OPTS`
    - Starts the SSH daemon with specified options.
- **`Restart`**: Specifies the restart behavior for the service.
    - `Restart=on-failure`
- **`RestartPreventExitStatus`**: Prevents automatic restarts for specific exit statuses.
    - `RestartPreventExitStatus=255`
    - If the service exits with status `255`, it will not restart.

**[Install]** – Installation settings
- **`WantedBy`**: Specifies the target under which this service is enabled.
    - `WantedBy=multi-user.target`
    - This makes the service part of the multi-user runlevel.
- **`Alias`**: Provides an alternative name for the service.
    - `Alias=sshd.service`


## 4.7 Abhängigkeiten

• Units hängen voneinander ab
    • viele Dienste setzen voraus, dass alle Dateisysteme gemountet sind
    • Netzwerkdienste verlangen, dass die Netzwerkschnittstellen konfiguriert sind
    • Targets fassen "gewünschte" Dienste zusammen
• A requires B: A Wird erst gestartet, wenn B erfolgreich gestartet wurd
• A wants B: B Wird gestartet. Falls das scheitert, Wird A trotzdem gestartet
• A after B: Der Start von B Wird initiiert, danach der Start von A



## 4.8 Journald

• Logging-Dienst von systemd
• Binäre Speicherung von Lognachrichten
    • für effiziente Suche
• Abfragen über journalctl(l)


# 5 Q and A

## 5.1 sind dann alle prozesse ein fork() von init? bzw forks von unterprozessen von init...

answer:
systemd is wurzelprocess von alle prozess 


通过执行下面的命令,  我们可以看出来 systemd is wurzelprocess von alle prozess 
pstree -a   systemd

```
pstree -a  
systemd  
├─ModemManager  
│ └─3*[{ModemManager}]  
├─NetworkManager --no-daemon  
│ └─3*[{NetworkManager}]  
├─accounts-daemon  
│ └─3*[{accounts-daemon}]  
├─agent  
│ └─3*[{agent}]  
├─agetty -o -p -- \\u --noclear - linux  
├─anydesk --service  
│ └─5*[{anydesk}]  
├─avahi-daemon  
│ └─avahi-daemon  
├─ayatana-indicat  
│ └─4*[{ayatana-indicat}]  
├─ayatana-indicat  
│ └─4*[{ayatana-indicat}]  
├─ayatana-indicat  
│ └─4*[{ayatana-indicat}]  
├─ayatana-indicat  
│ └─4*[{ayatana-indicat}]  
├─ayatana-indicat  
│ └─3*[{ayatana-indicat}]  
├─blueman-tray /usr/bin/blueman-tray  
│ └─4*[{blueman-tray}]  
├─bluetoothd  
├─colord  
│ └─3*[{colord}]  
├─cron -f -P  
├─cups-browsed  
│ └─3*[{cups-browsed}]  
├─cupsd -l  
│ ├─dbus dbus://  
│ └─dbus dbus://  
├─dbus-daemon --system --address=systemd: --nofork --nopidfile...  
├─geoclue  
│ └─3*[{geoclue}]  
├─group-admin-dae  
│ └─3*[{group-admin-dae}]  
├─kerneloops --test  
├─kerneloops  
├─lightdm  
│ ├─Xorg -core :0 -seat seat0 -auth /var/run/lightdm/root/:0-nolisten  
│ │ └─22*[{Xorg}]  
│ ├─lightdm --session-child 14 21  
│ │ ├─mate-session  
│ │ │ ├─agent  
│ │ │ │ └─3*[{agent}]  
│ │ │ ├─ayatana-indicat  
│ │ │ │ └─4*[{ayatana-indicat}]  
│ │ │ ├─ayatana-indicat  
│ │ │ │ └─4*[{ayatana-indicat}]  
│ │ │ ├─ayatana-indicat  
│ │ │ │ └─6*[{ayatana-indicat}]  
│ │ │ ├─ayatana-indicat  
│ │ │ │ └─4*[{ayatana-indicat}]  
│ │ │ ├─ayatana-indicat  
│ │ │ │ └─3*[{ayatana-indicat}]  
│ │ │ ├─ayatana-indicat  
│ │ │ │ └─3*[{ayatana-indicat}]  
│ │ │ ├─ayatana-indicat  
│ │ │ │ └─4*[{ayatana-indicat}]  
│ │ │ ├─ayatana-indicat  
│ │ │ │ └─4*[{ayatana-indicat}]  
│ │ │ ├─blueman-applet /usr/bin/blueman-applet  
│ │ │ │ └─4*[{blueman-applet}]  
│ │ │ ├─caja  
│ │ │ │ └─5*[{caja}]  
│ │ │ ├─diodon  
│ │ │ │ └─20*[{diodon}]  
│ │ │ ├─evolution-alarm  
│ │ │ │ └─7*[{evolution-alarm}]  
│ │ │ ├─keepassxc  
│ │ │ │ └─15*[{keepassxc}]  
│ │ │ ├─marco  
│ │ │ │ └─5*[{marco}]
```


## 5.2 define a service immer neustart nach reboot 


Die Information, ob ein Dienst beim nächsten Systemstart gestartet werden soll, wird in Symlinks gespeichert, die sich im Verzeichnis ==/etc/systemd/system/ ==befinden.  
Details:  
  
```
Pfad:  
/etc/systemd/system/  
(enthält Unterordner wie multi-user.target.wants/, graphical.target.wants/ usw.)  
  
Mechanismus:  
Beim Befehl systemctl enable <dienst> wird ein symbolischer Link zur Service-Datei (.service) des Dienstes in das Zielverzeichnis erstellt. Zum Beispiel:  
  
/etc/systemd/system/multi-user.target.wants/<dienst>.service
```


![](images/Pasted%20image%2020250120205627.png)

---

wo ist gespeichert, ob ein dienst beim nächsten Systemstart wieder gestartet werden soll.... also das, was man mit systemctl enable einstellt

![](images/Pasted%20image%2020250120205144.png)

