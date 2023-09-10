# Installing hardware, OS and other required software

## Setting up the software

### Partitioning the SD Card

* Insert the SD card (via adapter) into MBP
* Figure out disk id (for me, it was `disk4`)
* `diskutil partitionDisk disk4 2 MBR "MS-DOS FAT32" ALPINE 1GB "MS-DOS FAT32" LINUX R`
* Download Raspberry Pi Alpine Linux from
  <https://dl-cdn.alpinelinux.org/alpine/v3.15/releases/armhf/alpine-rpi-3.15.0-armhf.tar.gz>
* `cd /Volumes/ALPINE; tar xvf ~/Downloads/alpine-rpi-3.15.0-armhf.tar.gz`
* Create usercfg.txt consisting of:

    ```text
    dtparam=audio=on
    display_rotate=2
    ```

* Unmount the SD card
* Put the SD card into the Pi, cable up the Pi and apply power

### Installing/configuring Alpine

* localhost login prompt: Username `root`, no password required
* If needed, fix up usercfg.txt (e.g. to set/remove `display_rotate=2`)
* Basically, follow <https://wiki.alpinelinux.org/wiki/Classic_install_or_sys_mode_on_Raspberry_Pi#Installation>
    * Format the second partition: `apk add e2fsprogs; mkfs.ext4 /dev/mmcblk0p2`
    * Run `setup-alpine` and work through the setup
    * Then do the various faff to get 'sys' mode working, which avoids the need to `lbu commit -d` all the time
* Check for OS updates: `apk update; apk  upgrade`

### Installing prerequisites for the music player

Logged in as root:

```sh
apk add mpg123
apk add alsa-utils
addgroup root audio
apk add py3-pillow
apk add git
apk add python3
wget https://bootstrap.pypa.io/get-pip.py -O get-pip.py
python3 get-pip.py
# If it's something you'll use:
apk add rsync
```

(Attempts to `pip install` Pillow results in it trying to build from source)

### Installing prerequisites for the touchscreen UI

Logged in as root:

* `setup-xorg-base`
* `apk add mesa-dri-vc4 mesa-dri-swrast mesa-gbm xf86-video-fbdev xfce4 xfce4-terminal`
* `apk add xset py3-gobject3`
* Edit /media/mmcblk0p1/usercfg.txt, to add:

    ```text
    lcd_rotate=2
    dtoverlay=vc4-fkms-v3d
    gpu_mem=256
    ```

    * `lcd_rotate=2` seems to be needed instead of `display_rotate=2` after installing X and these drivers

* Create `/etc/X11/xorg.conf`:

    ```text
    Section "Device"
      Identifier "default"
      Driver "fbdev"
    EndSection
    ```

### Installing prerequisites for showing now-playing information for radio stations

If you're planning to use piju to play radio stations, and want to include
now-playing artist and track information in the web ui or the touchscreen UI,
you'll need to install another piece of software:

Logged in as root:

```sh
apk add jq
```

### Installing prerequisites for playing tracks from YouTube

If you're planning to play tracks from YouTube, you'll need to install a couple
of extra pieces of software. If you don't plan to use the YouTube
functionality, you can skip this section.

Logged in as root:

```sh
wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O /usr/local/bin/yt-dlp
chmod a+rx /usr/local/bin/yt-dlp
apk add ffmpeg
```

### Create a user to run the software

Logged in as root:

```sh
adduser piju
addgroup piju wheel  # for sudo
addgroup piju audio,tty,input,video  # input and video needed for X, i.e. for the touchscreen UI
```
