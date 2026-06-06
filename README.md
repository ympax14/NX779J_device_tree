# TWRP / OrangeFox device tree — RedMagic 10 Air (NX779J)

| | |
|---|---|
| **Device** | Nubia RedMagic 10 Air |
| **Codename** | NX779J |
| **SoC** | Qualcomm Snapdragon 8 Elite (SM8650 / pineapple) |
| **Android** | 16 (API 36) |
| **Partition scheme** | A/B (Virtual A/B + dynamic partitions) |
| **Encryption** | Hardware-wrapped key FBE + metadata encryption |
| **Maintainer** | [Ympax](https://github.com/ympax14) |

---

## Status

| Feature | Status |
|---|---|
| Booting to recovery | ✅ |
| ADB | ✅ |
| Display (DRM atomic) | ✅ |
| Fastbootd | ✅ |
| Flashing (zip / image) | ✅ |
| MTP | ✅ |
| USB-OTG | ✅ |
| Data decryption (FBE + metadata) | 🔧 WIP |
| Vibrator | 🔧 WIP |

---

## Building

### Prerequisites

- Linux build environment
- Android build dependencies installed (`python3`, `git`, `repo`, `ccache`, etc.)
- At least 200 GB of free disk space and 16 GB of RAM

### 1. Sync the TWRP Android 16 build tree

```bash
mkdir TWRP && cd TWRP
repo init -u https://github.com/TWRP-Test/platform_manifest_twrp_aosp -b twrp-14.1 --depth=1
repo sync -j$(nproc) --no-tags --no-clone-bundle
```

### 2. Clone this device tree

```bash
git clone https://github.com/ympax14/NX779J_device_tree device/nubia/NX779J
```

### 3a. Build TWRP

```bash
source build/envsetup.sh
export ALLOW_MISSING_DEPENDENCIES=true
lunch twrp_NX779J-bp2a-eng
mka recoveryimage -j$(nproc)
```

Output: `out/target/product/NX779J/recovery.img`

### 3b. Build OrangeFox (recommended)

OrangeFox uses the fox_14.1 UI with fox_16.0 vendor tools. Use the dedicated build scripts:

```bash
# Clone OrangeFox vendor
git clone https://gitlab.com/OrangeFox/vendor/recovery.git -b fox_16.0 vendor/recovery

# Switch bootable/recovery to the NX779J-patched branch
cd bootable/recovery
git remote add nx779j https://github.com/ympax14/NX779J_TWRP
git fetch nx779j
git checkout nx779j/nx779j-fox14.1-fixes -b nx779j-fox14.1-fixes
cd ../..

# Clone OrangeFox build scripts and build
cd ..
git clone https://github.com/ympax14/NX779J_OrangeFox
bash NX779J_OrangeFox/setup_device.sh
bash NX779J_OrangeFox/build.sh
```

See [NX779J_OrangeFox](https://github.com/ympax14/NX779J_OrangeFox) for the complete guide.

---

## Flashing

> **Important:** The NX779J uses A/B slots. Flash to **both** slots.

### From fastboot

```bash
adb reboot bootloader
fastboot flash recovery_a out/target/product/NX779J/recovery.img
fastboot flash recovery_b out/target/product/NX779J/recovery.img
fastboot reboot recovery
```

### Using the flash script

```bash
bash NX779J_OrangeFox/flash.sh
```

---

## Notes

- This device tree was adapted from the NX789J Pro (RedMagic 9 Pro series)
- The recovery uses **DRM atomic commit** for display (`drmSetMaster` before each commit required)
- Metadata decryption requires `servicemanager`, `ssgtzd`, `keymint` and `keystore2` running in recovery — all included
- `libperfetto_c.so` is bundled in the recovery ramdisk so that `servicemanager` can start
- `libminkdescriptor_shim.so` provides the missing `Utils_getTrace` symbol for `ssgtzd`

---

## Credits

- [OrangeFox Recovery Project](https://orangefox.tech)
- [TWRP Team](https://github.com/TeamWin)
- Adapted from NX789J Pro device tree
