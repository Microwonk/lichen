# lichen-vm

A QEMU harness for running lichen against a throwaway disk.

The VM boots the AerynOS live ISO under UEFI (OVMF) with one extra virtio
disk that is already laid out the way the installer expects:

| # | Type                | Size   | Filesystem |
|---|---------------------|--------|------------|
| 1 | EFI System          | 512MiB | FAT32      |
| 2 | Linux extended boot | 4GiB   | FAT32      |
| 3 | Linux filesystem    | rest   | XFS        |

The pristine base image is created once without root privileges. Each run
writes to a qcow2 overlay on top of it, `reset` returns the disk to its
initial state.

The repository's `target/` directory is shared read-only into the guest via
9p, so the VM does not have to be rebuilt every time the binary is compiled.

## Requirements

- `qemu-system-x86_64` with GTK display support and 9p (`virtfs`)
- OVMF UEFI firmware
- `curl`, `bsdtar`, `mtools`, `sfdisk`, `mkfs.fat`, `mkfs.xfs`

## Usage

See the scripts help output by running it.

In the guest, open a terminal and run:

```bash
sudo mkdir -p /mnt/lichen
sudo mount -t 9p -o trans=virtio,version=9p2000.L lichen /mnt/lichen
sudo /mnt/lichen/debug/lichen_cli
```

After the installation finished, power off the guest and boot the result:

```bash
tools/vm/lichen-vm installed
```

Start over with a clean disk:

```bash
tools/vm/lichen-vm reset
```

`live --serial` boots the live kernel directly with its console on the
terminal instead of a window and skips the desktop. Log in as `live` or `root`.
Quit with `Ctrl-a x`. Any arguments after the command options are passed through to QEMU
