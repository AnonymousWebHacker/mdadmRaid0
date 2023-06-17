# mdadmRaid0
## how to add a new disk to a linear RAID0 and increase space and performance .

In this example, I'm going to create a RAID 0 with 3 drives, populate it with data, and then add a new drive to the array.

### Create Raid
Type: RAID0 (level=0)

`mdadm --create /dev/md0 --verbose --level=0 --raid-devices=3 --chunk=256 /dev/sda /dev/sdb /dev/sdc`

### Formate to EXT4
`mkfs.ext4 /dev/md0`

### Mount MD0 and copy data
1 - `mount /dev/md0 /tmp/data`

2 - Copy eny files (videos, music ..etc)

3 - `umount /opt/raid`

### See if you have information
md0 has 36% of use
```
df -h
S.ficheros     Tamaño Usados  Disp Uso% Montado en
/dev/md0         4,9G   1,7G  3,0G  36% /opt/raid
```

### Checking disks on RAID
```
cat /proc/mdstat
Personalities : [raid0] [linear] [multipath] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid0 sdc[2] sdb[1] sda[0]
      5232640 blocks super 1.2 512k chunks
```
If you notice, you have 3 disks and a RAID0



## ADD OTHER DISC 

### Prepare the new disk
`mkfs.ext4 /dev/sdd`

If I try to add the disk now as usual, it will give the following error
```
mdadm --add /dev/md0 /dev/sdd
mdadm: add new device failed for /dev/sdd as 3: Invalid argument
```

### how fix?
`mdadm --grow /dev/md0 --verbose --level=0 --raid-devices=4 --add /dev/sdd`

You cannot miss the `--level=0` option, because it converts RADI0 to RAID4.
`--raid-devices=4` , is specifying that the array is no longer 3 disks, it is 4 disks

### Check New Add Disk
#### Is migrating, no power off
```
cat /proc/mdstat
Personalities : [raid0] [linear] [multipath] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid4 sdd[3] sdc[2] sdb[1] sda[0]
      5232640 blocks super 1.2 level 4, 512k chunk, algorithm 5 [7/6] [UUUUU__]
      [=>...................]  reshape =  6.4% (67072/1046528) finish=3.4min speed=4790K/sec

```
#### Note: 
1 - You will notice that you now have a raid4 , but it is because of the migration process, once it is done, it returns to its RAID0 state.
2 - The copy process is at 6.4%, you should not turn off the cp or do anything else, until it reaches 100%

 ```cat /proc/mdstat
Personalities : [raid0] [linear] [multipath] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid0 sdd[3] sdc[2] sdb[1] sda[0]
      5232640 blocks super 1.2 512k chunks
```

### Update RAID Size
Once the copy is finished, the file system in linux will still see the same disk size. If your raid is 3 x 1TB drives, you will still see 3TB instead of 4TB with the new drive added
1 - mount de raid again
`mount /dev/md0 /tmp/data`
2 - Resize `resize2fs /dev/md0`

this is All :)

Bibliography: https://tufora.com/tutorials/linux/general/resize-raid-0-array-adding-new-disks
