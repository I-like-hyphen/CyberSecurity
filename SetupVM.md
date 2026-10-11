# Cybersecurity Practice Lab Setup
## Kali Linux + Metasploitable 2 on Apple Silicon using UTM

## Overview

This guide explains how to set up an isolated cybersecurity practice lab on an Apple Silicon Mac using UTM.

| Virtual Machine | RAM | CPU Cores | Disk Space | Architecture |
|---|---:|---:|---:|---|
| Kali Linux | 4 GB | 4 | 64 GB | ARM64 |
| Metasploitable 2 | 1 GB | 1 | 10 GB | x86 (emulated) |

**Important:** Metasploitable 2 is intentionally vulnerable. Keep it isolated from public networks and use it only in your own lab.

---

## Step 1: Install Kali Linux (Attacker VM)

### 1. Download Kali Linux

1. Visit the [official Kali Linux website](https://www.kali.org/get-kali/).
2. Download the **ARM64 installer ISO** for Apple Silicon.

### 2. Create the virtual machine

1. Open UTM.
2. Click **Create a New Virtual Machine**.
3. Choose **Virtualize**.
4. Select **Linux**.
5. Choose the downloaded Kali Linux ARM64 ISO.
6. Allocate the following resources:
   - RAM: 4 GB
   - CPU cores: 4
   - Disk space: 64 GB
7. Complete the VM creation wizard.

### 3. Troubleshoot the installer display

If the installer produces a black screen:

1. Shut down the VM.
2. Open its settings and inspect the Display configuration.
3. If necessary, try an alternative console or serial-based installation method supported by your UTM version.
4. Avoid removing the display device unless you have confirmed that your installation method supports serial output.

### 4. Install Kali Linux

1. Start the VM from the installer ISO.
2. Select **Graphical Install** or **Install**.
3. Follow the prompts to configure:
   - Language and location
   - Keyboard layout
   - Hostname
   - User account and password
   - Disk partitioning
4. Select the default Xfce desktop environment if offered.
5. Complete the installation and restart the VM.

### 5. Boot into the installed system

1. Shut down the VM if necessary.
2. Eject or disconnect the installer ISO.
3. Restore the normal display configuration if you changed it.
4. Start the VM and verify that Kali Linux boots from its virtual disk.

---

## Step 2: Set Up Metasploitable 2 (Target VM)

### 1. Download Metasploitable 2

1. Download the Metasploitable 2 archive from [SourceForge](https://sourceforge.net/projects/metasploitable/).
2. Extract the archive on your Mac.
3. Locate the virtual disk file, usually named `Metasploitable.vmdk`.

### 2. Convert the virtual disk to QCOW2

Install QEMU using Homebrew if it is not already installed:

```bash
brew install qemu
```

Navigate to the directory containing the extracted VMDK file:

```bash
cd /path/to/metasploitable
```

Convert the virtual disk:

```bash
qemu-img convert -f vmdk -O qcow2 Metasploitable.vmdk Metasploitable.qcow2
```

Replace the example directory path and filename with the actual location and name of your file.

### 3. Create the Metasploitable VM

1. Open UTM.
2. Click **Create a New Virtual Machine**.
3. Choose **Emulate** because Metasploitable 2 uses an x86 architecture.
4. Select **Other** as the operating system type.
5. Configure the VM:
   - RAM: 1024 MB (1 GB)
   - CPU cores: 1
   - Boot device: None, if available
6. Finish the creation wizard.

### 4. Configure the virtual machine

1. Open the VM settings.
2. Disable **UEFI Boot** under the QEMU settings, if enabled.
3. Under Drives, remove the default empty drive if one was created.
4. Import or attach `Metasploitable.qcow2` as the virtual hard disk.
5. Verify that the attached disk is configured as a bootable hard drive.

**Note:** Metasploitable 2 is typically a 32-bit x86 system. Choose a compatible emulated architecture and machine configuration. If the VM does not boot, verify the image and drive interface before changing other settings.

### 5. Start Metasploitable 2

1. Start the VM.
2. Wait for the login prompt.
3. Use the default credentials:

   - **Username:** `msfadmin`
   - **Password:** `msfadmin`

4. Confirm that you can log in successfully.

---

## Step 3: Configure Networking

### 1. Configure Kali Linux
- Open **UTM → Kali → Edit → Network**.
- Enable **Host Only** for the private lab network.
- Add a second network adapter using **Shared Network** for internet access.
- Start Kali and confirm internet works:

```bash
ping -c 4 google.com
```

### 2. Configure Metasploitable 2
- Open **UTM → Metasploitable 2 → Edit → Network**.
- Select **Host Only**.
- Do not enable Shared Network or Bridged.
- Start the VM and log in:

```text
Username: msfadmin
Password: msfadmin
```

### 3. Test the Private Connection

In Kali, run:

```bash
ip -4 addr
```

In Metasploitable 2, run:

```bash
ifconfig
```

Find each VM's Host Only IP address. From Kali, test the connection:

```bash
ping -c 4 <METASPLOITABLE_IP>
```

Replace `<METASPLOITABLE_IP>` with Metasploitable 2's actual IP address.

**Goal:** Kali has internet access, and Kali can communicate with Metasploitable 2 over the private network.

**Security:** Keep Metasploitable 2 on Host Only to isolate it from your normal network.

---

## Step 4: Verify the Lab

Use this checklist before beginning exercises:

- [ ] Kali Linux boots successfully.
- [ ] Metasploitable 2 boots successfully.
- [ ] Both VMs have sufficient RAM and disk space.
- [ ] Both VMs are connected to the intended isolated network.
- [ ] Kali can reach Metasploitable 2.
- [ ] Metasploitable 2 is not exposed to the public internet.
- [ ] Only authorized lab systems are used for testing.

## Troubleshooting

| Problem | What to check |
|---|---|
| Kali displays a black screen | Display configuration, installer mode, and ARM64 ISO |
| Metasploitable says “No bootable device” | Correct QCOW2 disk attached, UEFI disabled, compatible architecture |
| Kali cannot reach Metasploitable | Network mode, subnet, IP addresses, and interface status |
| VM runs slowly | RAM and CPU allocation, plus host resource usage |
| Disk is not detected | Drive attachment, disk format, and virtual storage interface |

## Safety Reminder

Metasploitable 2 is deliberately insecure and intended for training. Keep it on an isolated network, avoid bridging it to an untrusted network, and do not expose its vulnerable services to the internet.

