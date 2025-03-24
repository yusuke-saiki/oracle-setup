# oracle-setup

## Overview
This repository contains infrastructure-level setup guides and configurations for deploying Oracle 19c RAC and Data Guard on RHEL-based environments (RHEL 8/9).  
It is intended for infrastructure/database engineers who want to build highly available Oracle environments on bare metal or virtualized systems.

## Features / Highlights
- Oracle 19c RAC and Data Guard setup on RHEL 8
- Oracle 19c RAC setup on RHEL 9
- Includes pre-install scripts, OS/network configuration, and verification notes

## Target Environment
- OS: RHEL 8 / RHEL 9-based systems
- Oracle Grid Infrastructure 19.26
- Oracle Database 19.26
- Cluster: 2-node RAC + optional Data Guard standby

## Directory Structure
oracle-setup/
├── rac-dg-on-rhel8/        # Oracle RAC + Data Guard on RHEL 8
├── rac-on-rhel9/           # Oracle RAC on RHEL 9
└── common/                 # Common scripts and base configurations

## How to Use
Each subdirectory contains a step-by-step guide (setup-guide.md) and necessary configuration files/scripts.
Start with the environment that matches your target OS version.

## License
Apache 2.0

## Author
[yusuke-saiki](https://github.com/yusuke-saiki)
