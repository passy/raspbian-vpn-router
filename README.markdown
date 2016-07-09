# Passy's Raspbian Setup

<img src="https://www.raspberrypi.org/wp-content/uploads/2015/08/raspberry-pi-logo.png" width=150 align=left>

I've got myself a Raspberry Pi 3 and want to use it as my home router and VPN
gateway.
At some point, I'll probably accidentally step or it or pour a flat white on it
and then wonder how I set it up to do what it's supposed to do.

To avoid this, here are some notes and scripts to make it less painful when that
happens. Don't confuse this with a tutorial. I'm writing this first and foremost
for myself. However, if you have any suggestions, feel free to send PRs my way.

## Goal

Have a WiFi access point that I can connect my phone and ChromeCast to and
transparently get routed through an OpenVPN.

**Stretchgoal:** Have an API to switch between the VPNs. Perhaps even a physical
button? How cool would that be?

## Non-Goals

- Connect to the internet through a modem. I'd rather switch this on and off
  when I need it and it simplifies the setup and firewall rules a ton.

## Status

**It's working!**

You get a WiFi hotspot that tunnels all requests through `tun0` which is
backed by an OpenVPN connection. I'm not sure if this is stable enough for
use and there's no good way from the outside to enable/disable VPNs.

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

## Using the WiFi

Everything should automatically start up, but Linux boxes are painfully
stateful so if the VPN or something else didn't come up, just reboot that thing.

Afterwards, you should be able to connect to the device via WiFi. The defaults
are SSID "passy-pi" and password "raspberrypi". Afterwards, your connected
device should be transparently routed through the VPN. Sorry, Netflix.

<img src="https://i.imgur.com/f5V3BnV.jpg" width=500>

I now have a second ChromeCast that only connects to this AP and hence thinks
it's in whatever country I tell it to. Neat.

## Testing the Router manually

In case something is wrong with the WiFi and you just want to verify if
the routing works, you can add a route manually.

On your target device, assuming it's IPv4:

```
# Figure out your current default route:
$ route -n | grep 0.0.0.0
# Delete the default GW
$ route del default gw <ip_address_from_above>
# Add the new route to the Pi
$ route add default gw <pi_ip_addr>
```

## Updating parts of the config

Something I only found out about when working on this that I should have really known before is that you can selectively run ansible tasks by specifying tags.

If you, for instance, change your VPN settings but don't want to also run an `apt-get upgrade` you can use the `openvpn` tag like so:

```
ansible-playbook -i hosts playbook.yml --tags openvpn
```

Multiple tags can be comma-separated.

## DNS Rerouting

Certain movie streaming services has gotten *a lot more aggressive* lately and not only block the usual suspects, but entire fucking IP ranges (both IPv4 **and** IPv6) for hosting providers like DigitalOcean, AWS and Linode. I wish they had done this a couple of weeks earlier so I could have avoided all the previous work.

But anyway, one so far unaddressed attack vector is using a custom DNS. This even comes with a bunch of benefits like better performance and no traffic limits. And all it takes is a couple of `iptables` rules to rewrite all DNS requests to those custom servers.

I'm currently testing [ViperDNS](https://www.viperdns.com) for just that, which even offers a 7 day free trial.

In order to prepare your Pi for the service, you need to change your `playbook.yml` ever so slightly. You can either leave the `openvpn` part out entirely if you've never provisioned your device before or (damn you statefulness) make sure to set `openvpn.autostart` to `none` to avoid unnecessarily spinning up instances. Be aware that an empty string here means running daemons for *all* `*.conf` files in `/etc/openvpn/`.

Check out [`playbook.yml.dnsexample`](./playbook.yml.dnsexample) for an example. The interesting bits here are `firewall.mode: "dns"` rather than `"tunnel"` which is the default and the `force_dns: "185.51.194.194"` which overrides all incoming DNS requests (or anything really talking to port 53) to the given IP address.

Afterwards, just replay the playbook and reboot the device.

If you've updated just the DNS redirect and want to skip the other tasks, the
tag you want to use is `firewall`, i.e.


```
ansible-playbook -i hosts playbook.yml --tags firewall
```
