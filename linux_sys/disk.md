# 挂在设置权限
```bash
sudo umount  /dev/sdb1
sudo mount -o rw,uid=1000,gid=1000,iocharset=utf8 /dev/sdb1 /media/sky/SSD
```

# 新解压工具

```bash
sudo apt install bsdtar  # 安装
bsdtar -xf ShapeNetCore.v1.zip -C /path/to/ spacious_target/
```

# 本地短点续传

```bash
# 第一次不用指定目标目录
rsync -avh --progress --partial   --partial-dir=.rsync-partial  /media/sky/835c4fa2-7263-4df6-880c-3dedf3db1024/Dataset/CV/ShapeNetCore.v2.zip   ./
```


# 日志存储位置是坏块

```bash
(legged-rsl) sky@night:~$ sudo dd if=/dev/zero of=/dev/sda bs=1M count=100 status=progress
100+0 records in
100+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 5.60209 s, 18.7 MB/s
(legged-rsl) sky@night:~$ sudo parted /dev/sda
GNU Parted 3.4
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt                                                      
(parted) mkpart primary ext4 0% 100%                                      
(parted) print                                                            
Model: Flash USB DisK 3.0 (scsi)
Disk /dev/sda: 8389GB
SSector size (logical/physical): 4096B/4096BPartition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  8389GB  8389GB  ext4         primary

(parted) quit                                                             
Information: You may need to update /etc/fstab.

(legged-rsl) sky@night:~$ sudo mkfs.ext4 -L "T7" /dev/sda1                
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2047999488 4k blocks and 256000000 inodes
Filesystem UUID: eeb650ee-5998-458e-8598-163875b9e3fb
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
	102400000, 214990848, 512000000, 550731776, 644972544, 1934917632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done       

(legged-rsl) sky@night:~$ sudo mount /dev/sda1 /mnt
mount: /mnt: wrong fs type, bad option, bad superblock on /dev/sda1, missing codepage or helper program, or other error.
(legged-rsl) sky@night:~$ sudo dmesg | tail -20
[77626.864616] EXT4-fs (sda1): Could not load journal inode
[77685.873886] EXT4-fs error (device sda1): ext4_find_extent:936: inode #8: comm pool-udisksd: pblk 1023967231 bad header/extent: invalid magic - magic 5355, entries 17218, max 14516(0), depth 0(0)
[77685.874660] EXT4-fs (sda1): Remounting filesystem read-only
[77685.874663] jbd2_journal_init_inode: Cannot locate journal superblock
[77685.874665] EXT4-fs (sda1): Could not load journal inode
[77909.848565] sd 0:0:0:0: [sda] Very big device. Trying to use READ CAPACITY(16).
[77951.082759] sd 0:0:0:0: [sda] Very big device. Trying to use READ CAPACITY(16).
[77951.091215] Alternate GPT is invalid, using primary GPT.
[77951.091223]  sda: sda1
[78103.622922] EXT4-fs error (device sda1): ext4_find_extent:936: inode #8: comm mount: pblk 1023967231 bad header/extent: invalid magic - magic 5355, entries 17218, max 30119(0), depth 0(0)
[78103.623903] jbd2_journal_init_inode: Cannot locate journal superblock
[78103.623907] EXT4-fs (sda1): Could not load journal inode
[78117.583705] EXT4-fs error (device sda1): ext4_find_extent:936: inode #8: comm pool-udisksd: pblk 1023967231 bad header/extent: invalid magic - magic 5355, entries 17218, max 30156(0), depth 0(0)
[78117.584182] EXT4-fs (sda1): Remounting filesystem read-only
[78117.584191] jbd2_journal_init_inode: Cannot locate journal superblock
[78117.584196] EXT4-fs (sda1): Could not load journal inode
[78125.487012] EXT4-fs error (device sda1): ext4_find_extent:936: inode #8: comm pool-udisksd: pblk 1023967231 bad header/extent: invalid magic - magic 5355, entries 17218, max 30193(0), depth 0(0)
[78125.487473] EXT4-fs (sda1): Remounting filesystem read-only
[78125.487480] jbd2_journal_init_inode: Cannot locate journal superblock
[78125.487484] EXT4-fs (sda1): Could not load journal inode
(legged-rsl) sky@night:~$ sudo mount -t ext4 -o sb=32768 /dev/sda1 /mnt
mount: /mnt: wrong fs type, bad option, bad superblock on /dev/sda1, missing codepage or helper program, or other error.
(legged-rsl) sky@night:~$ sudo mkfs.ext4 -L "T7" -O ^has_journal /dev/sda1
mke2fs 1.46.5 (30-Dec-2021)
/dev/sda1 contains a ext4 file system labelled 'T7'
	created on Sat Jan  3 12:06:35 2026
Proceed anyway? (y,N) y
Creating filesystem with 2047999488 4k blocks and 256000000 inodes
Filesystem UUID: c33b6669-f1e6-4c35-b6a3-2536a11552a7
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
	102400000, 214990848, 512000000, 550731776, 644972544, 1934917632

Allocating group tables: done                            
Writing inode tables: done                            
Writing superblocks and filesystem accounting information: done       

(legged-rsl) sky@night:~$ sudo mount /dev/sda1 /mnt
(legged-rsl) sky@night:~$ df -hT /mnt
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sda1      ext4  7.6T   53G  7.2T   1% /mnt
(legged-rsl) sky@night:~$ echo "硬件诊断测试通过 - $(date)" | sudo tee /mnt/test_read_write.txt
硬件诊断测试通过 - 2026年 01月 03日 星期六 12:12:13 CST
(legged-rsl) sky@night:~$ sudo cat /mnt/test_read_write.txt
硬件诊断测试通过 - 2026年 01月 03日 星期六 12:12:13 CST
(legged-rsl) sky@night:~$ ls -lh /mnt/test_read_write.txt
-rw-r--r-- 1 root root 70  1月  3 12:12 /mnt/test_read_write.txt
(legged-rsl) sky@night:~$ # 清理测试文件（如果不再需要）
(legged-rsl) sky@night:~$ sudo rm /mnt/test_read_write.txt
```
