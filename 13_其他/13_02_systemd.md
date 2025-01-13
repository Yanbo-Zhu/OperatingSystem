
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