# kiro-bootloader-grub-nemesis

Pacman hooks that keep a **GRUB** Kiro install bootable across upgrades.

> Nemesis (development) variant. Proven here first, then promoted to the
> production package `kiro-bootloader-grub`.

## What it does

| File | Trigger | Action |
|---|---|---|
| `kiro-grub-install.hook` | `grub` package upgrade/install | runs `/usr/bin/kiro-grub-install` — re-flashes the GRUB **boot image** |
| `kiro-grub-mkconfig.hook` | `grub`, kernel or microcode change | runs `grub-mkconfig -o /boot/grub/grub.cfg` — refreshes the **menu** |

### Why the install hook exists

A `grub` package upgrade refreshes the modules in `/boot/grub` but **not**
the boot image already written to disk (BIOS `core.img` / UEFI
`grubx64.efi`). The mismatch is the classic Arch
`error: symbol 'grub_*' not found` brick. Re-running `grub-install` on every
`grub` upgrade prevents it.

### Disk targeting — never assumes `/dev/sda`

`/usr/bin/kiro-grub-install` discovers the boot disk instead of guessing:

- **UEFI** → `grub-install --target=x86_64-efi --efi-directory=/boot/efi`,
  re-using the existing `EFI/<id>` so no duplicate boot entry is created.
- **BIOS** → finds the disk holding `/boot` via
  `grub-probe --target=device /boot` → `lsblk -no pkname`, so it resolves
  `sda` / `vda` / `sdb` / `nvme0n1` / `mmcblk0` correctly (RAID `/boot`
  installs to each parent disk).

## Inert on systemd-boot

Both hooks carry `Depends = grub`. Kiro's default install is systemd-boot,
where `grub` is not installed, so the hooks never fire. They activate only
on BIOS installs and on UEFI users who chose GRUB via the Calamares Tweak
Tool.

## Testing (BIOS, in a KVM/QEMU VM — disk is `vda`)

```bash
# inside a GRUB+BIOS Kiro VM
sudo kiro-grub-install            # should print: BIOS grub-install -> /dev/vda
sudo pacman -S grub               # re-flash fires via the hook; reboot must work
```

<!-- KIRO-FUNDING-FOOTER:START — managed by Kiro-HQ/cascade-readme-footer.sh -->
## Help fund Kiro

Everything I build here stays free and open — always. If Kiro or any of these
tools have ever saved you time or taught you something, a small monthly
contribution helps keep the work going. Donations target break-even, nothing
more — the core always stays free for everyone.

- GitHub Sponsors: https://github.com/sponsors/erikdubois
- Patreon: https://www.patreon.com/c/kiroproject
- YouTube memberships: https://www.youtube.com/@ErikDubois/join
- Ko-fi: https://ko-fi.com/erikdubois
- PayPal: https://www.paypal.me/erikdubois
<!-- KIRO-FUNDING-FOOTER:END -->
