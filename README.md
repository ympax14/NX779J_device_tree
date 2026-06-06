# Device tree — RedMagic 10 Air (NX779J)

| | |
|---|---|
| **Device** | Nubia RedMagic 10 Air |
| **Codename** | NX779J |
| **SoC** | Qualcomm Snapdragon 8 Elite (SM8650 / pineapple) |
| **Android** | 16 (API 36) |
| **Partition scheme** | Virtual A/B + dynamic partitions |
| **Encryption** | Hardware-wrapped key FBE + metadata encryption |
| **Maintainer** | [Ympax](https://github.com/ympax14) |

Shared device tree for TWRP, OrangeFox, and future AOSP ROM builds.

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

## Related repositories

| Repo | Description |
|---|---|
| [NX779J_device_tree](https://github.com/ympax14/NX779J_device_tree) | This repo — shared device tree |
| [NX779J_TWRP](https://github.com/ympax14/NX779J_TWRP) | TWRP `bootable/recovery` fork (TWRP-Test base + NX779J patches) |
| [NX779J_OrangeFox](https://github.com/ympax14/NX779J_OrangeFox) | OrangeFox `bootable/recovery` fork (fox_14.1 base + NX779J patches) + build scripts in `scripts/` |

---

## Prerequisites

- Linux build environment with Android build dependencies (`git`, `repo`, `python3`, `ccache`, …)
- ~200 GB free disk space, 16 GB RAM minimum

### Sync the TWRP Android 16 build tree

```bash
mkdir TWRP && cd TWRP
repo init -u https://github.com/TWRP-Test/platform_manifest_twrp_aosp -b twrp-16.0 --depth=1
repo sync -j$(nproc) --no-tags --no-clone-bundle
```

### Clone this device tree

```bash
git clone https://github.com/ympax14/NX779J_device_tree device/nubia/NX779J
```

---

## Building TWRP

```bash
# Switch bootable/recovery to the NX779J TWRP branch
cd bootable/recovery
git remote add nx779j https://github.com/ympax14/NX779J_TWRP
git fetch nx779j
git checkout nx779j/main -b nx779j-twrp-fixes
cd ../..

# Build
source build/envsetup.sh
export ALLOW_MISSING_DEPENDENCIES=true
lunch twrp_NX779J-bp2a-eng
mka recoveryimage -j$(nproc)
```

Output: `out/target/product/NX779J/recovery.img`

---

## Building OrangeFox

OrangeFox uses the fox_14.1 UI with fox_16.0 vendor tools.

```bash
# Clone OrangeFox vendor (fox_16.0)
git clone https://gitlab.com/OrangeFox/vendor/recovery.git -b fox_16.0 vendor/recovery

# Switch bootable/recovery to the NX779J OrangeFox branch
cd bootable/recovery
git remote add nx779j-of https://github.com/ympax14/NX779J_OrangeFox
git fetch nx779j-of
git checkout nx779j-of/main -b nx779j-orangefox-fixes
cd ../..

# Run setup and build using the included scripts
bash bootable/recovery/scripts/setup_device.sh
bash bootable/recovery/scripts/build.sh
```

Output: `out/target/product/NX779J/recovery.img`

---

## Flashing

> **Important:** NX779J uses A/B slots — flash to **both** slots.

### Using the flash script

```bash
bash bootable/recovery/scripts/flash.sh
```

### Manual

```bash
adb reboot bootloader
fastboot flash recovery_a out/target/product/NX779J/recovery.img
fastboot flash recovery_b out/target/product/NX779J/recovery.img
fastboot reboot recovery
```

---

## Technical notes

- Adapted from the NX789J Pro (RedMagic 9 Pro) device tree
- DRM display requires `drmSetMaster()` before each atomic commit (SM8650 driver rejects commits without DRM master)
- `servicemanager` needs `libperfetto_c.so` to start in recovery — bundled in the ramdisk
- `ssgtzd` (Qualcomm SSG TZ Daemon) needs `Utils_getTrace` — provided by `libminkdescriptor_shim.so`
- Metadata decryption uses a 60 s timeout to avoid indefinite hang on `smcinvoke_ioctl` when TrustZone is not ready

---

## Credits

- [OrangeFox Recovery Project](https://orangefox.tech)
- [TWRP Team](https://github.com/TeamWin)
- Adapted from NX789J Pro device tree
