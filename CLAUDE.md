# kiro-bootloader-grub-nemesis

Pacman hooks keeping a **GRUB** Kiro install bootable across upgrades.
Nemesis = development variant; promote to `kiro-bootloader-grub` once proven.

## Layout
- `usr/bin/kiro-grub-install` — firmware-aware `grub-install` wrapper; the
  heart of the package. BIOS disk is **discovered**, never hardcoded.
- `etc/pacman.d/hooks/*.hook` — installed to `/usr/share/libalpm/hooks/`
  by the PKGBUILD (build dir: `KIRO-PKG-BUILD-APPS/kiro-bootloader-grub-nemesis`).

## Invariants — do not break
- Both hooks MUST keep `Depends = grub` so they stay inert on systemd-boot
  (Kiro's default). Removing it makes every `pacman -Syu` error on
  systemd-boot systems.
- Never reintroduce a hardcoded boot disk (`/dev/sda`). Resolve from where
  `/boot` lives. This is the whole point vs. the old ArcoLinux hook.
- The install hook re-uses the existing `EFI/<id>` on the ESP; do not pass a
  fixed `--bootloader-id` that could create a duplicate UEFI entry.

## Promotion to production
Strip `-nemesis`: pkgname/url/conflicts → `kiro-bootloader-grub`, then add
to the ISO default package set. Conflicts/provides flip accordingly.
