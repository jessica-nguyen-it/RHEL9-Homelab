# Week 6 - Configuring Local Storage (Partitions, RAID, LVM, SWAP, & Encryption)

I'll admit, when this topic first came up during my summer internship, I was pretty intimidated... filesystems, partitions, and all the different acronyms hit me hard. But revisiting it now while preparing for the RHCSA, the concepts feel much more natural. 

In fact, if there’s any topic I feel most confident about for the exam, it’s probably this one!

So, yet again, here are my notes for the week:


## Managing physical storage partitions

1. Check current layout  
   ```bash
   lsblk
   ```
   → This lets you see disks and partitions.
   → / contains everything except the directories where other filesystems are mounted.

3. Open partition tool  
   ```bash
   sudo cfdisk /dev/sdb
   ```
   → Interactive version of `fdisk` for partition creation/deletion/resizing/typing. I was surprised at how intuitive it was to use!

4. Make it persistent

    After formatting and mounting, add an entry in /etc/fstab so the partitions you made auto‑mount at boot. For example:

   ```bash
   /dev/sdb1   /data   xfs   defaults   0 0
   ```

## Configuring and managing swap space

1. Check existing swap

   - Command: `swapon --show`
   -  Purpose: See what swap areas are currently active.
   -  Extra check: `lsblk` to list all partitions and spot unused ones.

2. Prepare a swap area

    Option A: Partition

     - Use `mkswap /dev/vdb3` to format a partition as swap.

    Option B: File

      - Create with `dd if=/dev/zero of=/swap bs=1M count=2048` (for 2 GB).
      -   Secure it: `chmod 600 /swap`.
      -   Format: `mkswap /swap`.

3. Activate swap

   - Command: `swapon /dev/vdb3` or `swapon /swap`

   - Verify: `swapon --show` or `free -h` to confirm it’s active.

4. Make it persistent

   - Edit `/etc/fstab` and add an entry for the swap partition or file. For example:

   ```bash
	/dev/vdb3 none swap sw 0 0
	/swap     none swap sw 0 0
	```



## Managing and configuring LVM storage

LVM was one of the trickiest topics for me to memorize. If you’re like me, having a simple workflow makes it much easier to recall: PV → VG → LV → FS → Mount. 

Thinking of it as this chain helps the concepts click into place and keeps the commands organized in your head. Also remember to use man pages, specifically: `man lvm`

1. Physical Volumes (PV) → Mark raw disks or partitions as usable by LVM.

   - Command: `pvcreate /dev/sdc`
   -  Additionally: `pvs`, `pvremove`
   - Big picture: “This disk is now part of my Lego set.”

2. Volume Groups (VG) → Pool multiple PVs into one big storage bucket.

   - Command: `vgcreate my_volume /dev/sdc /dev/sdd`
	- Additionally: `vgextend`, `vgreduce`, `vgs`
	- Big picture: “I’ve combined two 5GB disks into one 10GB pool.”

3. Logical Volumes (LV) → Slice the VG into usable partitions.

   - Command: `lvcreate --size 2G --name partition1 my_volume`
	- Additionally: `lvs`, `lvdisplay` 
   - Big picture: “From my 10GB pool, I carved out a 2GB partition.”

4. Filesystem + Mounting → Format the LV with ext4/XFS and mount it.

    - Command: `mkfs.xfs /dev/my_volume/partition1`
   - Big picture: “Now this logical slice behaves like a normal disk partition.”

5. Resizing → Grow/shrink LVs as needs change.

	- Expanding one partition to take 100% of remaining capacity: `lvresize --extents 100%VG my_volume/partition1`
	- Shrinking back: `lvresize --resizefs --size 3G my_volume/partition1`
   - Big picture: “I can expand my partition without reinstalling or repartitioning the whole disk.”

6. Persistence → Add LV to /etc/fstab so it mounts at boot.
 ```bash
/dev/my_volume/partition1   /lvdata   xfs   defaults   0 0

```

## Configuring Encrypted Storage

Encryption protects sensitive data from unauthorized access, especially if the disk is stolen. The tool we'll use for this is `cryptsetup`.
 
This week, I went over two main methods: **Plain** (simple password) vs **LUKS** (preferred: metadata + key management).  

#### 1. Plain Mode Workflow  
```bash
sudo cryptsetup --verify-passphrase open --type plain /dev/vde mysecuredisk
sudo mkfs.xfs /dev/mapper/mysecuredisk
sudo mount /dev/mapper/mysecuredisk /mnt
# Close when done
sudo umount /mnt
sudo cryptsetup close mysecuredisk
```

#### 2. LUKS Workflow  
```bash
sudo cryptsetup luksFormat /dev/vde      # confirm with YES
sudo cryptsetup open /dev/vde mysecuredisk
sudo mkfs.xfs /dev/mapper/mysecuredisk
sudo mount /dev/mapper/mysecuredisk /mnt
# Close when done
sudo umount /mnt
sudo cryptsetup close mysecuredisk
```

Key Management 
```bash
sudo cryptsetup luksChangeKey /dev/vde
```

Peristance 

```bash
# Add an entry in /etc/crypttab to auto‑unlock at boot:
mysecuredisk   /dev/vde   none   luks

# Then add the mapped device to /etc/fstab so it mounts automatically:
/dev/mapper/mysecuredisk   /securedata   xfs   defaults   0 0
```

## Working with RAID

RAID (Redundant Array of Independent Disks) lets you combine multiple disks into one logical unit for capacity or redundancy. Managed in Linux with the `mdadm` utility. 

#### 1. Creating a RAID 0 Array

```Bash
sudo mdadm --create /dev/md0 --level=0 --raid-devices=3 /dev/vdc /dev/vdd /dev/vde
sudo mkfs.ext4 /dev/md0
```  

#### 2. Stopping your RAID Array

```Bash
sudo mdadm --stop /dev/md0
sudo mdadm --zero-superblock /dev/vdc /dev/vdd /dev/vde

```  

#### 3. Creating a RAID 1 Array with a spare

```Bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/vdc /dev/vdd --spare-devices=1 /dev/vde
```  

#### 4. Expanding a RAID 1 Array

```Bash
cat /proc/mdstat   # check status
sudo mdadm --add /dev/md0 /dev/vdf   # add new disk

```  

#### 5. Persistence

As always, make sure to save your configurations so that they can be reassembled at boot!

```Bash
sudo mdadm --detail --scan >> /etc/mdadm.conf

# Add RAID device to /etc/fstab for auto‑mount:
/dev/md0   /raiddata   ext4   defaults   0 0

```  

#

