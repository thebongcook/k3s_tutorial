# Tips and tricks

## Here's how to mount the Raspberry Pi's ext4 filesystem on a Mac:

1. Plug the microSD card into a card reader connected to your Mac. The boot volume will be automatically mounted, but it doesn't contain all the files from the Pi's primary filesystem.
2. Make sure you have Homebrew installed (instructions here), so you can install the tools you need to mount the filesystem.
3. Using Homebrew, install osxfuse and ext4fuse (find out more about the tools on the FUSE for macOS website):
```
brew install --cask macfuse
brew install --formula --build-from-source ./src/ext4fuse.rb
```
4. Use Disk Utility on the command line to find the Raspberry Pi's partition ID; run `diskutil list` to get output like below:
```
$ diskutil list
/dev/disk0 (internal):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                         500.3 GB   disk0
   1:                        EFI EFI                     314.6 MB   disk0s1
   2:                  Apple_HFS Macintosh HD            499.3 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3

/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *32.0 GB    disk4
   1:             Windows_FAT_32 boot                    66.1 MB    disk4s1
   2:                      Linux                         31.9 GB    disk4s2
```
You should be able to tell which drive is your Pi drive by the description (external, physical), the **Linux** partition type, and the size of the disk (e.g. 31.9 GB for my 32 GB card). The ID is the disk4s2 in my case, in the IDENTIFIER column.

5. Create a 'mount point'—a folder on your Mac where you will 'mount' the Linux partition so you can read data from it: `sudo mkdir /Volumes/rpi` (sudo requires you to enter your Mac account's admin password, since it performs actions with elevated privileges—enter your password when prompted.)

6. Mount the drive using ext4fuse: `sudo ext4fuse /dev/disk4s2 /Volumes/rpi -o allow_other`
> The -o allow_other is required to make sure the mounted disk is readable by everyone (and not just the sudo/root user). See this issue: Unable to open ext4 mounted partition on El Captain.

Now you'll see the rpi volume mounted in the Finder. You can open it and read from it just like any other disk, card, or flash drive you connect to your Mac.

Once you're finished, make sure you safely unmount the disk, by either ejecting the disk in the finder, or running sudo umount /Volumes/rpi in Terminal. After that, you can unplug the card and put it back in your Pi, where it will be ready to do more awesome Pi things!

Credits: Jeff Geerling

#### Source
- [Original Blogpost](https://www.jeffgeerling.com/blog/2017/mount-raspberry-pi-sd-card-on-mac-read-only-osxfuse-and-ext4fuse)
- [Update for 2021](https://github.com/gerard/ext4fuse/issues/66)
