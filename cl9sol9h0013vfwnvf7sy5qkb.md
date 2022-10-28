# Automatic LUKS Partition Unlock at Boot Time

My hard drive is encrypted using `cryptsetup` (with LUKS), and at boot time I am prompted to type my passphrase. This is perfectly normal, and this is a security layer I rely on when I'm at the office or in a train. However when I'm at home, I would gladly skip this step. (I'm already locked up in my apartment, the computer is safe.)
That's what I've achieved using a __USB stick__ that always __stays at home__. It's plugged in my display's USB hub, and it provides `cryptsetup` a valid deciphering key at boot time, without asking me. When the USB stick is not found I'm prompted for my passphrase, just as before.

I'm using **Archlinux** and I'll show you how to configure this.

# Prepare the USB stick
With your favorite partitioning tool, create a 1MiB `ext4` partition on the USB stick you picked. (This USB stick should always stay at the same place and do nothing but provide keyfiles, so don't pick your favorite one.)

Mount this partition to something like `/tmp/keyfiles` on your computer. On my computer the partition is `/dev/sda1`.
```bash
# Create a temporary mount point
mkdir /tmp/keyfiles

# Mount your USB stick's new partition
sudo mount /dev/sda1 /tmp/keyfiles
```

We're now using `dd` to generate a random 2kiB binary keyfile named `cryptsetup_keyfile.bin`, directly into the USB stick. We're also making it accessible only by `root` (permissions 600).
```bash
# Generate a random keyfile
$ sudo dd bs=512 count=4 if=/dev/random of=/tmp/keyfiles/cryptsetup_keyfile.bin iflag=fullblock
4+0 records in
4+0 records out
2048 bytes (2.0 kB, 2.0 KiB) copied, 0.000293115 s, 7.0 MB/s

# Limit permissions to root
$ sudo chmod 600 /tmp/keyfiles/new_key.bin
```
You're done with the key.

#  Improve the init ramdisk
For the init ramdisk to be able to read your USB stick at boot time you have to add a module into its configuration. As `root`, edit `/etc/mkinitcpio.conf` and add the module `ext4` into the `MODULES` list. On my computer the list was empty.
```vim
MODULES=(ext4)
```
Save the file and recompile the init ramdisk using `sudo mkinitcpio -P`.

# Edit Archlinux entrypoint
To tell the init ramdisk it has to look for a keyfile into a USB stick, you have to edit Archlinux's loader configuration. As `root`, edit `/boot/loader/entries/arch.conf` (it may be named differently on your end, but it's probably the only entry). Between the existing `options` and `cryptdevice=` statements, insert the following:
```txt
cryptkey=UUID=b0a8cae9-366e-4e32-a6db-21d3724c773e:ext4:/cryptsetup_keyfile.bin
```
This tells the system that a key named `cryptsetup_keyfile.bin` can be found on a device identified by its UUID, in an `ex4` filesystem. The UUID of your USB stick partition can be found from `lsblk -f`.

Here is my complete `arch.conf` file, just in case.
```txt
title Arch Linux de Luc
linux /vmlinuz-linux
initrd /initramfs-linux.img
options cryptkey=UUID=b0a8cae9-366e-4e32-a6db-21d3724c773e:ext4:/cryptsetup_keyfile.bin cryptdevice=UUID=c504d83c-1ed7-42e6-a566-70f2b8ee1b13:cryptlvm root=/dev/vg/root quiet
```

# Register the key with cryptsetup
This is the last step. You need to register your new key on your hard drive's LUKS headers so your keyfile is valid (at this point it's a purposeless keyfile). LUKS supports having several keys, so we're using `luksAddKey` assuming you have slots left.

Type the following command with __your own__ hard drive's main partition (mine  is `/dev/nvme1n1p2`).
```bash
sudo cryptsetup luksAddKey /dev/nvme1n1p2 /tmp/keyfiles/cryptsetup_keyfile.bin
```

That's it! Starting from now your computer will seek your USB stick and try to unlock your hard drive all by itself. If it fails to, it will fall back on prompting you to type your passphrase. Hurray!