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

## Step 3: Configure Isolated Networking

### Goal

Create a private cybersecurity lab where Kali Linux can communicate with Metasploitable 2 without exposing the vulnerable target to the internet or the normal home network.

Kali does not need internet access for basic lab exercises. Internet access is optional and useful for updates, installing tools, and researching documentation.

### 3.1 Configure Networking in UTM

Shut down both virtual machines before changing their network settings.

#### Kali Linux

1. Open UTM and select the Kali Linux VM.
2. Open its settings and locate **Network**.
3. Configure one network adapter for the private lab network (**Host Only** or the equivalent custom host network option available in your UTM configuration).
4. Optionally add a second adapter using **Shared Network (NAT)** if Kali needs internet access.
5. Do not use Bridged networking for the lab.

#### Metasploitable 2

1. Open UTM and select the Metasploitable 2 VM.
2. Open its settings and locate **Network**.
3. Configure its only network adapter to use the same private lab network as Kali.
4. Do not attach Metasploitable 2 to Shared Network (NAT) or Bridged networking.

**Important:** Both VMs must use the same private network. UTM's available modes vary by configuration, so verify that the selected network actually isolates the target. Selecting Host Only does not necessarily create a host interface or automatically assign IP addresses.

### 3.2 Plan the Lab IP Addresses

Use the following example addresses if the private network is configured for `192.168.50.0/24` and the addresses are available.

| Device | IP address | Subnet mask |
|---|---|---|
| Kali lab adapter | `192.168.50.10` | `255.255.255.0` |
| Metasploitable 2 lab adapter | `192.168.50.20` | `255.255.255.0` |
| Optional Mac host interface | `192.168.50.1` | `255.255.255.0` |

The Mac address is optional and should be assigned only if UTM's private network provides a host interface on that subnet.

- Keep the Mac's normal Wi-Fi address unchanged. Its current network uses the `10.0.0.x` subnet.
- Leave the gateway and DNS blank on the lab adapters.
- Do not use these example addresses until you confirm the actual private network's subnet and available addresses.
- Configure the addresses on the correct lab interfaces, not on unrelated network adapters.

### 3.3 Configure a Manual IP Address in Kali Linux

1. Start Kali Linux and log in.
2. Open Terminal and run:

   ```bash
   ip addr
   ```

3. Identify the interface connected to the private lab network. It may be named `eth0`, `enp0s1`, `ens3`, or something else.
4. Open Kali's network settings from the desktop.
5. Find the wired connection for the lab adapter and open its settings.
6. Select the **IPv4** tab.
7. Change the IPv4 method to **Manual**.
8. Add the following settings:

   - **Address:** `192.168.50.10`
   - **Netmask:** `255.255.255.0`
   - **Gateway:** Leave blank
   - **DNS:** Leave blank

9. Click **Apply** and reconnect the lab connection if necessary.
10. Run `ip addr` again to confirm that the address was applied to the correct interface.

If your desktop environment does not offer these settings, identify the interface first and configure it using the network-management tools available in your Kali installation.

### 3.4 Configure a Manual IP Address in Metasploitable 2

1. Start Metasploitable 2.
2. Log in using the default lab credentials:

   - **Username:** `msfadmin`
   - **Password:** `msfadmin`

3. Identify its network interfaces:

   ```bash
   ifconfig
   ```

4. Find the interface connected to the private lab network. It is commonly named `eth0`, but verify this before continuing.
5. For a temporary IP configuration, run the following command, replacing `eth0` if your interface has a different name:

   ```bash
   sudo ifconfig eth0 192.168.50.20 netmask 255.255.255.0 up
   ```

6. Verify the address:

   ```bash
   ifconfig
   ```

This command configures a temporary address. The setting may disappear after reboot. Configure a persistent address only after confirming the interface and network settings.

### 3.5 Test Connectivity Between the VMs

1. Start both VMs and ensure their private lab adapters are connected.
2. In Kali, verify the lab interface:

   ```bash
   ip addr
   ```

3. Ping Metasploitable 2:

   ```bash
   ping -c 4 192.168.50.20
   ```

4. If replies arrive, Kali can reach Metasploitable 2 over the private network.

If the ping fails, check the following:

- Both VMs are running.
- Both adapters use the same private network.
- The actual network subnet matches the example addresses.
- The IP addresses are configured on the correct interfaces.
- No other device is using either address.

If the addresses appear correct but the connection still fails, inspect UTM's network configuration before changing more settings.

### 3.6 Optional: Enable Internet Access for Kali

Internet access is **not required** for basic practice against Metasploitable 2.

It can be useful for:

- Updating Kali Linux.
- Installing additional packages and tools.
- Accessing cybersecurity documentation.

If internet access is needed:

1. Shut down Kali.
2. Open its UTM settings.
3. Keep its private lab adapter connected.
4. Add or enable a separate **Shared Network (NAT)** adapter.
5. Start Kali and test internet connectivity:

   ```bash
   ping -c 4 1.1.1.1
   ```

6. Test DNS resolution:

   ```bash
   ping -c 4 google.com
   ```

If the first test works but the second fails, investigate DNS configuration.

Keep the private lab adapter for communication with Metasploitable 2. Do not add an internet-facing adapter to Metasploitable 2.

### 3.7 Security Checklist

- [ ] Kali and Metasploitable 2 use the same private lab network.
- [ ] The private network uses a confirmed subnet that does not overlap with the normal home network.
- [ ] Kali has the correct manual lab IP address.
- [ ] Metasploitable 2 has the correct manual lab IP address.
- [ ] Kali can reach Metasploitable 2 over the private network.
- [ ] Metasploitable 2 has no Shared Network (NAT) or Bridged adapter.
- [ ] Kali internet access is optional and uses a separate adapter.
- [ ] Any optional Mac host address is configured only when UTM provides a compatible host interface.
- [ ] The lab's isolation has been verified before beginning security exercises.

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

