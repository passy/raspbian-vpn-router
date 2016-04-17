# Passy's Raspbian Setup

<img src="https://www.raspberrypi.org/wp-content/uploads/2015/08/raspberry-pi-logo.png" width=150 align=left>

I've got myself a Raspberry Pi 3 and want to use it as my home router.
At some point, I'll probably accidentally step or it or pour a flat white on it
and then wonder how I set it up to do what it's supposed to do.

To avoid this, here are some notes and scripts to make it less painful when that
happens. Don't confuse this with a tutorial. I'm writing this first and foremost
for myself. However, if you have any suggestions, feel free to send PRs my way.

## Preparing the SD Card

- [Get Raspbian Jessie](https://downloads.raspberrypi.org/raspbian_latest.torrent).
  I used `2016-03-18` for this. Also, don't be a jerk and use the Torrent
  option.
- Copy it onto an SD card. I'll leave it to you to figure out which device
  you're writing to, but keep in mind that this image is intended to partition
  the entire card, so don't specify a partition (i.e. `sdb` not `sdb1`):
  `dd bs=4M if=2016-03-18-raspbian-jessie.img of=/dev/sdb`

## Initial setup

- Plug the Pi in and connect it to a DHCP-enabled router via ethernet.
- My old DD-WRT had the `.lan` domain, so I could directly connect to
  `raspberrypi.lan`. Obviously, take care of conflicts if you have more than
  one. `nmap` is your friend.
- `ssh pi@raspberrypi.lan`, password is `raspberry`.
- This could potentially be part of the ansible script, but I prefer pushing the
  SSH key over manually to minimize the chance of locking myself out.
- `ssh pi@raspberrypi.lan "mkdir -p ~/.ssh/"; scp ~/.ssh/id_rsa.pub pi@raspberrypi.lan:/home/pi/.ssh/authorized_keys`
