# MacBookPro13,3 dGPU-only Performance Guide

This guide covers generating CPUFriend data, disabling the boot chime, and applying additional tweaks for a MacBookPro13,3 spoofed as MacBookPro14,1. The goal is maximum performance when using the AMD dGPU.

## 1. Generating CPUFriend Data

CPUFriend requires a matching `CPUFriendDataProvider.kext` to control macOS power management. The OpenCore Legacy Patcher repository already includes CPUFriend, but you need to generate the DataProvider yourself. You can do this directly on macOS using the official script from Acidanthera.

### Steps

1. Download the CPUFriend source:
   ```sh
   git clone https://github.com/acidanthera/CPUFriend.git
   ```
2. Build the project (requires Xcode command line tools):
   ```sh
   cd CPUFriend
   xcodebuild -configuration Release
   ```
   The built `CPUFriend.kext` can be found in `build/Release`.
3. Generate a base plist for MacBookPro14,1:
   ```sh
   cd Tools
   ./ResourceConverter.sh --platform MacBookPro14,1
   ```
   This creates `CPUFriendDataProvider.kext` with the default Apple power profile for that model.
4. Copy both `CPUFriend.kext` and `CPUFriendDataProvider.kext` into `EFI/OC/Kexts/` on your EFI partition.
5. Update `config.plist` to add these kexts under `Kernel -> Add` and ensure `Kernel -> Quirks -> CustomSMBIOSGuid` is **True**.
6. Rebuild OpenCore and reboot.

This configuration locks Turbo Boost and avoids throttling by using the stock MacBookPro14,1 power profile. If you wish to modify the profile (for example, force maximum multipliers), edit the generated plist before building the DataProvider.

## 2. Disabling the Boot Chime

The chime can be disabled either by NVRAM or OpenCore settings.

### NVRAM Method

```sh
sudo nvram SystemAudioVolume=%80
sudo nvram StartupMute=%01
```

### OpenCore Method

Edit `config.plist` and set:

```
UEFI -> Audio -> AudioSupport = False
```

## 3. Installation Script Example

Below is a small script that copies the generated kexts to the EFI partition and updates permissions. Run it after building CPUFriend:

```sh
#!/bin/bash
EFI=/Volumes/EFI
mkdir -p "$EFI/EFI/OC/Kexts"
cp CPUFriend/build/Release/CPUFriend.kext "$EFI/EFI/OC/Kexts/"
cp CPUFriend/Tools/CPUFriendDataProvider.kext "$EFI/EFI/OC/Kexts/"
chown -R root:wheel "$EFI/EFI/OC/Kexts/CPUFriend.kext" "$EFI/EFI/OC/Kexts/CPUFriendDataProvider.kext"
```

Mount your EFI partition (for example using `diskutil mount EFI`), run the script, then reboot.

## 4. Extra Performance Tips

* `pmset -a gpuswitch 0` keeps macOS using the discrete AMD GPU permanently.
* Ensure `Kernel -> Quirks -> AppleXcpmCfgLock` is **True** in OpenCore if your firmware allows writing MSR 0xE2.
* Consider using `cpufreq` menu bar tools like `Intel Power Gadget` to monitor frequencies and ensure Turbo remains active.

## 5. Automator Mini Setup (optional)

You can wrap the copy steps in an Automator application:

1. Open Automator and create a new **Application**.
2. Add a **Run Shell Script** action with the installation commands shown above.
3. Save the application as `Install CPUFriend.app` and run it whenever you rebuild the kexts.

---

This setup focuses on stability and maximum performance. Always monitor temperatures when locking Turbo Boost to avoid overheating.
