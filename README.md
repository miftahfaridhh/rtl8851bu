# RTL8851BU Linux Driver

Linux driver for Realtek RTL8851BU/RTL8831BU WiFi chipset (USB).

## Supported Devices

- ipTIME AX900UA and ipTIME AX900 (but disabling the bluetooth function)
- Realtek RTL8851BU/RTL8831BU based adapters
- **USB ID `3625:010b`** (Third-party adapter - newly added)

## Tested Environment

This driver has been successfully tested on:

| Component | Details |
|-----------|---------|
| **Platform** | NVIDIA Jetson (aarch64/arm64) |
| **OS** | Ubuntu 18.04.6 LTS |
| **Kernel** | 4.9.337-tegra |
| **Architecture** | aarch64 (ARM 64-bit) |
| **USB Adapter** | USB ID `3625:010b` |

## Changes from Original Repository

This fork includes the following modifications:

### 1. Added ARM64/Jetson Platform Support
**File:** `Makefile`
- Changed `CONFIG_PLATFORM_I386_PC = n`
- Added `CONFIG_PLATFORM_ARM64_JETSON = y`
- Enabled platform-specific object files

### 2. Added New USB Device ID
**File:** `os_dep/linux/usb_intf.c`
- Added support for USB adapter with ID `3625:010b`

### 3. Created Platform Configuration for Jetson
**File:** `platform/arm64_jetson.mk`
- ARM64 cross-compile configuration
- Little endian support
- CFG80211 wireless extensions

## Installation

### Prerequisites
```bash
sudo apt-get update
sudo apt-get install build-essential linux-headers-$(uname -r)
```

### Build & Install
```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/rtl8851bu.git
cd rtl8851bu

# Build the driver
make

# Install the driver
sudo make install

# Load the driver
sudo modprobe 8851bu
```

### Verify Installation
```bash
# Check if module is loaded
lsmod | grep 8851bu

# Check wireless interface
iwconfig
```

## Uninstall
```bash
sudo make uninstall
```

## Troubleshooting

If the driver doesn't load automatically:
```bash
# Check dmesg for errors
dmesg | grep 8851

# Manually load the module
sudo insmod 8851bu.ko
```

## Credits

- Original driver by Realtek
- Original repository modifications by ipTIME
- ARM64/Jetson support and USB ID additions by [github.com/miftahfaridhh]

## License

This driver is released under the GPL license, following the original Realtek driver licensing.
