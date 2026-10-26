# Proxmox VE Home Lab on VirtualBox

A step-by-step record of how I built a Proxmox VE 9.2 learning lab inside VirtualBox on a Windows PC, on an office network, including every problem I hit and how I fixed it.

## Goals

- Learn Proxmox VE hands-on without dedicated hardware
- Keep the lab isolated from the office LAN (no IP conflicts)
- Document the setup so it can be rebuilt quickly

## Lab architecture

```
Windows PC (host)
└── VirtualBox
    └── Proxmox VE 9.2 VM  (192.168.xx.xx)
        ├── LXC containers   (planned)
        └── Test VMs         (needs nested virtualization, see Known limitations)
```

| Component | Value |
|---|---|
| Host OS | Windows (11) |
| Hypervisor | VirtualBox |
| Guest | Proxmox VE 9.2 (Debian 13 "trixie" base) |
| vCPUs / RAM / Disk | 2+ / 4-8 GB / 32 GB |
| Adapter 1 | Host-only (`192.168.xx.xx/24`), used for the web UI |
| Adapter 2 | NAT, used for internet (updates, templates) |
| Proxmox IP | `192.168.xx.xx/24` (static, no gateway on `vmbr0`) |
| Web UI | `https://192.168.xx.xx:8006` |
| Login | user `root`, realm **Linux PAM** |

## 1. Prerequisites

- 16 GB RAM on the host is recommended (8 GB minimum)
- ~50 GB free disk
- Virtualization (Intel VT-x / AMD-V) enabled in BIOS/UEFI
- VirtualBox installed on Windows
- Proxmox VE ISO from <https://www.proxmox.com/en/downloads>

## 2. Create the VM

In VirtualBox, click **New**:

- Type: **Linux**, Version: **Debian (64-bit)**
- RAM: 4096 MB or more
- CPUs: 2 or more
- Disk: 32 GB+, dynamically allocated
- Attach the Proxmox ISO to the optical drive

Switch Settings to **Expert** mode and tick **System > Processor > Enable Nested VT-x/AMD-V**. Untick **Enable Audio** (not needed).

## 3. Networking (office-network friendly)

Bridged mode caused an IP conflict with the Windows PC on the office network, so this lab uses two adapters instead.

**3.1 Create the Host-only network (Windows)**
VirtualBox > **File > Tools > Network Manager > Host-only Networks**. Use the existing one or click **Create**. Note its address (usually `192.168.xx.x/24`). Leave the DHCP server off.

**3.2 Set the VM adapters** (VM powered off)

- **Adapter 1:** Host-only Adapter, pick the network above
- **Adapter 2:** Enable, NAT

## 4. Install Proxmox

1. Start the VM and choose **Install Proxmox VE (Graphical)**.
2. Accept the license and keep the default target disk.
3. Choose country, time zone, and keyboard.
4. Set the **root password** and an email (any address). The installer does **not** create a separate user; the login is `root`.
5. Set hostname (e.g. `pve.lab`) and network, then **Install** and **Reboot**.
6. After the reboot, remove the ISO (**Devices > Optical Drives > Remove disk from virtual drive**) so it doesn't boot the installer again.

## 5. Configure Proxmox networking

Log in on the VM console as `root` and edit `/etc/network/interfaces`:

```
auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.xx.xx/24
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0

auto enp0s8
iface enp0s8 inet dhcp

source /etc/network/interfaces.d/*
```

- `vmbr0` (Host-only) has a static address and **no gateway**
- `enp0s8` (the NAT adapter) gets its address and default route via DHCP
- Check the real NIC names with `ip -br link` and use those

Update `/etc/hosts`:

```
192.168.xx.xx pve.lab pve
```

Apply:

```bash
ifreload -a
ping -c 3 google.com
```

Then open `https://192.168.xx.xx:8006` from Windows, accept the certificate warning, and log in as `root` (Linux PAM).

## 6. Fix repositories and update

The enterprise repos require a paid subscription. Switch to the free ones.

In the web UI: **pve > Updates > Repositories**

1. Disable the `pve-enterprise` and `ceph-squid enterprise` entries
2. **Add > No-Subscription**
3. **Reload**

Then update from the node shell:

```bash
DEBIAN_FRONTEND=noninteractive apt full-upgrade -y
reboot
```

The "no valid subscription" popup at login is expected on the free edition.

## 7. Snapshot the VM

Proxmox's web UI can only snapshot guests inside it. Snapshot the Proxmox VM itself from VirtualBox:

1. Shut down Proxmox (**Shutdown** in the web UI or `poweroff`)
2. VirtualBox > select the VM > **Snapshots > Take**
3. Name it, e.g. `Fresh updated Proxmox 9.2`

Restore with: power off, **Snapshots**, right-click the snapshot, **Restore**.

## Troubleshooting log

| Problem | Cause | Fix |
|---|---|---|
| "No support for hardware-accelerated KVM virtualization detected" | Windows Virtualization-Based Security / Credential Guard holds the hypervisor, blocking nested virtualization | See [Known limitations](#known-limitations) |
| Installer menu appears again after install | VM still booting from the ISO | Remove the ISO from the optical drive, or put Hard Disk first in boot order |
| Can't log in / don't remember the username | The installer only asks for a root password | Username is `root`, realm Linux PAM |
| Web UI unreachable on office network | IP conflict with the Windows PC in bridged mode | Use Host-only adapter with static `192.168.56.10` |
| No internet in Proxmox | Host-only has no route out | Add Adapter 2 (NAT) and configure it with DHCP |
| `Cannot find device "nic1"` | Second NIC is named `enp0s8`, not `nic1` | Use the name from `ip -br link` |
| `auto lo` split in `/etc/network/interfaces` | New lines pasted at the top of the file | Rewrite the file cleanly (section 5) |
| `apt-listchanges ... received signal 2` during upgrade | Changelog viewer interrupted the upgrade | Run `DEBIAN_FRONTEND=noninteractive apt full-upgrade -y` |

## Known limitations

- **Nested virtualization:** on this Windows machine, VBS/Credential Guard prevents VirtualBox from exposing hardware virtualization to Proxmox. Proxmox installs and runs fine, but **full VMs inside it may not start**. **LXC containers are unaffected.**
- Possible fixes: disable Memory Integrity, Credential Guard, and Hyper-V features and set `bcdedit /set hypervisorlaunchtype off`, or use VMware Workstation, which handles nested virtualization better.
- This is a learning lab, not a production setup. Do not expose it to the internet.

## Learning roadmap

- [x] Install Proxmox VE in VirtualBox
- [x] Host-only + NAT networking
- [x] Repositories and system update
- [x] VirtualBox snapshot
- [ ] Create the first LXC container
- [ ] Upload ISOs and create a VM (needs nested virtualization)
- [ ] Proxmox snapshots and backups
- [ ] Users, groups, and permissions
- [ ] Storage types (local, LVM-thin, directory)
- [ ] Networking: bridges and VLANs
- [ ] Templates and cloning
- [ ] Firewall
- [ ] Clustering (clone the VM into a second node)

## References

- Proxmox VE documentation: <https://pve.proxmox.com/pve-docs/>
- Proxmox downloads: <https://www.proxmox.com/en/downloads>
- VirtualBox manual: <https://www.virtualbox.org/manual/>
