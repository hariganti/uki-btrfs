# uki-btrfs
Add BTRFS-snapshot-based boot options to a multi-profile UKI

## Description
Automate the process of generating a multi-profile UKI for a UEFI boot manager to directly boot into a BTRFS subvolume, such as the standard root (typically "@") or a snapshot (ex. "timeshift-btrfs/snapshots/<DATE_TIME>/@").

## Why?
Modern UEFI boot managers can remove the need for a separate bootloader, such as systemd-boot or GRUB, which removes one possible point of failure as well as simplifying the bootchain. Bootloaders offer a level of convenience in exchange for possible vulnerabilities and slightly lengthening the boot process. Previously, this convenience was worthwhile for the ability to boot into separate OS's, older kernel versions, or even filesystem snapshots, such multi-profile UKIs can enable all of those directly.

UKIs (Unified Kernel Images) combine the kernel, `initramfs`, and command line into a single ELF that the UEFI boot manager can load directly. When combined with secure boot and appropriate TPM/PCR policies, this can help to enhance overall security by providing a chain of trust in the boot process (assuming you trust the underlying firmware and UEFI). The embedded `initramfs` and command line become implicitly signed since the entire UKI is signed, so any changes will be measured by the UEFI, preventing any automatic unsealing of secrets from the TPM. As a result, tampering with the boot process becomes much more difficult, so Full Drive Encryption becomes more effective.

[A UKI can contain profiles](https://uapi-group.org/specifications/specs/unified_kernel_image/) which are selected with a command line argument indicating the profile to load. These profiles can share elements, such as kernels and initrd, or have unique elements, like the command line. By leveraging multiple profiles, snapshots can be booted with just a change to the embedded command line in a profile (several bytes) instead of needing an entirely separate UKI (several hundred megabytes). This sharing of common resources makes it much more feasible to have many snapshots available, similarly to GRUB menuentries when using a tool like [`grub-btrfsd`](https://github.com/Antynea/grub-btrfs), but selected through the UEFI boot manager rather than a bootloader.

In my limited testing, booting a UKI was a couple seconds--almost 10%--quicker to load than a bootloader-based process, with tighter control over the kernel, `initramfs`, and command line.

## Features
None yet!

## Dependencies
### Required
BTRFS
Timeshift
Python 3+
`systemd-ukify`
### Optional
`sbsigntools`
`sbctl`
`efibootmgr`

## Installation
Don't worry about it yet--it's not ready

## TODO
- [ ] Script to generate command lines from config and snapshots
  - [ ] Use provided command line or file, or concatenate `/etc/cmdline.d/*.conf` (with filters)
  - [ ] Get snapshot subvolume paths
    - [ ] Filter by date based on config (ex. snapshots={1, 7, 30} for 1 day prior, 7 days prior, and 30 days prior)
- [ ] Script to generate UKI profiles from `initramfs` (from config) and command lines
- [ ] Script to assemble UKI from profiles
- [ ] Script to install UKI to EFI partition
  - [ ] Rename previous UKI based on config (ex. keep=2 => arch.efi, arch.baka1.efi, arch.baka2.efi)
- [ ] `setup` capability
  - [ ] `chmod -x /usr/lib/initcpio/sbctl`
  - [ ] Alias `mkuki` as `mkinitcpio` for a wrapper function
  - [ ] `sbctl sign --save /boot/EFI/Linux/arch.efi`
- [ ] Drop-in hooks/replacements for hooks, and systemd services
  - [ ] `pacman` hook (`/etc/pacman.d/hooks`)
  - [ ] [systemd service](https://github.com/Antynea/grub-btrfs/blob/master/grub-btrfsd.service)
     
## Acknowledgements
The aforementioned [`grub-btrfsd`](https://github.com/Antynea/grub-btrfs) for being the inspiration behind this endeavor
