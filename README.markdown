# Passy's Raspbian Setup

<img src="https://www.raspberrypi.org/wp-content/uploads/2015/08/raspberry-pi-logo.png" width=150 align=left>

I've got myself a Raspberry Pi 3 and want to use it as my home router.
At some point, I'll probably accidentally step or it or pour a flat white on it
and then wonder how I set it up to do what it's supposed to do.

To avoid this, here are some notes and scripts to make it less painful when that
happens. Don't confuse this with a tutorial. I'm writing this first and foremost
for myself. However, if you have any suggestions, feel free to send PRs my way.

## Goals

- Have a Pi connected via LAN to an existing router/gateway to the internet.
- Serve as DHCP server.
- Serve as VPN-Client and relay. Enable/disable country masking for all
  connected devices.

## Plans

- Investigate if using a modem (or router in bridge-mode) makes sense. Having
full control over the ADSL could be useful. Maybe.

## Preparing the SD Card

- [Get Raspbian Jessie Lite](https://downloads.raspberrypi.org/raspbian_latest.torrent).
  I used `2016-03-18` for this. Also, don't be a jerk, and use the Torrent
  option.
- I use Lite because I don't even want to connect a monitor to the Pi and
  updating Xorg takes ages.
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
- *Dont forget this one:* Resize your partition or you're gonna have a bad time.
  Raspbian only leaves you with a few hundred megabytes left, so run
  `sudo raspi-config` and select option 1: "Expand Filesystem".

## Ansible

Copy `playbook.yml.example` to `playbook.yml` and make the necessary adjustments,
notably add your premiumize.me credentials to it. Afterwards, you can run it
like:

```
ansible-playbook -i hosts playbook.yml
```

This assumes that the hostname in `hosts` can be resolved and you can log in
password-less via the `pi` user. It also expects the Pi to already have a
working internet connection.
