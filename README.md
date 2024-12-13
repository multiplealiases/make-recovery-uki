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

### Building a system

Make a recovery system that works for the machine I'm typing this on.

> [!TIP]
> For your machine, examine
> [the Alpine Linux package index](https://pkgs.alpinelinux.org/packages?name=linux-firmware-*&branch=v3.21&repo=&arch=x86_64&origin=&flagged=&maintainer=)
> and the output of `dmesg | grep -i firmware`.

```
# amdgpu: AMD iGPU
# amdtee: probably relevant
# other: Intel WiFi card
sudo ./make-recovery-uki -o worksforme \
--packages 'linux-firmware-amdgpu linux-firmware-amdtee linux-firmware-other'
```

For me, this produces a ~150 MB UKI called `worksforme.efi`
as well as a corresponding kernel/initramfs pair `worksforme`/`worksforme.img`

If you use Nvidia, pass `-m 'nouveau'`.

### Alternative: building a universal system

If your ESP is large enough (1 GB or larger), it may be more convenient to
build a recovery system that contains all available firmware and modules.

```
sudo ./make-recovery-uki -o big \
--packages 'linux-firmware' \
--modules '.*'
```

This works out to a ~600 MiB UKI; large, but much more broadly compatible.

### Secure Boot

You can use `sbsign` to sign the UKI
using the same key and certificate you use to sign your kernel.

```console
$ sudo sbsign --key /etc/mok.key --cert /etc/mok.crt \
  recovery.efi --output recovery.signed.efi
```

I have only tested this on a machine
using the default Microsoft set of Secure Boot keys,
but a careful reading of
[Linux kernel documentation for module signing](https://www.kernel.org/doc/html/latest/admin-guide/module-signing.html)
suggests that this approach will work just fine even
if you enrolled your own PK, KEK, and db keys.

## Testing

> [!WARNING]
> Missing firmware can significantly reduce the recovery system's usefulness.
> Please test before you actually need it.

> [!NOTE]
> Consider adding a timeout to your bootloader before testing,
> as it might default to the new UKI.

Boot the recovery system using your bootloader of choice;
systemd-boot and rEFInd will autodetect UKIs.

Once you're booted (hopefully!), log in as root (no password).
If you find the default TTY font too small, use `setfont -d`
to double the font size.

If you have an Ethernet connection,
no further network configuration should be required.

If you have a WiFi card, run `setup-interfaces`, then
run `rc-service networking restart`.
Assuming you have the RAM to store it in,
this now lets you install additional packages,
in case the default package set isn't enough.

Now check if you can mount your filesystems.
A directory called /sysroot has been provided for convenience.

Shut down using `openrc-shutdown -p now`.

## Limitations

* ~~No Secure Boot support~~

* No GUI

* Does not print status information

* Assumes x86_64 (testing would be appreciated;
  I don't have aarch64 hardware that boots UKIs)

* Dependent on Bash and coreutils

* Hardcoded to use zstd for compression

* Kinda a mess internally

* Creates both a kernel/initramfs pair and a UKI,
  regardless of what you need

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
