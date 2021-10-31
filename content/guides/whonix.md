+++
title = "Hacking Whonix onto Hyper-V"
date = 2021-06-05
description = "A guide to get Whonix running on Hyper-V."
summary = "A guide to get Whonix running on Hyper-V."
+++

This guide aims to get the Whonix™ operating system running on Hyper-V. This
guide does not expect Virtualbox specific features such as guest additions to
work and does not expend any effort to make them work.

{{< table_of_contents >}}

## Prerequisites

### Acquire the necessary tools

*   [Whonix for VirtualBox](https://www.whonix.org/wiki/VirtualBox)
*   [VirtualBox for Windows](https://www.virtualbox.org/wiki/Downloads)

### Verify the signature

See https://www.whonix.org/wiki/Verify_the_Whonix_images

### Extract the archive

Use `tar` or another archive utility of choice to extract the `ova` archive,
producing two `vmdk` virtual disks.

### Convert the disks

Convert both virtual disks to `vhd`.

```powershell
VBoxManage.exe clonehd source.vmdk target.vhd --format vhd
```

## Virtual Machine Setup

### Set up networking

*   Create an internal switch named WhonixGateway and find its
    [interface index](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network#create-a-nat-virtual-network).
    *   Create the NAT gateway with the previously mentioned interface index

        ```powershell
        New-NetIPAddress -IPAddress 10.0.2.2 -PrefixLength 24 -InterfaceIndex <ifIndex>
        ```
    *   Configure the NAT Network's name and NAT subnet prefix

        ```powershell
        New-NetNat -Name Gateway -InternalIPInterfaceAddressPrefix 10.0.2.0/24
        ```
*   Create a private switch, used to connect Whonix-Gateway to Whonix-Workstation.

    ```powershell
    New-VMSwitch -SwitchName WhonixPrivate -SwitchType Private
    ```

### Create virtual machines

Whonix still uses GRUB's BIOS payload. It isn't UEFI compatible and doesn't run
in Hyper-V's "Gen 2" VMs, which bring various security improvements. However, 
you can attempt to run Whonix in "Gen 2" VMs here.

*   Create two virtual machines and one virtual disk (`whonix-xxxx-disk00x.vhd`)
    to each.
*   Create a virtual network adapter on the Whonix-Gateway VM connecting to the
    "WhonixGateway" virtual switch.
*   Create virtual network adapters on both VMs connecting to the "WhonixPrivate"
    virtual switch.

## Post-boot configuration

We need to load some kernel modules to get networking working on Whonix. See
[this link](https://blog.jitdor.com/2020/02/08/enable-hyper-v-integration-services-for-your-ubuntu-guest-vms/)
for more info.

```shell
printf "hv_utils \nhv_vmbus \nhv_storvsc \nhv_blksvc \nhv_netvsc" >> /etc/initramfs-tools/modules
update-initramfs -u
```

## Round 2: Working with EFI

Turns out EFI is relatively simple; install `grub-efi` and convert the partition
table to GPT. This next section is for the people who need it.

>   Generation 2 virtual machines only support `.vhdx` disks. You should convert
>   the disks within the Hyper-V Manager UI or with the PowerShell cmdlet
>   `Convert-VHD`.

### Install grub-efi

Boot up your existing Generation 1 virtual machine and install `grub-efi-amd64`.

```shell
apt update
apt install grub-efi-amd64
apt purge grub-pc-bin
```

### Partitioning

*   Resize /dev/sda1 to create some free space for the EFi system partition with `cfdisk /dev/sda`.
*   Use `gdisk /dev/sda` to convert the partition table to GPT and create the ESP on `/dev/sda2`.
*   Format the EFI partition

    ```shell
    mkfs.fat -F 32 /dev/sda2
    ```
*   Mount partitions

    ```shell
    mount /dev/sda1 /mnt
    mkdir -p /mnt/boot/efi
    mount /dev/sda2 /mnt/boot/efi
    for d in dev proc run sys; do sudo mount -B /$d /mnt/$d; done
    ```
*   Finish the GRUB install

    ```shell
    chroot /mnt
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=DEBIAN
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

*   Unmount and reboot

    ```shell
    umount -R /mnt
    reboot
    ```

## Secure Boot

Debian's `grub-efi` package comes with `shim` out of the box signed by Microsoft's third party UEFI CA; it is up to the user to enable it in VM settings.

Do keep in mind this is mainly theater, as userspace components are not verified.

## See also

*   [https://forums.whonix.org](https://forums.whonix.org)
*   some link to some hyper-v research
*   more links to hyper-v research