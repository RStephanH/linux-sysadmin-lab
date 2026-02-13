# Password reset on NixOs vm all in one partition

## Goal
- [ ] Obtain the access to the system 
- [ ] Change the password of a local user 
## Intro 
Theoricaly, we have 2 methods to reset a password of a local account :
- Method 1 : Boot into Single-User / Emergency Mode / Rescue Mode
- Method 2 : Using a Live ISO (Universal Recovery)

### Method 1 : Boot into Single-User / Emergency Mode / Rescue Mode 
- So when the GRUB appears, stop the count down and press e to enter in edit boot entry mode 
- Look for the kernel line : in my case i was looking for this line `linux /nix/store/xxxxxxxxx`
- Add at end of line `single` or `systemd.unit=rescue.target` after that we 

![[edit boot entry mode.png]]
- need to press Ctrl + x or F10 then then Os will reboot
- When the system finished to boot a prompt that ask root password 
![[root password prompt for password reset.png]]

> [!fail]
> I don't have the root password

>[!info]
>In case we know the root password and succeed to login on the root shell 

- We just use the `passwd` command with target user
```bash
passwd user
```
- Then make sure that the modification is written on the storage
```bash
sync
```
- Finally reboot the system
```bash
reboot
```
## Method 2 : Using a live ISO 
This is the safest fallback when single-user mode is blocked (e.g. root password required even in rescue, or custom security settings).
- First download the latest iso of the os (graphical or minimal)
- Attach / mount the ISO on the vm , we can do that by creating  a new SATA CDROM storage and select the path of the Os iso
- Boot the vm from that iso then open a terminal
- Find the root  partition  with the command `lsblk` or `fdisk `
```bash
lsblk -f 
#or
sudo fdisk -l

```
>[!tip]
>- /dev/vda1 or /dev/sda1 (most common in VMs)
>- Look for ext4 / btrfs partition that is **not** the live ISO

![[identify the root partition.png]]
- Mount the root partition to `/mnt` :
```bash
sudo mount /dev/vda1 /mnt
```

>[!warning]
>Use other alternative with btrfs file with subvolumes

- Bind mount the essential virtual filesystems :
```bash
sudo mount --bind /dev /mnt/dev 
sudo mount --bind /proc /mnt/proc 
sudo mount --bind /sys /mnt/sys 
sudo mount --bind /run /mnt/run 
# optional but often helpful
sudo mount --bind /dev/pts /mnt/dev/pts
```

- Enter the installed system (magic NixOS command):
>[!fail]
>This command doesn't work in my case
>```bash 
>sudo nixos-enter
>```

>[!error]
>It's broke the live system and almost all the command is not found so if you did it, you need to reboot again from live iso and restart the mount operation again.

>[!tip]
> Instead enter to the chroot manually

- So after mounting the root partition, It's better to mount the other fs with these following commands (I'm talking about --rbind option): 

```bash
# Bind-mount the essentials (very important!) 
mount --rbind /dev /mnt/dev 
mount --rbind /proc /mnt/proc 
mount --rbind /sys /mnt/sys 
mount --rbind /run /mnt/run
 # Optional but often fixes weird issues (I didn't run this):
mount --make-rslave /mnt/dev 
mount --make-rslave /mnt/proc 
mount --make-rslave /mnt/sys 
mount --make-rslave /mnt/run
```

-  Then change root and run bash with the command : 
  ```bash
  chroot /mnt /run/current-system/sw/bin/bash #(failed in my case)
  # so let's try this one
  chroot /mnt /nix/var/nix/profiles/system/sw/bin/bash #(I succeed the chroot but something went wrong with the path so I can't use any command)
  ```
  - So it's still fail then let's unmount it :
  ```bash
  unmount -R /mnt/dev
  unmount -R /mnt/proc
  unmount -R /mnt/sys
  unmount -R /mnt/run
 
  ```
>[!fail]
> After try to run the first command I already have an error that said the target is busy so let's figure out what wrong again

>[!tip]
>Just use the lazy unmount in our case with the `-l ` option of unmount.
>>[!warning]
>>Be carefull using the lazy unmount in production server

- Let's reboot again and restart the method 2 from the beginning 🥲
-  Let's remount the root partion 
```bash
sudo mount /dev/vda1 /mnt
```
- Use the `nixos-enter` command again
>[!success]
> This time it works perfectly with this command `sudo nixos-enter --root /mnt` after mounting only the root partition and nothing else cause the `nixos-enter` command will automatically mount and set up every thing that will needed.

![[chroot nixos.png]]
- After that we just need to change the password of the user 
```bash
passwd user
```
- Change the root password too
```bash
passwd
```
- Exit the chroot with `exit` command 
- Unmount the root partition with : 
```bash
sudo umount -R /mnt
```
![[exit chroot.png]]
- Finally reboot and remove the live ISO.
