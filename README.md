# PiJu

## About this project

This is a Raspberry Pi powered jukebox. Background: I had a mini hifi system, with speakers, etc, in the kitchen. In principle, I wanted to switch our old CD collection to digital, but having tried several systems, I'd not found a UI I liked. So, the idea is: Raspberry Pi plugged into aux in of existing hifi, with an iPad app to control it, and I'll eventually add a 7" touchscreen display to the Pi for simple controls too.

## Hardware

* RPi 3 model B
* 128GB micro SD
* Cable up the RPi (video, keyboard, mouse) but leave unpowered for now

## Install the OS

First installed Raspberry Pi OS, but decided to switch to Alpine Linux for its quicker boot time.

<strike>
### Using Raspberry Pi OS

* Image the micro SD with latest version of Raspberry Pi OS (2020-06-20)
* Insert the micro SD into the Pi
* Apply power
* Wait for a while as the RPi does its thing. Go through the setup wizard.
* Install OS updates
* Install mopidy <https://docs.mopidy.com/en/latest/installation/raspberrypi/>
* as per <https://docs.mopidy.com/en/latest/installation/debian/#debian-install>
* Install mopidy-local: `sudo apt install mopidy-local`
* (Note: Media folder `/var/lib/mopidy/media/`)
</strike>

### Using Alpine Linux

