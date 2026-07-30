# Cockpit Proxmox Overview

A lightweight Cockpit plugin that displays an overview of Proxmox VE virtual
machines and LXC containers. The interface is implemented as a single HTML page
and uses Bootstrap from jsDelivr.

## Features

- Cluster summary with QEMU VM and LXC totals
- Running, stopped, and paused machine counts
- Machines grouped by Proxmox node
- CPU, memory, virtual disk, and cluster storage usage
- Highest CPU consumers
- VMID, name, type, node, status, CPU, RAM, disk, network traffic, uptime,
  tags, description, and template information
- Search and filters by status and machine type
- Start, stop, and reboot actions for QEMU VMs and LXC containers

The main machine data is read with:

```shell
pvesh get /cluster/resources --type vm --output-format json
```

Additional `pvesh` requests are used to retrieve storage and machine
configuration details.

## Requirements

- Proxmox VE with the `pvesh` command available
- Cockpit 215 or newer
- A Cockpit user authorized to run the required `pvesh` commands as root
- Browser access to `https://cdn.jsdelivr.net` for Bootstrap

This plugin is intended to run directly on a Proxmox VE node. Installing it on
another machine will not provide access to the local Proxmox cluster API.

## Installation

### System-wide installation

Run these commands on the Proxmox VE node:

```shell
cd /usr/share/cockpit
sudo git clone https://github.com/fbsis/cockpit-proxmox.git proxmox-overview
sudo systemctl restart cockpit.socket
```

Open Cockpit in your browser and select **Proxmox Overview** from the
navigation menu:

```text
https://PROXMOX_HOST:9090
```

### Per-user development installation

For development, Cockpit can load the repository through a symbolic link:

```shell
mkdir -p ~/.local/share/cockpit
ln -s "$(pwd)" ~/.local/share/cockpit/proxmox-overview
```

Check whether Cockpit detects the package:

```shell
cockpit-bridge --packages
```

After editing `index.html`, refresh the Cockpit page. Remove the development
installation with:

```shell
rm ~/.local/share/cockpit/proxmox-overview
```

## Updating

Update the repository cloned inside the Cockpit packages directory:

```shell
cd /usr/share/cockpit/proxmox-overview
sudo git pull
sudo systemctl restart cockpit.socket
```

## Permissions

The page requests privileged command execution through Cockpit before invoking
`pvesh`. The signed-in account must be allowed to elevate privileges on the
Proxmox host. Proxmox API permissions must also allow the requested read and
power-management operations.

Power actions are sent to these Proxmox API paths:

```text
/nodes/{node}/qemu/{vmid}/status/{start|stop|reboot}
/nodes/{node}/lxc/{vmid}/status/{start|stop|reboot}
```

The **Stop** action performs an immediate Proxmox `stop` operation. It is not a
guest operating system shutdown.

## Project structure

```text
.
├── index.html
├── manifest.json
└── README.md
```

`manifest.json` is required by Cockpit even though the complete user interface,
styles, and application logic live in `index.html`.

## Security note

Start, stop, and reboot actions require confirmation in the interface. Cockpit
still enforces host authentication and privilege elevation. Only grant access
to trusted administrators.

## License

No license has been specified yet.
