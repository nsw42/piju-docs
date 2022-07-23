# PiJu

## About this project

This is a Raspberry Pi powered jukebox. Background: I had a mini hifi system, with speakers, etc, in the kitchen. In principle, I wanted to switch our old CD collection to digital, but having tried several systems, I'd not found a UI I liked. So, the idea is to use a Raspberry Pi plugged into aux in of existing hifi, with an iPad app to control it, and a 7" touchscreen display to the Pi for simple controls too.

I'd previously used <https://mopidy.com> but found that the API didn't really suit my project's needs. After adding more and more bespoke extensions, I eventually bit the bullet and replaced the entire backend.

## Hardware

* RPi 3 model B
* 128GB micro SD
* Cable up the RPi (video, keyboard, mouse) but leave unpowered for now

## Installation steps

1. Intall the OS: See (for now) <https://github.com/nsw42/piju-server/blob/main/doc/deploy.md>
1. Install the piju server. See <https://github.com/nsw42/piju-server/blob/main/doc/deploy.md>
1. Install the touch-screen UI software. See <https://github.com/nsw42/pijuui/blob/main/README.md>
1. Install the web ui. See <https://github.com/nsw42/pijuwebui/blob/main/doc/deploy.md>
1. If you're not interested in the web ui, you might want to install lighttpd to download music files. See [lighttpd_download.md](lighttpd_download.md)

## Troubleshooting

### No sound through audio jack

Not sure what caused this: an OS update? adding additional overlays? But `lsmod | grep snd` showed that `snd_bcm2835` was not being loaded. A simple `modprobe snd-bcm2835` fixes it temporarily, but obvs that shouldn't be necessary.

Adding `snd_bcm2835` to `/etc/modules`, with obligatory `lbu commit -d` ensures no `modprobe` is necessary.

Also installed all of the ALSA packages along the way. Not sure if they were necessary, or just useful for troubleshooting.

Once I had sound, audio output was very quiet. Added a line to `run.sh` that uses the `amixer` command to set audio output to 100%.
