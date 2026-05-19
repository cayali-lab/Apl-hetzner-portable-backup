# Hetzner Portable Backup Project

## Project Goal

The goal of this project is to create a portable backup solution for a Hetzner Cloud Server and restore it outside Hetzner using VirtualBox.

## Technologies Used

- Ubuntu 24.04
- SSH
- dd
- gzip
- SCP
- qemu-img
- VirtualBox
- GitHub

## Team Members

- Ali Cay
- Johannes Noyan
- Juan Martin 

---

## Research Phase

During the research phase, we investigated different methods for creating portable backups of Hetzner Cloud Servers.

We researched:
- dd
- Clonezilla
- rsync
- fsarchiver
- qemu-img

We compared block-level backup methods and file-level backup methods in order to determine the best solution for portability and restoration outside Hetzner.

---

## Planned Workflow

Hetzner Server
↓
Portable Image Creation
↓
Compressed Backup File
↓
Download via SCP
↓
Convert Image using qemu-img
↓
Restore in VirtualBox
↓
Verify WordPress Functionality