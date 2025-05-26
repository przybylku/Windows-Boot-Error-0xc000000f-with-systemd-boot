# Fixing Windows Boot Error 0xc000000f with systemd-boot (Dual Boot Linux + Windows 11)

## Problem

When trying to boot into Windows 11 after installing Linux with systemd-boot, you might see the following error:

```
Windows Boot Manager
Status: 0xc000000f
Info: The Boot Configuration Data for your PC is missing or contains errors.
```

This happens when the `bootmgfw.efi` file is missing or incorrectly copied, and the BCD (Boot Configuration Data) path is invalid.

---

## Goal

Fix the Windows boot entry in `systemd-boot` **without copying any files**, by pointing directly to the original EFI partition used by Windows.

---

## Step-by-step Instructions

### 1. Identify the Windows EFI partition

List all block devices and find the EFI partition:

```bash
lsblk -f
```

Look for a partition formatted as `vfat` (FAT32), often named `EFI` or `SYSTEM`. It might be something like `/dev/nvme0n1p1` or `/dev/sda1`.

### 2. Mount the Windows EFI partition

```bash
sudo mkdir -p /mnt/efi-win
sudo mount /dev/nvme0n1p1 /mnt/efi-win
```

> Replace `/dev/nvme0n1p1` with your actual device name.

### 3. Check if `bootmgfw.efi` exists

```bash
ls /mnt/efi-win/EFI/Microsoft/Boot/bootmgfw.efi
```

If the file exists, you're good to proceed.

### 4. Create a new `windows.conf` boot entry for systemd-boot

(Optional) Remove the old entry:

```bash
sudo rm -f /boot/efi/loader/entries/windows.conf
```

Create the file:

```bash
sudo nano /boot/efi/loader/entries/windows.conf
```

Add the following content:

```ini
title   Windows 11
efi     /EFI/Microsoft/Boot/bootmgfw.efi
```

Save and close (`Ctrl + O`, `Enter`, `Ctrl + X`).

### 5. (Optional) Bind-mount Microsoft EFI directory

If `systemd-boot` doesn't see `/mnt/efi-win`, bind-mount it like this:

```bash
sudo mount --bind /mnt/efi-win/EFI/Microsoft /boot/efi/EFI/Microsoft
```

> Use this method instead of copying files manually.

### 6. Make bind-mount persistent (optional)

Edit `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Add this line:

```fstab
/mnt/efi-win/EFI/Microsoft /boot/efi/EFI/Microsoft none bind 0 0
```

Ensure the directories exist:

```bash
sudo mkdir -p /mnt/efi-win /boot/efi/EFI/Microsoft
```

### 7. Reboot

```bash
sudo reboot
```

On boot, choose “Windows 11” in the `systemd-boot` menu. It should work correctly now.

---

## Troubleshooting

* Make sure the `bootmgfw.efi` file is not missing or replaced
* Avoid copying EFI files to `/boot/efi` – always use the original Windows EFI partition
* Use `bootctl list` to inspect visible boot entries in systemd-boot

---

## References

* [Arch Wiki: systemd-boot](https://wiki.archlinux.org/title/Systemd-boot)
* [Microsoft Docs: Boot Configuration Data](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/bcd-system-store)

---

## Author

This guide was created after solving a real-world dual-boot issue with Pop\_OS and Windows 11.

Feel free to fork, contribute, or report issues.

---

MIT License