* Initialise the 128GB SD to FAT32 using the [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/)
* Extract [alpine-rpi-3.12.0-aarch64.tar.gz](https://alpinelinux.org/downloads/) onto the SD card
* Add a `usercfg.txt` file to the root of the SD card that contains a single line `dtparam=audio=on`
* Insert the micro SD into the Pi
* Apply power.
* Now edit partition table: run `fdisk` to edit the partition table and create a small FAT32 and a big Linux partition
* Quit fdisk and `shutdown` the pi.
* (With hindsight, this partitioning scheme might have been possible on the Mac without the extra faff of preparing the Pi only to immediately reset it, but it was only a later idea to switch to separate partitions)
* Reinsert SD card into Mac. Use Disk Utility to format the FAT32 partition. Re-extract alpine...tar.gz onto the SD card and create the usercfg.txt file. Unmount the SD card.
* SD card back into Pi. Re-power Pi.
* Run `setup-alpine`. The [questions](https://wiki.alpinelinux.org/wiki/Installation#Questions_asked_by_setup-alpine) are all pretty obvious.
* Check for OS updates: `apk update; apk  upgrade`
* Format the second partition: `apk add e2fsprogs; mkfs.ext4 /dev/mmcblk0p2`
* `lbu commit -d; reboot` to check for correct behaviour. No DHCP IP address received for wlan0. After some investigation, and after a reboot:
* `rc-update add wpa_supplicant boot; rc-update add wpa_cli boot; lbu commit -d; reboot`
* Add the data partition to `/etc/fstab`:
    * `/dev/mmcblk0p2 /media/mmcblk0p2 ext4 relatime 0 0`
    * `lbu commit -d`
* Create a user to run mopidy: `adduser -h /media/mmcblk0p2/.mopidy mopidy`
* and give that user access to audio: `addgroup mopidy audio`
* Install mopidy (references [1](https://docs.mopidy.com/en/latest/installation/pypi/), [2](https://pip.pypa.io/en/stable/installing/), [3](https://wiki.alpinelinux.org/wiki/Enable_Community_Repository)), ensuring that everything is written to a persistent voluem: 
    * `apk add python3`
    * `apk add curl`
    * Enable the Community repository: `vi /etc/apk/repositories`, uncomment the community line, `:wq`, `apk update`
    * `apk add gstreamer gst-plugins-base gstreamer-tools gst-plugins-good gst-plugins-bad gst-libav`
        * `gst-plugins-good` enables mp3, `gst-plugins-bad` and `gst-libav` together enable mp4/m4a files
    * `apk add py3-gst`
    * `apk add faad2`
* For the next steps, you must be logged in as the mopidy user
    * `curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py`
    * `python3 get-pip.py --user`
    * Heed the warnings that pip isn't on the path, and create/edit .profile: `export PATH=~/.local/bin:$PATH`  Log out and log back in to ensure that the changes are correct
    * `pip install --user --upgrade mopidy`
    * `pip install --user --upgrade mopidy-local`  # but see below, too
    * `pip install --user --upgrade mopidy-iris`
    * `lbu commit -d`
* To facilitate future transfers:
    * `apk add rsync`
    * `lbu commit -d`


# Configuring mopidy and installing data

Note that directories will depend upon your OS. These instructions are for Alpine Linux.

* Log in as the mopidy user
* Start mopidy once, to generate default config file: `mopidy` / Ctrl-C
* Edit `~/.config/mopidy/mopidy.conf` to:
    * enable the local extension: in the `[local]` section, uncomment `enabled = true`
    * set the media folder: in the `[local]` section, set `media_dir = /media/mmcblk0p2/media`
    * increase the scan timeout: in the `[local]` section, set `scan_timeout = 30000`
    * set a low scan flush threshold, to ensure it doesn't run out of memory when scanning: in the `[local]` section, set `scan_flush_threshold = 5`
    * set the http listen port: in the `[http]` section, set `hostname = 0.0.0.0`
* If using Alpine Linux: `lbu commit -d`
* If using a non-default media folder (which will be the case with Alpine Linux): `mkdir /media/mmcblk0p2/media`
* Copy data into the media folder. `rsync -rvt /your/source/collection/Music/ mopidy@piju:/media/mmcblk0p2/media` will work, but will take a while.
    * Note that the trailing slash on the source folder is important, and `-t` is also important to avoid times changing the next time you run the command, which will cause `mopidy local scan` to revisit them.
* Finally, `mopidy local scan` 

# Switching to operational mode

* Install an init script - scp `init.d/mopidy` to `/etc/init.d/mopidy` (as root)
* Make the script executable:

  ```
  chmod +x /etc/init.d/mopidy
  ```
* Add this as a service:

  ```
  rc-update add mopidy default
  ```
* Commit the changes to the system:

  ```
  lbu include /etc/init.d
  lbu commit -dv
  ```
  
  Note that the `lbu commit` command only needs to be run once. The `-v` option to `lbu commit` makes it more verbose, so you can confirm that `/etc/init.d/mopidy` is included in the archive.
* Also, switch the media partition to read-only so that it doesn't matter if the Pi loses power:
    * As root, add `ro` to the options for the partition in `/etc/fstab`
    * Also, `mount -o remount,ro /media/mmcblk0p2`
* Then, as root:
    * `lbu commit -d`

# Adding new music to the Pi

On the Pi, as root:

```
mount -o remount,rw /media/mmcblk0p2
```

On the machine where the music is stored, repeat the rsync line from above. Consider adding `--ignore-existing` if you're only adding new files.

On the Pi, as mopidy user:

```
mopidy local scan
```

On the Pi, as root:

```
mount -o remount,ro /media/mmcblk0p2
```

# Adding artwork to the Pi

On the Pi, as root:

```
mount -o remount,rw /media/mmcblk0p2
```

The `rsync` rune will not suffice. The .jpg file gets copied, but `mopidy local scan` does not detect the new artwork. The only effective workaround seems to be `dir=$(dirname path/to/new/cover.jpg); touch $dir/*.mp3`.


# De-duplicating compilation albums

mopidy-local distinguishes albums based on their year as well as the name, etc. But the information it gets from gstreamer is only about the tracks, so this goes wrong for compilations: the tracks may span many years, despite definitely being a single release. Option 1: Remove all the date information from the mp3s. Option 2. Fork mopidy-local.  See <https://github.com/nsw42/mopidy-local> for the work in progress.

# Switching to custom mopidy-local

Instead of the `pip install mopidy-local` (and `pip uninstall mopidy-local` reverts it, if necessary), take the following steps:

* As root:

    ```
    mount -o remount,rw /media/mmcblk0p2
    ```

* As the mopidy user:

    ```
    wget https://github.com/nsw42/mopidy-local/archive/master.zip
    unzip mopidy-local.zip
    cd mopidy-local-master
    python3.8 setup.py install --prefix=/media/mmcblk0p2/.mopidy/.local/
    ```

* At this point, `mopidy --help` will confirm that mopidy-local was installed correctly.

* As the mopidy user:

    ```
    mopidy local scan --force
    ```

* As root:

    ```
    rc-service mopidy restart
    ```
