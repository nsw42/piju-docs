# PiJu

## About this project

This is a Raspberry Pi powered jukebox. Background: I had a mini hifi system,
with speakers, etc, in the kitchen. In principle, I wanted to switch our old
CD collection to digital, but having tried several systems, I'd not found a UI
I liked. So, the idea is to use a Raspberry Pi plugged into an existing hifi,
with an iPad to control it, and a 7" touchscreen display to the Pi for simple
controls too.

I'd previously used <https://mopidy.com> but found that the API didn't really
suit my project's needs. After adding more and more bespoke extensions, I
eventually bit the bullet and replaced the entire backend.

## Hardware

* RPi 3 model B (or newer)
* 128GB micro SD
* Cable up the RPi (video, keyboard, mouse) but leave unpowered for now

## Installation steps

1. Set up the hardware and configure the OS. See <install_hardware_and_os.md>
1. Add music to the server. See <add_music.md>
1. Install the piju server software.
   See <https://github.com/nsw42/piju-server/blob/main/doc/deploy.md>
1. If relevant, install the touch-screen UI software.
   See <https://github.com/nsw42/piju-touchscreen/blob/main/README.md>
1. Install the web ui software.
   See <https://github.com/nsw42/piju-webui/blob/main/doc/deploy.md>
1. If you're not interested in the web ui, you might want to install lighttpd
   to download music files. See [lighttpd_download.md](lighttpd_download.md)

## Troubleshooting: No sound through audio jack

If you find you are not getting audio through the audio jack, it's worth
checking that the right kernel modules are being loaded. Run
`lsmod | grep snd` and check that `snd_bcm2835` is included in the output.

If it's not, add `snd_bcm2835` to `/etc/modules` and reboot.

I also installed all of the ALSA packages during troubleshooting, which
may turn out to be part of the solution.

Once I had sound, audio output was very quiet, so I added a line to my local
copy of `run.sh` that uses the `amixer` command to set audio output to 100%.

## Better quality audio

If you are using one of the audio HATs for the Raspberry Pi (eg I have a
[HiFiBerry DAC2 Pro](https://www.hifiberry.com/shop/boards/hifiberry-dac2-pro))
then configure ALSA to make that your default:

Run the command `aplay -l`, and identify which card is the right one. Eg, with
the HiFiBerry card, there is a line that starts 'card 1: sndrpihifiberry ...'

Then, as root, create or edit the file `/etc/asound.conf`, so that it looks
something like this (with the appropriate card number submitted):

```text
pcm.!default {
  type hw card 1
}
ctl.!default {
  type hw card 1
}
```

## Additional systems

The PiJu system supports remote sytems, which act as a caching replica of the
main system. Decide which Pi you want to be the main unit, as this is the one
where you will add new music. The secondary units have exactly the same
hardware configuration as the main server, and just fetch music files from the
primary Pi as needed. Installation steps for the secondary systems are similar
to that of the main system:

1. Install the OS
1. Install the piju server software, but run the replica subsystem, rather than
   the main backend, and point it at the main Pi for its source.
1. If desired, install the touch-screen UI software.
1. Instal1 the web ui.
1. Connect to the replica's web ui,
