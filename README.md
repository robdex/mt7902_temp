# 🎯 mt7902 driver development (✅ Working)
We are trying to develop the driver for the Mediatek mt7902 wifi 6E chip

## ✅ Tested On (Verified Working)
This fix has been verified and is confirmed working on:

* **Brand:** ASUS
* **Model:** Vivobook Go (E1404FA)
* **Chipset:** MediaTek MT7902 (WiFi 6E)
* **Kernel Version:** 6.19.0 (Linux)
* **OS:** Ubuntu 24.04 (or similar Debian-based distros)

## 🚀 Easy Automatic Fix (Recommended)
If you want to quickly fix your WiFi and Bluetooth on any modern kernel, follow these steps:

1. **Open your terminal** and clone the repository:
   ```bash
   git clone --depth 1 https://github.com/OnlineLearningTutorials/mt7902_temp
   cd mt7902_temp
   ```

2. **Run the automatic fix script** with sudo:
   ```bash
   sudo bash fix_my_wifi.sh
   ```

### 📖 What this script does:
* **Checks for dependencies:** Ensures you have `gcc`, `make`, and your current `kernel-headers` installed.
* **Compiles Drivers:** Automatically builds both WiFi and Bluetooth drivers for your exact kernel version.
* **Persistent Fix:** Installs a system service that ensures your WiFi stays active even after you restart your computer.
* **Safety:** Installs modules into a custom directory (`/lib/modules/mt7902_custom`) to avoid messing with your default system files.

> [!NOTE]
> You will need an internet connection (via Ethernet or USB tethering from your phone) the first time you run this to download the necessary build tools.

## 🔧 Firmwares used
Firmwares are stored in `mt7902_firmware` folder.
Recently released firmware are in the `mt7902_firmware/latest` folder.

## 📁 Cloning the repository
Clone the repository to your local pc
  ```
  git clone https://github.com/OnlineLearningTutorials/mt7902_temp
  ```
If you don't want to clone past history than 
  ```
  git clone --depth 1 https://github.com/OnlineLearningTutorials/mt7902_temp
  ```


## 📱 Bluetooth ✅ (Working)
> [!WARNING]
> If bluetooth driver conflict with `gen4-mt7902` than please remove the bluetooth firmware so that it not interfere with this driver
> ``` sudo rm /lib/firmware/mediatek/mt7902/BT_RAM_CODE_MT7902_1_1_hdr.bin.zst ```
> This project uses the firmware
> ``` /lib/firmware/mediatek/BT_RAM_CODE_MT7902_1_1_hdr.bin.zst ```

To enable bluetooth go to the directory of your current kernel version. ``
Like if you have kernel linux-6.16 than go to the directory `./linux-6.16/drivers/bluetooth` .
Open terminal in this directory and compile with command `make`.
Two kernel modules are compiled `btusb.ko` and `btmtk.ko`.
To enable bluetooth in your device remove the btusb and btmtk in your system and install these two files, use commands
```
sudo rmmod btusb 
sudo rmmod btmtk

sudo insmod btmtk.ko
sudo insmod btusb.ko

```
Now check your bluetooth is working now.

## 💻 WiFi ✅ (Working)
> [!IMPORTANT]
> A working repo with some limitation is [here](https://github.com/hmtheboy154/gen4-mt7902)

WiFi driver for the mt7902, recently released by mediatek is inside the `latest` folder. 

If you are using Ubuntu than just go to the `latest` folder and run the following command in the termianl. 
```
make
```

It will compile all modules, compress it and install it (replace original kernel module with the modified module). If you are some other distro or not want all steps and only wants to compile the code, than run in the termianl 
```
make module_compile
```
To compress the module you compiled, than run in terminal
```
make module_compress
```
To install the compressed module to the system's kernel module, run in terminal
```
make module_install
```

### WiFi only (no Bluetooth)

If you only need WiFi, skip `fix_my_wifi.sh` and run these commands manually:

```bash
# 1. Install dependencies
sudo apt install build-essential linux-headers-$(uname -r) bc zstd

# 2. Compile and install
cd latest/
make module_compile
make module_compress
sudo make module_install

# 3. Load modules
sudo modprobe cfg80211
sudo modprobe mac80211
sudo insmod mt76.ko
sudo insmod mt76-connac-lib.ko
sudo insmod mt792x-lib.ko
sudo insmod mt7921/mt7921-common.ko
sudo insmod mt7921/mt7921e.ko
```

To make WiFi persistent across reboots, use the provided files `mt7902-wifi.sh` and `mt7902-wifi.service` from this repository:

```bash
# 1. Copy and install the startup script
sudo cp mt7902-wifi.sh /usr/local/bin/mt7902-wifi.sh
sudo chmod +x /usr/local/bin/mt7902-wifi.sh

# 2. Copy the compiled modules to a safe location
sudo mkdir -p /lib/modules/mt7902_custom
sudo cp latest/*.ko /lib/modules/mt7902_custom/
sudo cp latest/mt7921/*.ko /lib/modules/mt7902_custom/

# 3. Install and enable the systemd service
sudo cp mt7902-wifi.service /etc/systemd/system/mt7902-wifi.service
sudo systemctl daemon-reload
sudo systemctl enable mt7902-wifi.service
sudo systemctl start mt7902-wifi.service
```

### Reverting / Uninstalling

To completely remove the custom drivers and restore the system to its original state:

```bash
# 1. Stop and disable the systemd service (if installed)
sudo systemctl stop mt7902-wifi.service 2>/dev/null || true
sudo systemctl stop mt7902.service 2>/dev/null || true
sudo systemctl disable mt7902-wifi.service 2>/dev/null || true
sudo systemctl disable mt7902.service 2>/dev/null || true
sudo rm -f /etc/systemd/system/mt7902-wifi.service
sudo rm -f /etc/systemd/system/mt7902.service
sudo systemctl daemon-reload

# 2. Remove the startup script
sudo rm -f /usr/local/bin/mt7902-wifi.sh
sudo rm -f /usr/local/bin/mt7902-setup.sh

# 3. Unload the custom modules
sudo rmmod mt7921e mt7921_common mt792x_lib mt76_connac_lib mt76 2>/dev/null || true

# 4. Remove the custom modules directory
sudo rm -rf /lib/modules/mt7902_custom

# 5. Restore the original kernel modules
sudo apt reinstall linux-modules-$(uname -r)

# 6. Refresh module dependencies and reload the original driver
sudo depmod -a
sudo modprobe mt7921e
```

After step 6, the stock kernel driver will be active again.

### After a kernel update

The custom modules are installed directly into the running kernel's module directory (`/lib/modules/$(uname -r)/...`), so they survive reboots without any extra steps.

> **Currently compiled for:** `6.17.0-061700-generic`

However, if `apt upgrade` installs a **new kernel version**, the new kernel will have its own separate module directory and will fall back to the stock (broken) driver. To check if a kernel update is pending:

```bash
apt list --upgradable 2>/dev/null | grep linux-modules
```

If a new kernel appears, after rebooting into it simply re-run the fix script from the repository directory:

```bash
sudo bash fix_my_wifi.sh
```

### Kernel < 6.19 compatibility

The `latest/` sources require kernel 6.19 or newer by default due to the Airoha NPU offloading header (`linux/soc/airoha/airoha_offload.h`). This repository includes a fix that guards that header behind a kernel version check, so compilation on kernels 6.17 and above is supported.

If you are on **kernel 6.17**, make sure you have pulled the latest version of this repository before compiling.
