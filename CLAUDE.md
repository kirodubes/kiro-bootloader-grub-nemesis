# kiro-bootloader-grub-nemesis

Pacman hooks keeping a **GRUB** Kiro install bootable across upgrades.
Nemesis = development variant; promote to `kiro-bootloader-grub` once proven.

## Layout
- `usr/bin/kiro-grub-install` — firmware-aware `grub-install` wrapper; the
  heart of the package. BIOS disk is **discovered**, never hardcoded.
- `etc/pacman.d/hooks/*.hook` — installed to `/usr/share/libalpm/hooks/`
  by the PKGBUILD (build dir: `KIRO-PKG-BUILD-APPS/kiro-bootloader-grub-nemesis`).

## Invariants — do not break
- `grub` is part of archiso = ALWAYS installed, so package presence is no
  signal. Both hooks MUST guard via `kiro-grub-is-active` (GRUB live =
  `loader.conf` absent AND `grub.cfg` present). Without the guard the hooks
  would write a stray `grub.cfg` / grub EFI entry onto systemd-boot systems.
  Do not "simplify" the guard away to a `Depends = grub` check — that is
  always true and protects nothing.
- Never reintroduce a hardcoded boot disk (`/dev/sda`). Resolve from where
  `/boot` lives. This is the whole point vs. the old ArcoLinux hook.
- The install hook re-uses the existing `EFI/<id>` on the ESP; do not pass a
  fixed `--bootloader-id` that could create a duplicate UEFI entry.

## Promotion to production
Strip `-nemesis`: pkgname/url/conflicts → `kiro-bootloader-grub`, then add
to the ISO default package set. Conflicts/provides flip accordingly.
