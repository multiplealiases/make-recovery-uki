# make-recovery-uki

Assemble a recovery system out of Alpine Linux parts,
and cram it into an kernel/initramfs pair and a UKI.

## Usage

Make a recovery system that will probably not work as intended
(no display, no WiFi, etc) due to missing firmware:

```
sudo ./make-recovery-uki -o nonfunctional
```

While the script tries to include every loaded module on your system
and defaults to a reasonable package set for recovering a system,
I haven't yet figured out how to include firmware automatically.

Make a recovery system that works for this machine specifically.

> [!TIP]
> For your machine, examine
> [the Alpine Linux package index](https://pkgs.alpinelinux.org/packages?name=linux-firmware-*&branch=v3.21&repo=&arch=x86_64&origin=&flagged=&maintainer=).


```
# amdgpu: AMD iGPU
# amdtee: probably relevant
# other: Intel WiFi card
sudo ./make-recovery-uki -o worksforme \
--packages 'linux-firmware-amdgpu linux-firmware-amdtee linux-firmware-other'
```

For me, this produces a ~150 MB UKI.
File size may vary wildly depending on what firmware you include,
up to ~600 MiB in the worst case.

If you use Nvidia, pass `-m 'nouveau'`.

## Testing

> [!WARNING]
> Missing firmware can significantly reduce the recovery system's usefulness.
> Please test before you actually need it.

> [!NOTE]
> Consider adding a timeout to your bootloader before testing,
> as it might default to the new UKI.

Boot this UKI using your bootloader of choice;
systemd-boot and rEFInd will autodetect UKIs.

Once you're booted (hopefully!), log in as root (no password).
If you find the default TTY font too small, use `setfont -d`
to double the font size.

If you have an Ethernet connection,
no further network configuration should be required.

If you have a WiFi connection, run `setup-interfaces` to get connected.
Assuming you have the RAM to store it in,
this now lets you install additional packages,
in case the default package set isn't enough.

Now check if you can mount your filesystems.
A directory called /sysroot has been provided for convenience.

Shut down using `openrc-shutdown -p`.

## Limitations

* No Secure Boot support
  (you can `sbsign` the UKI, but the modules aren't signed)

* No GUI

* Does not print status information

* Assumes x86_64 (testing would be appreciated;
  I don't have aarch64 hardware that boots UKIs)

* Dependent on Bash and coreutils

* Hardcoded to use zstd for compression

* Kinda a mess internally

## Dependencies

* Bash and coreutils

* A suitable EFI stub (`linuxx64.efi.stub`)

* binutils (`objcopy`): making the UKI

* `curl`: downloading files

* squashfs-tools (`unsquashfs`): unpacking the modloop

* `tar`: dependency of [`alpine-make-rootfs`](https://github.com/alpinelinux/alpine-make-rootfs)

* `cpio`: makes the initramfs

* `zstd`: compresses the initramfs

## Inspirations

The concept is "heavily inspired" by the blog post
[frood, an Alpine initramfs NAS](https://words.filippo.io/dispatches/frood/).

[Archboot](https://archboot.com/): while I enjoy the concept, its UKIs are
often too large to practically include in an ESP,
negating its applicability as a recovery system;
the last line of defense before you need to plug in a USB stick.

[Arch Linux Wiki](https://wiki.archlinux.org/title/Unified_kernel_image#Manually)
invocations for generating a UKI, taken almost verbatim.

## Licensing

This script is licensed under MIT.
