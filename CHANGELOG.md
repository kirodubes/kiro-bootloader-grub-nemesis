# Changelog

## 2026.06.08

### What Changed
- Initial scaffold of `kiro-bootloader-grub-nemesis` — pacman hooks that
  keep a GRUB Kiro install bootable across `grub`/kernel upgrades. Created
  to close the boot-robustness gap identified in the CTT/GRUB study: BIOS
  installs (always GRUB) and UEFI-GRUB opt-ins were exposed to the classic
  Arch `grub_*` symbol brick because nothing re-ran `grub-install` after a
  `grub` upgrade.

### Technical Details
- Two hooks installed to `/usr/share/libalpm/hooks/`:
  - `kiro-grub-install.hook` — `Type=Package`, `Target=grub`, runs
    `/usr/bin/kiro-grub-install` on Install/Upgrade.
  - `kiro-grub-mkconfig.hook` — runs `grub-mkconfig`. Kernels are matched
    by the file every kernel installs (`Type=Path`,
    `Target=usr/lib/modules/*/vmlinuz`) rather than by package name, so it
    catches every kernel (linux/-lts/-zen/-cachyos/-hardened/-rt/-xanmod/
    -tkg/custom) with no list to maintain; a second `Type=Package` trigger
    covers `grub` + microcode.
- `/usr/bin/kiro-grub-install` discovers the target rather than assuming
  `/dev/sda`: UEFI uses `--efi-directory=/boot/efi` and re-uses the
  existing `EFI/<id>`; BIOS resolves the boot disk via
  `grub-probe`+`lsblk -no pkname` (handles `vda`/`nvme*`/`mmcblk*` and RAID).
- Both hooks guarded with `Depends = grub` so they stay inert on the
  default systemd-boot installs.
- Helper script intentionally off the Kiro flow-script template (no banner
  logging / sleep-on-error — it runs inside a pacman hook); to be listed in
  Kiro-HQ/TEMPLATE_EXCLUSIONS.md.

### Files Modified
- `etc/pacman.d/hooks/kiro-grub-install.hook` (new)
- `etc/pacman.d/hooks/kiro-grub-mkconfig.hook` (new)
- `usr/bin/kiro-grub-install` (new)
- `README.md`, `CHANGELOG.md`, `CLAUDE.md` (new)
- build: `KIRO-PKG-BUILD-APPS/kiro-bootloader-grub-nemesis/{PKGBUILD,.current-version,build.sh,readme.install}` (new)
