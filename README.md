# RTL8851BU Linux Driver

Linux driver for Realtek RTL8851BU/RTL8831BU WiFi chipset (USB).

## Supported Devices

- ipTIME AX900UA and ipTIME AX900
- TP-Link TX10UB (USB ID `3625:010b`)
- Realtek RTL8851BU/RTL8831BU based adapters

> **Note:** This driver provides **WiFi functionality only**. Bluetooth must be disabled for the WiFi driver to work properly. See [Disabling Bluetooth](#important-disabling-bluetooth-required) section below.

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
sudo apt-get install build-essential linux-headers-$(uname -r) usb-modeswitch
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
sudo depmod -a

# Load the driver
sudo modprobe 8851bu
```

### Verify Installation
```bash
# Check if module is loaded
lsmod | grep 8851bu

# Check wireless interface
iwconfig

# Check network interface
ip link show
```

## Important: Disabling Bluetooth (Required)

The RTL8851BU chip is a combo WiFi+Bluetooth device. The Linux `btusb` driver may conflict with this WiFi driver. **You must disable Bluetooth functionality** for WiFi to work properly.

### Step 1: Blacklist btusb Module
Create a blacklist configuration to prevent btusb from loading:
```bash
sudo tee /etc/modprobe.d/blacklist-btusb-tplink.conf << 'EOF'
install btusb /bin/false
EOF
```

### Step 2: Remove btusb Module (if loaded)
```bash
sudo rmmod btusb 2>/dev/null
```

### Step 3: Configure USB Modeswitch
Create USB modeswitch configuration for the device:
```bash
sudo tee /etc/usb_modeswitch.d/0bda:1a2b << 'EOF'
TargetVendor=0x3625
TargetProduct=0x010b
StandardEject=1
EOF
```

### Step 4: Create Udev Rules
Create udev rules for automatic device switching and driver loading:
```bash
sudo tee /etc/udev/rules.d/99-tplink-tx10ub.rules << 'EOF'
ACTION=="add", ATTR{idVendor}=="0bda", ATTR{idProduct}=="1a2b", RUN+="/usr/sbin/usb_modeswitch -K -v 0bda -p 1a2b"
ACTION=="add", ATTR{idVendor}=="3625", ATTR{idProduct}=="010b", RUN+="/sbin/modprobe 8851bu"
EOF
```

### Step 5: Enable Module Autoload on Boot
```bash
echo "8851bu" | sudo tee /etc/modules-load.d/8851bu.conf
```

### Step 6: Reload Udev Rules and Reboot
```bash
sudo udevadm control --reload-rules
sudo reboot now
```

### Verify Configuration
After reboot, verify all configurations are in place:
```bash
# Check module autoload config
cat /etc/modules-load.d/8851bu.conf

# Check USB modeswitch config
cat /etc/usb_modeswitch.d/0bda:1a2b

# Check udev rules
cat /etc/udev/rules.d/99-tplink-tx10ub.rules

# Check btusb blacklist
cat /etc/modprobe.d/blacklist-btusb-tplink.conf
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
