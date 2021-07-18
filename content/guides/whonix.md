---
title: "Hacking Whonix onto Hyper-V"
date: 2021-06-05
tags:
- hyper-v
- privacy
- security
- whonix
---

* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
* [Installation](#installation)
    * [Extraction](#extraction)
    * [Conversion](#conversion)
    * [Networking](#networking)
        * [Create the internal switch](#create-the-internal-switch)
            * [Create the gateway](#create-the-gateway)
            * [Configure the network](#configure-the-network)
        * [Create the private switch](#create-the-private-switch)
    * [VM creation](#vm-creation)
        * [Gateway](#gateway)
        * [Workstation](#workstation)
        * [Hyper-V networking](#hyper-v-networking)
* [Generation 2 installation](#generation-2-installation)
    * [Converting to vhdx](#converting-to-vhdx)
    * [Install grub-efi](#install-grub-efi)
    * [Partitioning](#partitioning)
        * [Mount partitions](#mount-partitions)
        * [Load the efivars kernel module](#load-the-efivars-kernel-module)
        * [Finish the GRUB install](#finish-the-grub-install)
        * [Unmount and reboot](#unmount-and-reboot)
    * [Problems](#problems)

## Introduction

This guide aims to get the Whonix™ operating system running on Hyper-V. This guide does not expect Virtualbox specific features such as guest additions to work and does not make any effort to make them work.

## Prerequisites

* The latest [Whonix for VirtualBox](https://www.whonix.org/wiki/VirtualBox)
* The latest [VirtualBox for Windows](https://www.virtualbox.org/wiki/Downloads)
* `qemu-img`[is broken when trying to convert to vhdx](https://gitlab.com/qemu-project/qemu/-/issues/250) and is not supported.

It is strongly recommended to verify the authenticity of the downloaded files.

## Installation

### Extraction

Use `tar` or another archive utility of choice to extract the `.ova` archive.

This should produce two `.vmdk` virtual disks.

### Conversion

Convert the two virtual disks.

```
VBoxManage.exe clonehd source.vmdk target.vhd --format vhd
```

### Networking

#### Create the internal switch

Create an internal switch and [get the interface index.](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network#create-a-nat-virtual-network)

##### Create the gateway

```
New-NetIPAddress -IPAddress 10.0.2.2 -PrefixLength 24 -InterfaceIndex <ifIndex>
```

##### Configure the network

```
New-NetNat -Name Gateway -InternalIPInterfaceAddressPrefix 10.0.2.0/24
```

#### Create the private switch

This switch is used to connect the Gateway to Workstation.

```
New-VMSwitch -SwitchName Whonix -SwitchType Private
```

### VM creation 

There should be two Generation 1 virtual machines. Whonix does not have UEFI support and will not boot on Generation 2 VMs. 

UEFI setup so that Whonix will run in Generation 2 virtual machines is detailed [later](#generation-2-installation)

#### Gateway

Create a Generation 1 virtual machine and add your `whonix-xxxx-disk001.vhd` virtual disk.

Create two virtual network adapters, one connecting to the Gateway virtual switch, and one connecting to the Whonix virtual switch.

#### Workstation

Create a Generation 1 virtual machine and add your `whonix-xxxx-disk002.vhd` virtual disk.

Create a virtual network adapter connecting to the Whonix virtual switch.

#### Hyper-V networking

To get networking working on Whonix, you need to enable some [kernel modules](https://blog.jitdor.com/2020/02/08/enable-hyper-v-integration-services-for-your-ubuntu-guest-vms/) for Hyper-V integration.

```
sudo printf "hv_utils \nhv_vmbus \nhv_storvsc \nhv_blksvc \nhv_netvsc" >> /etc/initramfs-tools/modules
sudo update-initramfs -u
```

## Generation 2 installation

Do not attempt any of the steps below unless you are familiar with the implications of executing them wrong.

### Converting to vhdx

Generation 2 virtual machines only support `.vhdx` disks. You should convert the disks within the Hyper-V Manager UI or with the PowerShell cmdlet `Convert-VHD`.

### Install grub-efi

Boot up your existing Generation 1 virtual machine and install `grub-efi-amd64`.

```
sudo apt update
sudo apt install grub-efi-amd64
sudo apt purge grub-pc-bin
```

### Partitioning

Resize /dev/sda1 to create some free space for the EFi system partition with `cfdisk /dev/sda`.

Use `gdisk /dev/sda` to convert the partition table to GPT and create the ESP on `/dev/sda2`.

#### Format the EFI partition

```
sudo mkfs.fat -F 32 /dev/sda2
```

#### Mount partitions

```
sudo mount /dev/sda1 /mnt
sudo mkdir -p /mnt/boot/efi
sudo mount /dev/sda2 /mnt/boot/efi
for d in dev proc run sys; do sudo mount -B /$d /mnt/$d; done
```

#### Load the efivars kernel module

```
sudo modprobe efivars
```

#### Finish the GRUB install

```
sudo chroot /mnt
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

#### Unmount and reboot

```
sudo umount -R /mnt
sudo reboot
```

### Problems

There may be a problem with the grub config file.

```
sudo mount /dev/sda2 /boot/efi
sudo mv /boot/efi/EFI/GRUB/grubx64.efi /boot/efi/EFI/GRUB/grubx64.efi.bak
sudo cp /boot/grub/x86_64-efi/grub.efi /boot/efi/EFI/GRUB/grubx64.efi
```
