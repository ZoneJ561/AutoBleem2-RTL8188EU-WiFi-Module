# AutoBleem 2 RTL8188EU Wi-Fi module

This package fixes Wi-Fi startup for a specific PlayStation Classic setup:

- AutoBleem 2.0 alpha1.1
- Linux kernel 4.4.22 and systemd 229
- Realtek RTL8188EU USB adapter, USB ID `0bda:8179`, kernel driver `r8188eu`

The console detected the adapter, but its original `wpa_supplicant` lacked the WEXT driver interface this adapter needed. This package provides a WEXT-enabled `wpa_supplicant` and a systemd service that starts it and restarts it if AutoBleem stops it. It also disables the competing dhcpcd supplicant hook.

The release ZIP contains the binary, Windows installer, verification and rollback scripts, build notes, and license notice. It does not contain your Wi-Fi password.

## Requirements

- A PlayStation Classic with the tested AutoBleem/kernel combination above.
- The RTL8188EU adapter (USB ID `0bda:8179`) connected through a powered USB hub.
- Wi-Fi credentials already saved on the console in `/etc/wpa_supplicant.conf`.
- Windows 11 OpenSSH client (`ssh.exe` and `scp.exe`) and root SSH access to the console.
- A reachable console IP. Use the RNDIS address, typically `169.254.215.100`, or pass the console’s reachable Wi-Fi/LAN address.

## Install on Windows

1. Download `AutoBleem2-RTL8188EU-WiFi-Module.zip` from the [v1.0.0 release](https://github.com/ZoneJ561/AutoBleem2-RTL8188EU-WiFi-Module/releases/tag/v1.0.0) and extract all files into one folder.
2. Open PowerShell in that folder.
3. Run a no-change preflight. Replace the address if your console uses a different reachable IP:

   ```powershell
   powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\AutoBleem2-WiFi-Install-Service.ps1 -DryRun -ConsoleAddress 169.254.215.100
   ```

4. If it reports `Dry run complete`, run the installer:

   ```powershell
   powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\AutoBleem2-WiFi-Install-Service.ps1 -ConsoleAddress 169.254.215.100
   ```

5. Review the summary, type `INSTALL`, then enter the console’s SSH password when prompted. Keep the password out of chat and logs.

The installer transfers the binary to the console, backs up the existing supplicant and configuration files under `/root/autobleem-wifi-service-backup-<date>`, installs and enables `autobleem-wext-wlan0.service`, and checks its status. It does not flash the kernel, alter firmware, or write to the AutoBleem USB drive. The Wi-Fi configuration file is backed up but not edited.

## Verify

After install and after a normal reboot, run:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\AutoBleem2-WiFi-Verify-Service.ps1 -ConsoleAddress 169.254.215.100
```

Use the console’s current reachable address. A successful check shows the service as `active`, `wpa_state=COMPLETED`, and a wlan0 IP address.

## Roll back

Use the exact backup directory printed by the installer:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\AutoBleem2-WiFi-Rollback-Service.ps1 -BackupPath /root/autobleem-wifi-service-backup-YYYYMMDD-HHMMSS -ConsoleAddress 169.254.215.100
```

Type `ROLLBACK` and enter the SSH password. Reboot the console normally afterward.

## Compatibility and limits

This was tested on one PlayStation Classic with the exact AutoBleem version, kernel, and adapter ID listed above. Other hardware revisions, AutoBleem builds, and adapters have not been verified. The installer replaces `/bin/wpa_supplicant`, adds a global dhcpcd `nohook` entry, and runs WEXT on wlan0, so use it for the tested adapter setup.

This is an unofficial community build, not an AutoBleem release. The binary is based on wpa_supplicant 2.6; see `COPYING.txt` in the ZIP for its license notice.
