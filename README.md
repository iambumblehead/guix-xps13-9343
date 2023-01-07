![guix](https://upload.wikimedia.org/wikipedia/commons/8/81/Guix_logo.svg)

**A guide to setting up guix on an xps13 9343, generic enough to be used for other machines.** Only one xps13-specific thing is here, and it is safe to load that thing anywhere —the i915 kernel module referenced in config.scm. This guide credits and follows [steps outlined][1] by [David Wilson][2] of systemcrafters. The systemcrafters guide has a few outdated and missing areas, updated and covered here.

**When steps are completed to success, the machine boots a minimal environment with emacs and networking tools to enable wifi and ethernet.** Use these to continue setting up a system you prefer, probably using [guix home.][6]

This guide assumes you have,
 1. [A guix iso][0] with [non-free][9] drivers and [iwlwifi][7] kernel module,
 2. A pre-existing disk partition with swap, efi and root. [Create these yourself,][8] or run guix's guided-install once to create them. The guided-install does not install non-free kernel and wifi drivers needed by xps13, but shell-install outlined here provides them.
    * _/dev/sda1 efi_
    * _/dev/sda2 swap my-swap_
    * _/dev/sda3 ext4 my-root_

_wifi.config_
```ruby
network={
  ssid="ssid-name"
  key_mgmt=WPA-PSK
  psk="unencrypted passphrase"
}
```

Boot from the iso. Select locale, language and shell install option. Setup a network connection.
```console
rfkill unblock all
ifconfig -a # list networking devices
wpa_supplicant -c wifi.conf -i wlan0 -B
dhclient -v wlan0
```

Format and label "root" and "swap". Mount them. The config will mount root using the label, rather than UUID. (todo: investigate if this can be done with swap as well)
```console
mkfs.ext4 -L my-root /dev/sda3
mount LABEL=my-root /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
mkswap -L my-swap /dev/sda2
swapon /dev/sda2
mkdir -p /mnt/home
```

Begin installation, pulling non-free channels.
```console
herd start cow-store /mnt
git clone https://github.com/iambumblehead/guix-xps13-9343
mkdir -p ~/.config/guix
cp guix-xps13-9343/.config/guix/channels.scm ~/.config/guix/
guix pull # takes a long time
hash guix
```

Install the config-bare.scm small environment with operational wifi and emacs  [gnu manual][4]
```console
mkdir /mnt/etc
cp guix-xps13-9343/config-bare.scm /mnt/etc/config.scm
emacs -nw /mnt/etc/config.scm # edit root mount point uuid
guix system init /mnt/etc/config.scm /mnt
cp /etc/channels.scm /mnt/home/
reboot
```

Setup root and not-root users to run guix pull and reconfigure, [as recommended by gnu,][5]
```console
passwd # root
passwd <your username> # non-root
cp /home/channels.scm ~/.config/guix/channels.scm
exit # logout and back in, as non-root
cp /home/channels.scm ~/.config/guix/channels.scm
guix pull
sudo guix system reconfigure /etc/config.scm
```

[0]: https://github.com/SystemCrafters/guix-installer/releases/latest
[1]: https://wiki.systemcrafters.cc/guix/nonguix-installation-guide
[2]: https://github.com/daviwil/
[4]: https://guix.gnu.org/manual/en/html_node/Proceeding-with-the-Installation.html
[5]: https://guix.gnu.org/en/manual/en/html_node/After-System-Installation.html#After-System-Installation
[6]: https://guix.gnu.org/manual/devel/en/html_node/Home-Configuration.html
[7]: https://wiki.gentoo.org/wiki/Iwlwifi
[8]: https://guix.gnu.org/manual/en/html_node/Keyboard-Layout-and-Networking-and-Partitioning.html#Disk-Partitioning
[9]: https://gitlab.com/nonguix/nonguix
