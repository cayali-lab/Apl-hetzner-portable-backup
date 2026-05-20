# Hetzner Portable Backup Project

## Project Goal

The goal of this project is to create a portable backup solution for a Hetzner Cloud Server and restore it outside Hetzner using VirtualBox.

---

## Project Architecture

```text
Hetzner Cloud Server
        ↓
Raw Disk Imaging (dd)
        ↓
Compression (gzip)
        ↓
SSH Streaming Transfer
        ↓
Portable Backup Image
        ↓
QEMU Image Conversion
        ↓
VirtualBox Restoration
        ↓
WordPress Validation
```

---

## Technologies Used

- Ubuntu 24.04
- SSH
- dd
- gzip
- SCP
- qemu-img
- VirtualBox
- GitHub

---

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

```text
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
```

---

## Current Progress

### Completed

- Project repository initialized
- README documentation created
- GitHub repository connected
- SSH authentication completed
- Hetzner infrastructure discovery completed
- Apache2 and MySQL validation completed
- WordPress environment verified
- Portable compressed backup image created
- Local backup storage completed

### In Progress

- RAW image extraction
- VDI conversion testing
- VirtualBox restoration

### Planned

- Virtual machine boot verification
- WordPress functionality validation
- Final restoration testing
- Project presentation

---

## Infrastructure Discovery Results

During the infrastructure discovery phase, we identified that the server was running a classic LAMP stack:

- Ubuntu 24.04
- Apache2
- MySQL Community Server
- WordPress

The WordPress installation was located in:

```bash
/var/www/html
```

The primary operating system partition was mounted on:

```bash
/dev/sda1
```

We also identified an additional mounted storage volume:

```bash
/dev/sdb → /mnt
```

This discovery phase was essential for planning the portable backup and restoration strategy.

---

## Service Validation

We verified that the following infrastructure services were operational:

- Apache2 web server
- MySQL database server
- WordPress installation

During testing, Apache2 was initially inactive and had to be manually restarted.

Before creating the disk image, both Apache2 and MySQL services were temporarily stopped in order to reduce the risk of filesystem or database inconsistencies during the cloning process.

After the backup operation completed successfully, both services were restored and verified again.

---

## Portable Image Creation

We successfully created a portable compressed raw disk image of the Hetzner cloud server using:

- `dd`
- `gzip`
- `SSH streaming`

The image was streamed directly from the Hetzner server to a local Linux environment in order to avoid server-side storage limitations.

### Backup Workflow

```text
Hetzner Server
↓
Raw disk clone using dd
↓
Compression using gzip
↓
SSH stream transfer
↓
Local backup image
```

### Command Used

```bash
ssh -i ~/.ssh/private_openssh root@46.62.248.133 "dd if=/dev/sda bs=64M status=progress | gzip -" > server-backup.img.gz
```

### Result

- Raw disk size: ~41 GB
- Compressed portable image: ~35 GB

The backup image was successfully downloaded outside Hetzner and stored locally for future restoration testing in:

- VirtualBox
- VMware Workstation Pro

---

## Current Project Status

### Completed

- SSH authentication
- Infrastructure discovery
- Service validation
- Portable image creation
- Local backup storage

### Next Steps

- Extract raw image
- Convert RAW image to VDI format
- Restore image in VirtualBox
- Verify Ubuntu boot process
- Verify Apache, MySQL and WordPress functionality

---

## Backup Result

The portable backup image was successfully created and downloaded locally.

Generated backup file:

```text
server-backup.img.gz
```

Backup details:

- Backup type: Raw disk image
- Compression: gzip
- Transfer method: SSH streaming
- Source disk: /dev/sda
- Approximate raw size: 41 GB
- Approximate compressed size: 35 GB

Current status:

- Portable image successfully created
- Backup stored locally
- Ready for extraction and VirtualBox restoration testing

Due to local storage limitations, image extraction and VDI conversion will continue during the next project phase.