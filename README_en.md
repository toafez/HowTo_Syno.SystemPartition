English | [Deutsch](README.md) | [HowTo's: Table of contents](https://github.com/toafez/Tutorials/blob/main/README_en.md)

# HowTo: Analyze and clean up a overfilled system partition on a Synology NAS
[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Ftoafez%2FHowTo_Syno.SystemPartition%2Fblob%2Fmain%2FREADME_en.md&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

## What is this about?
The following guide describes how to analyse and clean up a overfilled **System Partition** on a **Synology NAS** to restore proper operation of the DSM.

## Introduction
Physical data storage devices such as hard disk drives (HDD), solid state drives (SSD), NVM Express (NVMe), etc., commonly referred to as hard disk drives, usually have one or more partitions for serving and storing data. 

In a Synology NAS, all the internal hard drives are basically divided into three partitions, namely a **System Partition**, a **SWAP Partition**, and a **Data Partition**. The system and SWAP partitions are mirrored as RAID 1 on each internal drive, while the data partition consists of storage pools and volumes where all user data and installed packages are stored. For more information, see ![What are drive partitions?](https://kb.synology.com/en-nz/DSM/tutorial/What_are_drive_partitions)

## The System Partition
The **system partition** typically has a **storage capacity of 2.3 GiByte (up to DSM 6) or 7.9 GiByte (from DSM 7)** and contains the actual operating system, i.e. the DiskStation Manager, all user, system and network settings as well as the system logs. Immediately after initial or reinstallation, depending on the total storage capacity, **about 25 to about 65 percent of the available space is used**. The remaining free space is usually sufficient for DSM to operate properly. 

## Causes of an overfilled system partition
Sometimes, however, data ends up on the system partition that does not belong there. This includes both conscious and unconscious actions, e.g. when user data and/or program settings are stored in the home directory of the superuser root, which is also located on the system partition, or when, for example, misdirected backup jobs or downloads are stored there. USB or SATA drives that are not properly disconnected, unpredictable system crashes, or a sudden power failure can also result in data fragments ending up on the system partition.

Up to a certain point, this does not affect the system stability of the DSM. However, once the amount of storage used reaches a certain critical mass, which is somewhere between 80 and 100% of the available storage capacity, the DSM will slowly begin to stumble and start to fail. For example, you will no longer be able to log in to the DSM, you will no longer be able to perform system or package updates, or you will no longer be able to run packages. You will often see error messages like

_**This operation cannot be performed. The network connection may be unstable or the system may be busy. Please try again later**_

## Analyzing the system partition 
Unfortunately, it is not always obvious what is causing an overcrowded system partition, so part of the solution to such problems may be to understand what actions were previously performed, what tasks were completed, and what changes or adjustments were made. This will make it easier to find possible culprits within the system partition.

A classic example would be an overfilled home directory of the superuser root, so you should definitely take a look at this directory if the disk space used seems disproportionately high. Since there are no storage pools or volumes on the system partition, any directory named /volumeUSB... should also make you sit up and take notice. Directories or files with known package names or conspicuous file names of previously downloaded programs or archives should also be scrutinized. 

As is often the case, there is no universal solution, only tips and hints to make the search easier. The first step should always be to display the contents of the system partition, and then work your way through the directory jungle from top to bottom, starting with the directory that takes up the most disk space, until you have found the culprit(s).

For all further steps, it is necessary to log in to the Synology NAS console as root via a terminal program using SSH. Synology describes how to do this in the instructions ![How can I sign in to DSM/SRM with root privilege via SSH?](https://kb.synology.com/en-id/DSM/tutorial/How_to_login_to_DSM_with_root_permission_via_SSH_Telnet) 

#### _Note: Text in uppercase letters within square brackets serves as a placeholder and must be replaced with your own text, while mixed text consisting of uppercase and lowercase letters, numbers, and special characters within square brackets can be used optionally. In any case, the square brackets must be removed when replacing placeholders or using an option._

## The `df` Program (**d**isk **f**ree)
The `df` program displays the size, used and free space of all mounted partitions. To output the displayed space in a human readable format, the `-h` or `-H` option is appended to the `df` command, depending on whether the output is in metric or binary form.

- _**Syntax:** df [OPTION] [FILE]_

   **_Options used for the df program (more options ![see manpage](https://manpages.debian.org/bookworm/coreutils/df.1.en.html))_**

   - **-h, --human-readable** (Specifies the storage capacity in a binary format in powers of 1024, which is easier for humans to read.)
      ```
      "K" ≙ "Kibibyte (KiB)" ≙ 1,024¹ ≙ 1,024 Bytes
      "M" ≙ "Mebibyte (MiB)" ≙ 1,024² ≙ 1,048,576 Bytes
      "G" ≙ "Gibibyte (GiB)" ≙ 1,024³ ≙ 1,073,741,824 Bytes
      "T" ≙ "Tebibyte (TiB)" ≙ 1,024⁴ ≙ 1,099,511,627,776 Bytes
      "P" ≙ "Pebibyte (PiB)" ≙ 1,024⁵ ≙ 1,125,899,906,842,620 Bytes
      ```
   - **-H, --si** (Indicates storage capacity in a metric format in powers of 1000, which is easier for humans to read.)
      ```
      "K" ≙ "Kilobyte (kB)" ≙ 1,000¹ ≙ 1,000 Bytes
      "M" ≙ "Megabyte (MB)" ≙ 1,000² ≙ 1,000,000 Bytes
      "G" ≙ "Gigabyte (GB)" ≙ 1,000³ ≙ 1,000,000,000 Bytes
      "T" ≙ "Terabyte (TB)" ≙ 1,000⁴ ≙ 1,000,000,000,000 Bytes
      "P" ≙ "Petabyte (PB)" ≙ 1,000⁵ ≙ 1,000,000,000,000,000 Bytes
      ```
  #### To view the status of the system partition, run the following command
  ```
  df -h
  ```
     This is what the output of a "healthy" system partition would look like...

  ```
  root@SynologyNAS:~# df -h
  Filesystem              Size  Used Avail Use% Mounted on
  /dev/md0                2.3G  1.5G  773M  66% /
  devtmpfs                3.8G     0  3.8G   0% /dev
  tmpfs                   3.9G  240K  3.9G   1% /dev/shm
  tmpfs                   3.9G   46M  3.8G   2% /run
  tmpfs                   3.9G     0  3.9G   0% /sys/fs/cgroup
  tmpfs                   3.9G   30M  3.8G   1% /tmp
  /dev/loop0               27M  770K   24M   4% /tmp/SynologyAuthService
  Additional storage pools or volumes...
  ...
  ..
  .
  ```
   Of particular interest is the first line, which shows the total capacity, the used and free space, and the used space as a percentage of the system partition (mounted on `/`). The leading pathname of the system partition (/dev/md0) may vary.

  ```
  /dev/md0                2.3G  1.5G  773M  66% /
  ```
    With the introduction of DiskStation Manager 7 (DSM), the system partition was increased to 7.9G (GiByte) when DSM was reinstalled at some point. The result of a "healthy" system partition would look like this.

  ```
  root@SynologyNAS:~# df -h
  Filesystem         Size  Used Avail Use% Mounted on
  /dev/md0           7.9G  2.0G  5.8G  26% /
  devtmpfs           8.7G     0  8.7G   0% /dev
  tmpfs              8.8G  244K  8.8G   1% /dev/shm
  tmpfs              8.8G   38M  8.7G   1% /run
  tmpfs              8.8G     0  8.8G   0% /sys/fs/cgroup
  tmpfs              8.8G   29M  8.7G   1% /tmp
  /dev/loop0          27M  767K   24M   4% /tmp/SynologyAuthService
  Additional storage pools or volumes...
  ...
  ..
  .
  ```

## The `du` program (**d**isk **u**se) 
The `du` program displays the used disk space of files or, if a directory is specified, recursively the used disk space of all files contained in it. Other options are available to customize the output and improve readability.

- _**Syntax:** du [OPTION] [FILE]_

   _**Options used for the program `du` (more options ![see manpage](https://manpages.debian.org/bookworm/coreutils/du.1.en.html))**_
   - **-x, --one-file-system** (Directories on other file systems will be skipped.)
   - **-h, --human-readable** (Specifies the storage capacity in a binary format in powers of 1024, which is easier for humans to read).
   - **--si** (Specifies the storage capacity in a metric format in powers of 1000, which is easier for humans to read).
   - **-d, --max-depth=N** (Returns the total size of a directory up to the depth N of the given argument).

  #### The command is (please do not run it yet, but read on)
  ```
  du -x -h -d 1 /
  ```
  
## The `sort` program
Before the actual execution of the `du` command, the memory allocation of the output directories should be sorted in descending order of size for a better overview. This is done by the `sort` program.

- _**Syntax: sort [OPTION] [FILE]_

    _**Options used for the sort program (more options ![see manpage](https://manpages.debian.org/bookworm/coreutils/sort.1.en.html))**_
    - **-h, --human-numeric-sort** (prints the file size in a more readable format)
    - **-r, --reverse** (sort in reverse order)

   #### The command is (please do not run it yet, but read on)
   ```
   sort -r -h
   ```

## Combined output from `du` and `sort`
The output of the command `du -x -h -d 1 /` described above can now be passed to the command `sort -r -h` via a so-called pipe `|` to sort the memory allocation in descending order. The commands are combined in one line and executed together.

- _**Syntax:** du [OPTION] [FILE] **|** sort [OPTION] [FILE]_

  Execute the following command to display the used disk space of all directories directly below the root directory of the system partition, recursively and sorted by size.
  ```
  du -x -h -d 1 / | sort -r -h
  ```
    For example, the directory contents of an already slightly damaged system partition with a maximum capacity of 2.3 GiByte might look like this...

      ```
      root@SynologyNAS:~# du -x -h -d 1 / | sort -r -h
      1.9G    /
      1.1G    /usr
      605M    /root
      161M    /var
      43M     /.syno
      11M     /var.defaults
      3.7M    /etc
      2.5M    /etc.defaults
      1.3M    /.log.junior
      28K     /.old_patch_info
      20K     /volumeUSB1
      16K     /.system_info
      16K     /opt
      4.0K    /mnt
      4.0K    /lost+found
      4.0K    /initrd
      ```

  Two things stand out here. First, the `/root` home directory of the superuser root is much larger than expected, taking up 605M (MiByte) of disk space. Second, the directory /volumeUSB1 is shown. Although it seems to take up only 20K (KiByte) of the available disk space, this directory does not belong here because, as mentioned at the beginning, all volumes are located on the data partition. The `-x` option of the `du` command means that directories on other partitions are skipped.

## Narrowing down the combined output further
First, let's take a look at the `/root' directory, as the disk space taken up here may be the cause of an already failed DSM. To find out more about the contents of this directory, we modify the command chain used above and add the directory `/root` to the command `du`.

```
du -x -h -d 1 /root | sort -r -h
```

  The result in this example is as follows.

  ```
  root@SynologyNAS:~# du -x -h -d 1 /root | sort -r -h
  605M    /root/.vscode-server
  605M    /root
  176K    /root/.dotnet
  48K     /root/.ssh
  28K     /root/.cache
  24K     /root/.local
  16K     /root/.config
  ```

To get deeper into the directory structure, you could add the `/root/.vscode-server` directory to the command chain and run it again until you have located the culprit. In this case, since the name of the directory already tells me that it is stored application data of the Visual Studio Code program that logs in to my Synology NAS as root via an SSH connection, I will stop the search at this point and delete the folder completely in the next step to get my DSM working again. Alternatively, I can move individual files or the whole directory to an internal or external volume if I am not sure if I will need the data again.

## The `mv` program (move)
The `mv` program is usually used to move files or entire directories. However, `mv` can also be used to rename files and directories, which will not be discussed here. In addition, no further options are needed in this example, so only the corresponding ![manpage](https://manpages.debian.org/bookworm/coreutils/mv.1.en.html) of `mv` is referred to here. 

- _**Syntax:** mv [SOURCE] [TARGET]_

  In the following, the `mv` command is used to move the last found directory `/root/.vscode-server` to an internal volume of the DSM at `/volume1/backup/system-partition-data`. The directories used here are examples only and must always be replaced with your own.

   ```
   mv /root/.vscode-server /volume1/backup/system-partition-data/
   ```
   
  **Important note:** Make sure that the destination directory ends with a `/`, otherwise `mv` will try to rename the directory `/volume1/backup/system-partition-data` to `/volume1/backup/.vscode-server` instead of moving the source directory `/root/.vscode-server` to `/volume1/backup/system-partition-data`.

## The rm (remove) utility
The `rm` program deletes files or entire directories on the console without going through the recycle bin. **Deleted data cannot be recovered. For this reason, all delete commands should be considered carefully, especially when, for example, the `-r` option for recursive deletion (deletes entire directory structures) is combined with the `-f` option to execute the command without prompting. On the other hand, a permanent prompt can be quite annoying in the long run, which is why this option is still very popular.  

- _**Syntax:** rm [OPTION] [FILE OR DIRECTORY]_

     Options used for the rm program (more options ![see manpage](https://manpages.debian.org/bookworm/coreutils/rm.1.en.html))**_
     - **-r, --recursive** (Delete directories and their contents recursively)
     - **-f , --force** (no prompt when deleting) 

  Although the `rm` command should always be used with caution, it is often necessary to use it. In this example, the `.vscode-server` subdirectory of the `/root` directory will be deleted recursively, along with everything in that directory.

   ```
   rm /root/.vscode-server
   ```

## Final words
In theory, the system partition should now provide enough space for the DSM to run smoothly. What remains in this example is the /volumeUSB1 directory, which can be parsed, moved, or deleted in the same way as the `/root/.vscode-server` directory above. 
