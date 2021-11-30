# Proxmox VE by HTTP

Monitors PVE VMs and nodes by PVE HTTP API

Tested on Zabbix 5.4.7

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

|Macro|Description|Example|
|-----|-----------|-------|
|`{$PVE_API_HOST}`|PVE web UI address|`pve.example.org`|
|`{$PVE_API_PORT}`|PVE web UI Port|`8006`|
|`{$PVE_API_TOKEN}`|PVE API Token|`root@pam!monitoring=69ab8098-f2aa-455c-add3-e387aef0a47e`|

## Discoveries

### PVE API Cluster

Gets `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/cluster/status/`, then creates items, triggers and graphs

#### Cluster items

|Item|Description|Example|
|----|-----------|-------|
|PVE Node {#NODE} status|JSON Response for `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/nodes/{#NODE}/status/` HTTP Query||
|PVE Node {#NODE} memory free|Free RAM in bytes|30.24 Gb|
|PVE Node {#NODE} memory size|Total RAM size in bytes|64 Gb|
|PVE Node {#NODE} memory used|Used RAM in bytes|33.76 Gb|
|PVE Node {#NODE} pveversion|PVE Version on node|`pve-manager/7.1-5/6fe299a0`|
|PVE Node {#NODE} rootfs available|Available size in `/` in bytes|13.46 Gb|
|PVE Node {#NODE} rootfs free|Free size in `/` in bytes|14.55 Gb|
|PVE Node {#NODE} rootfs size|Total size in `/` in bytes|20.99 Gb|
|PVE Node {#NODE} rootfs used|Used size in `/` in bytes|6.43 Gb|
|PVE Node {#NODE} swap free|Free SWAP size in bytes|462.49 Mb|
|PVE Node {#NODE} swap size|Total SWAP size in bytes|2.15 Gb|
|PVE Node {#NODE} swap used|Used SWAP size in bytes|1.68 Gb|
|PVE Node {#NODE} uptime|Node uptime in seconds|12d 13h 30m|

#### Cluster triggers

|Trigger|Severity|Description|
|-------|--------|-----------|
|PVE Node {#NODE} pve version changed|Information|pveversion was changed since last check|
|PVE Node {#NODE} uptime lesser than 10m|Warning|Node has been restarted and node uptime is < 600s|

#### Cluster graphs

|Graph|Description|
|-----|-----------|
|PVE Node {#NODE} memory|Shows free and total RAM of node|
|PVE Node {#NODE} rootfs|Shows free and total size in `/` of node|
|PVE Node {#NODE} swap|Shows free and total SWAP of node|

### PVE API Resources - qemu

Gets `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/cluster/resources/` and creates items, triggers and graphs

#### Qemu items

|Item|Description|Example|
|----|-----------|-------|
|PVE Vm {#VMID} - {#NAME}|JSON Response for `https://{$PVE_API_HOST}:{$PVE_API_PORT}/api2/json/nodes/{#NODE}/qemu/{#VMID}/status/current` HTTP Query||
|PVE Vm {#VMID} - {#NAME} cpu usage|VM CPU Usage in %|4.427 %|
|PVE Vm {#VMID} - {#NAME} status|String, current VM state|running|
|PVE Vm {#VMID} - {#NAME} uptime|VM uptime in seconds|12d 13h 38m|

#### Qemu triggers

|Trigger|Severity|Description|
|-------|--------|-----------|
|PVE Vm {#VMID} - {#NAME} high CPU usage|Warning|VM CPU usage > 80%|
|PVE Vm {#VMID} - {#NAME} is not running|Warning|VM state is not "running"|
|PVE Vm {#VMID} - {#NAME} uptime < 10m|Warning|VM uptime is < 600s|

#### Qemu graphs

|Graph|Description|
|-----|-----------|
|PVE Vm {#VMID} - {#NAME} CPU usage|CPU usage of VM|
