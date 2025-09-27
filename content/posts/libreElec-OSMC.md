+++
title = "Back to Kodi with OSMC"
styled_title = "Back to ~~XBMC~~ Kodi with ~~LibreELEC~~ OSMC"
date = "2025-09-20"
[taxonomies]
tags = ["sw", "hw", "rpi", "sysadmin", "media"]
categories = ["quick-wins", "article"]
+++

> **Note:** _This is the first post on the blog. It’s not the most technical deep-dive yet, but rather a write-up of a recent experiment setting up a media player. Think of it as a warm-up entry while I prepare more in-depth articles._
>

I still remember when it was called XBMC: it felt like regaining the freedom and ownership of media we once had with VHS and DVDs.

In 2025, being utterly dissatisfied with the current cesspool of smart android tv malware, clunky casting devices, and so forth, I am going back to it, via LibreELEC and raspberry pi.

I've dusted off my RPI4B, and removed its SD card.
I did a `dd` image copy of it, as there were still some data of my second home assistant server iteration (which I'm not running on ARM/RPI anymore), and then rewritten it with the latest [LibreELEC image for RPI 4](https://releases.libreelec.tv/LibreELEC-RPi4.aarch64-12.2.0.img.gz).

```bash
$ gunzip LibreELEC-RPi4.aarch64-12.2.0.img.gz
$ sudo dd if=./LibreELEC-RPi4.aarch64-12.2.0.img of=/dev/sdd bs=4M status=progress
```

Then I popped the SD card back in, a previously working 7" HDMI display, and a keyboard to the RPI.

Good start... then nothing.
Just a promising boot sequence, stopping to the not-so-impressive `LibreELEC (Official): 12.2.0` white on black screen.

Adding the `ssh` empty file to the boot partition didn't open up the RPI to ssh, and neither did tweaking `config.txt`.
At this point I was pondering how I need to build a dedicated LibreELEC hardware, but no time for it now.

Searching online I saw that some users have found issues with the `aarch64` version of LibreELEC combined with RPi4/5, and also thought that some regression bugs might have hit for the latest 12.x version.
And so I downloaded LibreELEC 11.0.6 (arm version) and flashed that, tweaked `config.txt`, created the ssh file...

Nothing, even worse.

Pull the SD out, `fsck` boot partition, `e2fsck` storage partition, errors corrected.

NOTHING.

And so, could it still be the SD card?
Downloaded raspios, flashed it, and...
Booted with absolutely no issues, I didn't even have to change hdmi options.

So, I guess that we'll install Kodi on a debian based distribution then.

```bash
$ sudo apt update
$ sudo apt upgrade -y
$ sudo apt install kodi -y
$ kodi-standalone
[...]
Segmentation fault
```

Apparently not.

## Revisiting the distro

Eventually I discovered [OSMC](https://osmc.tv/download/), which already looks better because...
[They've actually designed a dedicated hardware for that](https://osmc.tv/vero/)!
I gave the software a try, I felt more confident in the project since they provide a convenient [AppImage installer](https://ftp.fau.de/osmc/osmc/download/installers/osmc-installer.AppImage) to automatically configure the SD card image.
It boots and starts installing the necessary files.

On my 1080p portable HDMI screen I see the reboot splash and then..._just static noise_?

It turns out on the main 4k/50" screen it works fine, but apparently there are some issues with RPI4 and legacy displays, a problem I won't delve in this time.
And after setting up the library connections and jellyfin plugin (the wifi connection must be setup at the SD configuration, so that the RPI wireless peripheral is correctly installed) everything just worked.

I suppose I've found the solution!
So now, after the story part, some considerations.

## Pros and Cons

Let's start with the downsides.

While not an OSMC issue, raspberrypi 4 is still stuck at 30fps for 4k output.
Dual HDMI is great, but in the media scenario it’s not as useful, a stronger hardware platform becomes necessary - which gives more points to consider Vero4k+.

Kodi is great, but in 2025 the standard UI - at least used in the OSMC setup - is kind of dated looking.

<img src="/images/kodi-menu.png" alt="Clean and minimal, old looking Kodi standard menu" style="max-width: 100%; height: auto;">

There are some better interfaces, which is easy enough to download and install, but in general I guess there is some work to do on the UX and the feeling of the graphics.
Not much of a critic (as I only make `TUIs` - if I really have to) but a strong suggestion in order to make a valid alternative to streaming services for non technical people.

Then, the worse point for me, media server integration.
There is no specific advantage in using Kodi/Emby.
If I spent several days setting up my media server and streaming player with hardware accelerated transcoding, I then still have to kludge the jellyfin way into kodi to use that.
If not, use samba shares or other file system shares 🤔

Samba with my NAS works fine, but it should really be simpler (ideally _off the shelf_) to just connect Kodi directly to a Jellyfin server and synchronize libraries.
The jellyfin plugins worked just fine, but I'm thinking of the layman, the ecosystem still feels too fragmented.

Now for the pros:

- full linux environment with native ssh access 💖
- the OSMC installation media setup works great and is multiplatform 💯
- OSMC installed and ran flawlessly
- Library setup and sync is easy and works very well (3/10 difficulty for a 'layman', 0/10 for a sysadmin)
- Playback is smooth (4k - possibly transcoded, didn't really check - 30fps over wifi)

And one great surprise for me: CEC over HDMI works perfectly on first try!
This means that my family is going to _actually use it_, as they won't be restricted by a less friendly looking keyboard laying around the couch, but the TV remote works with Kodi now.
The menus are easy to browse with the remote, and the input lag is up to speed with basically any ~~smart~~ _commercial_ TV.

## Ideas for the next level

Hardware might be already sorted, I plan on trying OSMC Vero4k in the future, and see if that fixes the main RPI4 limitation of 60fps at 4k.
But if I had to draft the system architecture for such a device:

- Small sized application core, possibly with integrated (up to) HEVC10/12 decoding
- HDMI 2.1 output with proper 4k60 HDR support
- Dual-band Wi-Fi 6 and Gigabit (or 2.5Gb) Ethernet (for stable streaming)
- Sufficient RAM (≥ 2 GB) to cache large media libraries
- eMMC or NVMe storage option for faster boot and metadata access
- Silent cooling (passive or near-silent fanless design)
- Optional external storage expansion (USB-C / SATA)

For the software:

- wizard-like setup for the libraries
- native Jellyfin (and Plex/Emby) integration
- unified search across local files + media servers (seems to work, but with many duplicates)
- automatic codec and display optimization (e.g., detect 4k60 HDR and adjust)
- minimal “streaming service” mode with only essential functions exposed
- optional advanced mode for power users (SSH, Samba, NFS, etc.)
- continuous updates without breaking configs (transactional upgrades, rollbacks)
- integrated parental controls and multi-user profiles
- remote control HDMI-CEC + apps

## Conclusion

So in the end, LibreELEC on RPi4 was a dead end for me this time: too many regressions, too much trial and error for something that should _just work_.

OSMC, on the other hand, booted, installed, and ran with barely any friction.
Kodi isn’t perfect, the UI feels dated, and media server integration is still hacky, but the basics are there: HDMI-CEC works, playback is smooth enough, and my family can actually use it without me acting as tech support every night 🎉.

That’s already a win.
Next step will be trying proper dedicated hardware (Vero4k or similar) to see if it fixes the 4k/60fps ceiling and gives a cleaner experience.

For now, job done: back to Kodi, in 2025.