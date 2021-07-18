---
title: "Hacking Whonix onto Hyper-V"
date: 2021-06-05
tags:
- hyper-v
- privacy
- security
- whonix
---

Hi, disclaimer, this is not officially supported by Whonix so expect no support.

Whonix does not have UEFI support at the moment, so you will have to resort to this guide hacking UEFI together if you want to run Generation 2 virtual machines.

1. Download [Whonix](https://www.whonix.org/wiki/VirtualBox) and [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
	
	* `qemu-img`[is broken when trying to convert to vhdx](https://gitlab.com/qemu-project/qemu/-/issues/250).
	* Verify the authenticity of the downloaded files.

2. Extract the image with your archive utility of choice
	* Whonix packages their operating system in an `.ova` archive.
	* Once decompressed, two `.vmdk` files are produced.

3. Convert the `.vmdk` to `.vhd` 
	* Install Virtualbox if you have not already.
	* Convert both disks with
```
VBoxManage.exe clonehd source.vmdk target.vhd --format vhd
```

4. Set up Hyper-V virtual switches
	* [Create an internal switch for Whonix-Gateway and get the interface index.](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network#create-a-nat-virtual-network)
		* This network will be translated to the host network.

* Configure the gateway with 

```
New-NetIPAddress -IPAddress 10.0.2.2 -PrefixLength 24 -InterfaceIndex <ifIndex>
```

* Configure the NAT network with 

```
New-NetNat -Name "$Whonix" -InternalIPInterfaceAddressPrefix 10.0.2.0/24
```

* Create a private switch to facilitate communication between the Workstation and the Gateway

```
New-VMSwitch -SwitchName "$WhonixPriv" -SwitchType Private
```

5. Set up Whonix-Gateway
	* Create a Generation 1 virtual machine with the `whonix-xxxx-disk001.vmdk`.
	* Add a network adapter to the `$Whonix` switch.
	* Add a network adapter to the `$WhonixPriv` switch.

6. Set up Whonix-Workstation
	* Create a Generation 1 virtual machine with the` whonix-xxxx-disk002.vmdk`.
	* Add a network adapter to the `$WhonixPriv` switch.

7. Enable networking
	* To get networking working on Whonix, you need to enable some [kernel modules](https://blog.jitdor.com/2020/02/08/enable-hyper-v-integration-services-for-your-ubuntu-guest-vms/) for Hyper-V integration.
	* Run in both virtual machines
```
sudo printf "hv_utils \nhv_vmbus \nhv_storvsc \nhv_blksvc \nhv_netvsc" >> /etc/initramfs-tools/modules
sudo update-initramfs -u
```

8. Configure VMs for UEFI
	* This is Experimental
	* is Experimental
	* Experimental
	* Convert the `vhd` to `vhdx`.
	
* Install Grub EFI

```
sudo apt update
sudo apt install grub-efi-amd64
sudo apt purge grub-pc-bin
```

8a. Reboot into a live system

* Resize /dev/sda1 to create some free space for the EFI partition with `cfdisk`.
* Use `gdisk` to convert the partition table to GPT and create the EFI system partition on `/dev/sda2`.
* Format the EFI system partition

```
sudo mkfs.fat -F 32 /dev/sda2
```

* Mount partitions with

```
sudo mount /dev/sda1 /mnt
sudo mkdir -p /mnt/boot/efi
sudo mount /dev/sda2 /mnt/boot/efi
for d in dev proc run sys; do sudo mount -B /$d /mnt/$d; done
```
* Load the `efivars` kernel module

```
sudo modprobe efivars
```

* Chroot into the system and install GRUB

```
sudo chroot /mnt
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

* Unmount partitions

```
sudo umount -R /mnt
```

* Reboot

* If the GRUB config file won't load during boot run

```
sudo mount /dev/sda2 /boot/efi
sudo mv /boot/efi/EFI/GRUB/grubx64.efi /boot/efi/EFI/GRUB/grubx64.efi.bak
sudo cp /boot/grub/x86_64-efi/grub.efi /boot/efi/EFI/GRUB/grubx64.efi
```

9. Optional Step: Stop `whonixcheck` failing because of unsupported hypervisor

```
sudo sed -i 's/WHONIXCHECK_NO_EXIT_ON_UNSUPPORTED_VIRTUALIZER="0"/WHONIXCHECK_NO_EXIT_ON_UNSUPPORTED_VIRTUALIZER="1"/' /etc/whonix.d/30_whonixcheck_default.conf && sudo update-initramfs -u
```

10. Ask Patrick politely to make an Hyper-V image?
