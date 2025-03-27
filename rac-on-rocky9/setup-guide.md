# Oracle 19c RAC on Rocky Linux 9 (VirtualBox Environment)

## Overview

This guide documents the installation of Oracle 19c Real Application Clusters (RAC) on Rocky Linux 9.4 / 9.5 using Oracle VM VirtualBox.

It is intended for infrastructure/database engineers who want to simulate an Oracle RAC environment on personal workstations using VirtualBox and public Oracle installation media.

The environment is minimal and educational, not intended for production use.

---

## Virtual Environment Summary

### Hypervisor
- **Tool**: Oracle VM VirtualBox 7.1.6 r167084 (Qt6.5.3) + Extension Pack
- **ISO**: Rocky-9.4 / 9.5 x86_64 DVD ISO
- **Oracle Grid**: V982068-01.zip
- **Oracle RDBMS**: V982063-01.zip
- **OPatch**: p6880880_190000_Linux-x86-64.zip
- **GI RU Patch**: p37257886_190000_Linux-x86-64.zip

---

## Virtual Machines

### 1. DNS Server

| Item         | Value                        |
|--------------|------------------------------|
| Hostname     | `dns.ha.jp`                  |
| OS           | Rocky Linux 9.5              |
| Memory       | 2 GB                         |
| CPU          | 1 vCPU                       |
| Disk         | 20 GB (dynamic)              |
| NIC (eth0)   | Host-only: `10.0.10.100/24`  |
| Partitions   | EFI 600 MiB / Boot 1024 MiB / Swap 2 GB / Root rest |
| Users        | root/root                    |

---

### 2. RAC Node01

| Item         | Value                        |
|--------------|------------------------------|
| Hostname     | `node01.ha.jp`               |
| OS           | Rocky Linux 9.4              |
| Memory       | 10 GB                        |
| CPU          | 4 vCPU                       |
| Disk         | 100 GB (dynamic)             |
| NIC1 (eth0)  | Host-only: `10.0.10.101/24`  |
| NIC2 (eth1)  | Internal: `10.0.20.101/24`   |
| Partitions   | EFI 600 MiB / Boot 1024 MiB / Swap 10 GB / Root rest |
| Users        | root/root, oracle/oracle, grid/grid |

---

### 3. RAC Node02

| Item         | Value                        |
|--------------|------------------------------|
| Hostname     | `node02.ha.jp`               |
| OS           | Rocky Linux 9.4              |
| Memory       | 10 GB                        |
| CPU          | 4 vCPU                       |
| Disk         | 100 GB (dynamic)             |
| NIC1 (eth0)  | Host-only: `10.0.10.102/24`  |
| NIC2 (eth1)  | Internal: `10.0.20.102/24`   |
| Partitions   | EFI 600 MiB / Boot 1024 MiB / Swap 10 GB / Root rest |
| Users        | root/root, oracle/oracle, grid/grid |

---

## Next Steps

- Network & Hostname Configuration
- DNS Forward/Reverse Setup
- Shared Storage Preparation
- OS Kernel & Package Tuning
- Grid Infrastructure Installation
- Oracle RDBMS Installation
