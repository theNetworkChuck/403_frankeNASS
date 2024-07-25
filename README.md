

# Building a FrankeNASS with Ceph

In this tutorial, we will build a NAS using a variety of old hardware and the open-source storage solution Ceph. Follow along with the video and the steps below.

## Video Walkthrough

[![Watch the video](https://img.youtube.com/vi/jJrnJ9rj6fs/maxresdefault.jpg)](https://youtu.be/jJrnJ9rj6fs)

## Step-by-Step Guide

### 1. Introduction

> [00:00 - 00:02] I'm building a NAS with all of this: A bunch of junk and spare parts I found in my storage closet. A couple old laptops, a Zima board, a Ugreen NAS, a TerraMaster NAS, an old Dell server, a pile of random servers and hard drives, and a few Raspberry Pi's.

### 2. What is Ceph?

> [00:21 - 00:24] With an open-source magic technology called Ceph. It's software-defined storage and it's crazy.

### 3. Setting Up Ceph

#### 3.1. Hardware Requirements

- **CPU**: ARM or x86
- **RAM**: Minimum 4GB
- **Storage**: External or internal hard drives/SSDs (USB flash drives are not recommended)
- **Network Switch**: At least 1 Gigabit for lab, higher for production

#### 3.2. Software Requirements

- **OS**: Ubuntu 22.04 (Use 20.04 for Raspberry Pi with Reef 18.2.2)
- **Additional Tools**: SSH, Docker, LVM2

### 4. Preparing Your Hosts

#### 4.1. Install Ubuntu

1. Install Ubuntu 22.04 on all hosts.
2. Install SSH server:
   ```sh
   sudo apt install openssh-server
   ```

3. Set static IP address in `netplan` configuration:
   ```sh
   cd /etc/netplan
   sudo nano your-config.yaml
   ```

4. Update and upgrade system:
   ```sh
   sudo apt update
   sudo apt upgrade -y
   ```

5. Set a password for the root user:
   ```sh
   sudo passwd root
   ```

6. Enable SSH login for the root user:
   ```sh
   sudo nano /etc/ssh/sshd_config
   # Un-comment and set "PermitRootLogin yes"
   sudo systemctl restart ssh
   ```

#### 4.2. Prepare Storage

1. Identify storage devices:
   ```sh
   lsblk
   ```

2. Wipe storage devices:
   ```sh
   sudo sgdisk --zap-all /dev/sdX
   sudo wipefs --all /dev/sdX
   ```

#### 4.3. Install Required Software

1. Install Docker:
   ```sh
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh

   ## refer to this link: https://docs.docker.com/engine/install/ubuntu/
   ```

2. Install LVM2:
   ```sh
   sudo apt install lvm2
   ```

3. Verify time synchronization:
   ```sh
   timedatectl status
   ```

### 5. Deploying Ceph

#### 5.1. Install CephADM

1. Set environment variable:
   ```sh
   export CEPH_RELEASE=18.2.2
   ```

2. Download and install CephADM:
   ```sh
   curl -o cephadm https://raw.githubusercontent.com/ceph/ceph/${CEPH_RELEASE}/src/cephadm/cephadm
   chmod +x cephadm
   sudo ./cephadm add-repo --release ${CEPH_RELEASE}
   sudo ./cephadm install
   ```

3. Bootstrap Ceph:
   ```sh
   sudo cephadm bootstrap --mon-ip <Manager-IP>
   ```

#### 5.2. Add Hosts

1. Copy SSH keys to hosts:
   ```sh
   ssh-copy-id root@<host-ip>
   ```

2. Add hosts to the Ceph cluster:
   ```sh
   sudo ceph orch host add <host-name> <host-ip>
   ```

#### 5.3. Add Storage (OSDs)

1. List available devices:
   ```sh
   ceph orch device ls
   ```

2. Add all available devices as OSDs:
   ```sh
   ceph orch apply osd --all-available-devices
   ```

### 6. Create and Mount Ceph File System

1. Create Ceph file system:
   ```sh
   ceph fs volume create cephfs
   ```

2. Mount CephFS on Linux:
   ```sh
   mkdir -p /mnt/ceph
   mount -t ceph <mon-ip>:6789:/ /mnt/ceph
   ```

3. Install and configure Samba for Windows access:
   ```sh
   sudo apt install samba
   ```

## Conclusion

> [41:00 - End] Enjoy your new Ceph-powered FrankeNASS!

---

