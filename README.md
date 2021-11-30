# Proxmox VE by HTTP

- [Proxmox VE by HTTP](#proxmox-ve-by-http)
  - [Usage](#usage)
  - [Macroses](#macroses)
  - [Discoveries](#discoveries)
    - [PVE API Cluster](#pve-api-cluster)
      - [Cluster items](#cluster-items)
      - [Cluster triggers](#cluster-triggers)
      - [Cluster graphs](#cluster-graphs)
    - [PVE API Resources - qemu](#pve-api-resources---qemu)
      - [Qemu items](#qemu-items)
      - [Qemu triggers](#qemu-triggers)
      - [Qemu graphs](#qemu-graphs)
    - [PVE API Resources - lxc](#pve-api-resources---lxc)
      - [LXC items](#lxc-items)
      - [LXC triggers](#lxc-triggers)
      - [LXC graphs](#lxc-graphs)
    - [PVE API Resources - storage](#pve-api-resources---storage)
      - [Storage items](#storage-items)
      - [Storage triggers](#storage-triggers)
      - [Storage graphs](#storage-graphs)

Monitors PVE VMs and nodes by PVE HTTP API

Tested on Zabbix 5.4.7 and PVE 7.1

## Usage

- Go to PVE WEB UI -> Datacenter -> Api Tokens
- Click "Add"
- Select User (usualy - `root@pam`)
- Enter Token ID (for example - `monitoring`)
- **Uncheck "Privelege separation"**
- Click "Add"

You will see "Token ID" (example - `root@pam!monitoring`) and "Secret" (some UUID). Save "Secret" value, because you can't display it again!

- Import template into Zabbix
- Attach template to any server you want, and fill macroses (described below)

## Macroses

| Macro              | Description        | Example                                                    |
| ------------------ | ------------------ | ---------------------------------------------------------- |
| `{$PVE_API_HOST}`  | PVE web UI address | `pve.example.org`                                          |
| `{$PVE_API_PORT}`  | PVE web UI Port    | `8006`                                                     |
| `{$PVE_API_TOKEN}` | PVE API Token      | `root@pam!monitoring=69ab8098-f2aa-455c-add3-e387aef0a47e` |

## Discoveries

### PVE API Cluster

Gets `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/cluster/status/`, then creates items, triggers and graphs

#### Cluster items

| Item                              | Description                                                                                            | Example                      |
| --------------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------- |
| PVE Node {#NODE} status           | JSON Response for `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/nodes/{#NODE}/status/` HTTP Query |                              |
| PVE Node {#NODE} memory free      | Free RAM in bytes                                                                                      | 30.24 Gb                     |
| PVE Node {#NODE} memory size      | Total RAM size in bytes                                                                                | 64 Gb                        |
| PVE Node {#NODE} memory used      | Used RAM in bytes                                                                                      | 33.76 Gb                     |
| PVE Node {#NODE} memory used in % | Used RAM in %                                                                                          | 15 %                         |
| PVE Node {#NODE} pveversion       | PVE Version on node                                                                                    | `pve-manager/7.1-5/6fe299a0` |
| PVE Node {#NODE} rootfs available | Available size in `/` in bytes                                                                         | 13.46 Gb                     |
| PVE Node {#NODE} rootfs free      | Free size in `/` in bytes                                                                              | 14.55 Gb                     |
| PVE Node {#NODE} rootfs size      | Total size in `/` in bytes                                                                             | 20.99 Gb                     |
| PVE Node {#NODE} rootfs used      | Used size in `/` in bytes                                                                              | 6.43 Gb                      |
| PVE Node {#NODE} rootfs used in % | Used size in `/` in %                                                                                  | 80 %                         |
| PVE Node {#NODE} swap free        | Free SWAP size in bytes                                                                                | 462.49 Mb                    |
| PVE Node {#NODE} swap size        | Total SWAP size in bytes                                                                               | 2.15 Gb                      |
| PVE Node {#NODE} swap used        | Used SWAP size in bytes                                                                                | 1.68 Gb                      |
| PVE Node {#NODE} swap used in %   | Used SWAP size in %                                                                                    | 80 %                         |
| PVE Node {#NODE} uptime           | Node uptime in seconds                                                                                 | 12d 13h 30m                  |
| PVE Node {#NODE} CPU usage in %   | Node CPU usage in %                                                                                    | 15.252 %                     |

#### Cluster triggers

| Trigger                                 | Severity    | Description                                       |
| --------------------------------------- | ----------- | ------------------------------------------------- |
| PVE Node {#NODE} pve version changed    | Information | pveversion was changed since last check           |
| PVE Node {#NODE} uptime lesser than 10m | Warning     | Node has been restarted and node uptime is < 600s |
| PVE Node {#NODE} swap usage > 90%       | High        | Node SWAP usage > 90%                             |
| PVE Node {#NODE} swap usage > 80%       | Warning     | Node SWAP usage > 80%                             |
| PVE Node {#NODE} memory usage > 90%     | High        | Node RAM usage > 90%                              |
| PVE Node {#NODE} memory usage > 80%     | Warning     | Node RAM usage > 80%                              |
| PVE Node {#NODE} rootfs usage > 90%     | High        | Node rootfs usage > 90%                           |
| PVE Node {#NODE} rootfs usage > 80%     | Warning     | Node rootfs usage > 80%                           |
| PVE Node {#NODE} rootfs usage > 80%     | Warning     | Node rootfs usage > 80%                           |
| PVE Node {#NODE} CPU usage > 80%        | High        | Node CPU usage > 80%                              |

#### Cluster graphs

| Graph                   | Description                              |
| ----------------------- | ---------------------------------------- |
| PVE Node {#NODE} memory | Shows free and total RAM of node         |
| PVE Node {#NODE} rootfs | Shows free and total size in `/` of node |
| PVE Node {#NODE} swap   | Shows free and total SWAP of node        |
| PVE Node {#NODE} CPU    | CPU usage of node                        |

### PVE API Resources - qemu

Gets `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/cluster/resources/` and creates items, triggers and graphs

#### Qemu items

| Item                               | Description                                                                                                                | Example     |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ----------- |
| PVE Vm {#VMID} - {#NAME}           | JSON Response for `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/nodes/{#NODE}/qemu/{#VMID}/status/current` HTTP Query |             |
| PVE Vm {#VMID} - {#NAME} cpu usage | VM CPU Usage in %                                                                                                          | 4.427 %     |
| PVE Vm {#VMID} - {#NAME} status    | String, current VM state                                                                                                   | running     |
| PVE Vm {#VMID} - {#NAME} uptime    | VM uptime in seconds                                                                                                       | 12d 13h 38m |

#### Qemu triggers

| Trigger                                 | Severity | Description               |
| --------------------------------------- | -------- | ------------------------- |
| PVE Vm {#VMID} - {#NAME} high CPU usage | Warning  | VM CPU usage > 80%        |
| PVE Vm {#VMID} - {#NAME} is not running | Warning  | VM state is not "running" |
| PVE Vm {#VMID} - {#NAME} uptime < 10m   | Warning  | VM uptime is < 600s       |

#### Qemu graphs

| Graph                              | Description     |
| ---------------------------------- | --------------- |
| PVE Vm {#VMID} - {#NAME} CPU usage | CPU usage of VM |

### PVE API Resources - lxc

Gets `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/cluster/resources/` and creates items, triggers and graphs

#### LXC items

| Item                                       | Description                                                                                                               | Example     |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | ----------- |
| PVE Ct {#VMID} - {#NAME}                   | JSON Response for `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/nodes/{#NODE}/lxc/{#VMID}/status/current` HTTP Query |             |
| PVE Ct {#VMID} - {#NAME} cpu usage         | Container CPU Usage in %                                                                                                  | 4.427 %     |
| PVE Ct {#VMID} - {#NAME} status            | String, current container state                                                                                           | running     |
| PVE Ct {#VMID} - {#NAME} uptime            | Container uptime in seconds                                                                                               | 12d 13h 38m |
| PVE Ct {#VMID} - {#NAME} disk size         | Container disk size in bytes                                                                                              | 8 Gb        |
| PVE Ct {#VMID} - {#NAME} disk usage        | Container disk usage in bytes                                                                                             | 2 Gb        |
| PVE Ct {#VMID} - {#NAME} disk usage in %   | Container disk usage in %                                                                                                 | 20 %        |
| PVE Ct {#VMID} - {#NAME} swap size         | Container SWAP size in bytes                                                                                              | 2 Gb        |
| PVE Ct {#VMID} - {#NAME} swap usage        | Container SWAP usage in bytes                                                                                             | 1 Gb        |
| PVE Ct {#VMID} - {#NAME} swap usage in %   | Container SWAP usage in %                                                                                                 | 50 %        |
| PVE Ct {#VMID} - {#NAME} memory size       | Container RAM size in bytes                                                                                               | 1 Gb        |
| PVE Ct {#VMID} - {#NAME} memory usage      | Container RAM usage in bytes                                                                                              | 512 Mb      |
| PVE Ct {#VMID} - {#NAME} memory usage in % | Container RAM usage in %                                                                                                  | 50 %        |

#### LXC triggers

| Trigger                                   | Severity | Description                      |
| ----------------------------------------- | -------- | -------------------------------- |
| PVE Ct {#VMID} - {#NAME} high CPU usage   | Warning  | Container CPU usage > 80%        |
| PVE Ct {#VMID} - {#NAME} is not running   | Warning  | Container state is not "running" |
| PVE Ct {#VMID} - {#NAME} uptime < 10m     | Warning  | Container uptime is < 600s       |
| PVE Ct {#VMID} - {#NAME} RAM usage > 80%  | Warning  | Container RAM usage > 80%        |
| PVE Ct {#VMID} - {#NAME} RAM usage > 90%  | High     | Container RAM usage > 90%        |
| PVE Ct {#VMID} - {#NAME} SWAP usage > 80% | Warning  | Container SWAP usage > 80%       |
| PVE Ct {#VMID} - {#NAME} SWAP usage > 90% | High     | Container SWAP usage > 90%       |
| PVE Ct {#VMID} - {#NAME} disk usage > 80% | Warning  | Container disk usage > 80%       |
| PVE Ct {#VMID} - {#NAME} disk usage > 90% | High     | Container disk usage > 90%       |

#### LXC graphs

| Graph                               | Description             |
| ----------------------------------- | ----------------------- |
| PVE Ct {#VMID} - {#NAME} CPU usage  | CPU usage of Container  |
| PVE Ct {#VMID} - {#NAME} RAM usage  | RAM usage of Container  |
| PVE Ct {#VMID} - {#NAME} disk usage | disk usage of Container |
| PVE Ct {#VMID} - {#NAME} SWAP usage | SWAP usage of Container |

### PVE API Resources - storage

Gets `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/cluster/resources/` and creates items, triggers and graphs

#### Storage items

| Item                                    | Description                                                                                                           | Example |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------- |
| PVE Storage {#NODE} - {#NAME}           | JSON Response for `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/nodes/{#NODE}/storage/{#NAME}/status` HTTP Query |         |
| PVE Storage {#NODE} - {#NAME} active    | Storage is active if value is 1                                                                                       | 1       |
| PVE Storage {#NODE} - {#NAME} enabled   | Storage is enabled if value is 1                                                                                      | 1       |
| PVE Storage {#NODE} - {#NAME} available | Available size in storage in bytes                                                                                    | 10 Gb   |
| PVE Storage {#NODE} - {#NAME} size      | Total size of storage in bytes                                                                                        | 100 Gb  |
| PVE Storage {#NODE} - {#NAME} used      | Used storage size in bytes                                                                                            | 90 Gb   |
| PVE Storage {#NODE} - {#NAME} used in % | Used storage size in %                                                                                                | 90 %    |

#### Storage triggers

| Trigger                                   | Severity | Description            |
| ----------------------------------------- | -------- | ---------------------- |
| PVE Storage {#NODE} - {#NAME} unavailable | High     | Storage is unavailable |
| PVE Storage {#NODE} - {#NAME} disabled    | Warning  | Storage is disabled    |
| PVE Storage {#NODE} - {#NAME} used > 80 % | Warning  | Storage usage > 80%    |
| PVE Storage {#NODE} - {#NAME} used > 90 % | High     | Storage usage > 90%    |

#### Storage graphs

| Graph                               | Description   |
| ----------------------------------- | ------------- |
| PVE Storage {#NODE} - {#NAME} usage | Storage usage |
