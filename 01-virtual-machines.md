# Setting Up the Virtual Machines

## Overview
This lab uses VirtualBox to run isolated virtual machines simulating a
small corporate network: two "victim" endpoints (Windows and Linux) that
get monitored and attacked, a SIEM server watching over them, and a
Kali Linux attacker machine.

## Tools Used
- Oracle VirtualBox (free hypervisor)
- Ubuntu 26.04 LTS (Linux victim)
- Windows 11 Enterprise Evaluation (Windows victim, free 90-day license)
- Kali Linux 2026.2 (attacker machine)

## Ubuntu-Victim Setup
1. Downloaded Ubuntu Desktop ISO from ubuntu.com
2. Created a new VM in VirtualBox (2GB RAM, 25GB disk)
3. Installed via VirtualBox's unattended installation feature
4. Installed VirtualBox Guest Additions for improved performance and
   clipboard sharing

## Windows-Victim Setup
1. Downloaded Windows 11 Enterprise Evaluation ISO from Microsoft's
   Evaluation Center (free, legal 90-day trial for lab/testing use)
2. Created a new VM (4GB RAM, 60GB disk)
3. Enabled EFI and TPM 2.0 in VM settings (required for Windows 11)
4. Installed with a local account (skipped Microsoft account requirement)

## Kali-Attacker Setup
1. Downloaded the official pre-built VirtualBox image from kali.org
2. Extracted the `.7z` archive using 7-Zip
3. Imported directly into VirtualBox

## Networking
All four VMs (Wazuh, Windows-Victim, Ubuntu-Victim, Kali) are configured
on a **Bridged Adapter**, placing them on the same local network so they
can communicate with each other, simulating a real internal network that
a SOC would monitor.

Initial troubleshooting note: Windows-Victim was originally left on
VirtualBox's default **NAT** network, which isolated it from the other
VMs. Switching it to Bridged Adapter resolved connectivity for later
attack simulations.

## Skills Demonstrated
- Virtualization and hypervisor configuration
- Basic Linux and Windows OS installation and administration
- Network configuration and troubleshooting (NAT vs. Bridged networking)
- Troubleshooting VM boot issues and display driver compatibility