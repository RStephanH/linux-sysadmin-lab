## Environment
- Main OS 
- Distro : Cachy Os
- Package manager : pacman (used)/ paru
- Disk : 238 GB ssd 
- Number of partition : 2
- File system : BTRFS
![[disk_partition.png]]
## Situation
- The root partition is full 
- Can't upgrade os anymore

## Goal
- Clean junk data 
- Clean old snappshots
## Checking 
- We need to check the available and used size on the disk in order we can use this command below  :
```bash
$ lsblk (checking the all partition state)
$ df
$ duf (modern alternative of df)
$ du
$ ncdu (modern alternative of du)
```
- Execute this command by adding the / as parameter and check which directory is has the greatest size.
## New command 
```bash 
snapper 
```
> [!faq] What is snapper ?

- The command i used during the operation is : 
```
$ sudo snapper list # To list all snapshots that remain on the disk
#Check the old range snapshots and remove it 
$ sudo snapper delete 170-215 # To remove the range of snapshot 
$ sudo snapper delete --sync range 170-215 # to force it a little bit and has the space 
```

