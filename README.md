![guix](https://upload.wikimedia.org/wikipedia/commons/8/81/Guix_logo.svg)

**A guide to setting up guix on an xps13 9343, generic enough to be used for other machines.** Only one xps13-specific thing is here, and it is safe to load that thing anywhere --an i915 kernel module defined in config.scm. This guide credits and follows [steps outlined][1] by [David Wilson][2] of systemcrafters. The systemcrafters guide has a few outdated and missing areas, and does not demonstrate the nonguix vanilla linux kernel used by this guide.

**When steps are completed to success, the machine boots a minimal environment with git, emacs and networking tools that enable wifi and ethernet.** Use these to continue setting up a system you prefer, probably using [guix home.][6] Needed configuration files are stored with this guide.

This guide assumes you have,
 1. A guix iso with non-free drivers and [iwlwifi][7] kernel module, [link][0]
 2. A pre-existing disk partition with swap, efi and root
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

Setup a network connection.
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

Begin installation, pulling non-free channels
```console
herd start cow-store /mnt
git clone https://github.com/iambumblehead/guix-xps13-9343
mkdir -p ~/.config/guix
cp guix-xps13-9343/.config/guix/channels.scm ~/.config/guix/
guix pull # takes a long time
hash guix
```

Install system config-bare.scm. A small environment is defined with operational wifi, git and emacs  [gnu manual][4]
```console
mkdir /mnt/etc
cp guix-xps13-9343/config-bare.scm /mnt/etc/config.scm
emacs -nw /mnt/etc/config.scm # edit root mount point uuid
guix system init /mnt/etc/config.scm /mnt
reboot
```

Setup root and not-root users to run guix pull and reconfigure, [as recommended by gnu,][5]
```bash
passwd # root
passwd <your username> # non-root
exit # logout and back in
guix pull
```

[0]: https://github.com/SystemCrafters/guix-installer/releases/latest
[1]: https://wiki.systemcrafters.cc/guix/nonguix-installation-guide
[2]: https://github.com/daviwil/
[4]: https://guix.gnu.org/manual/en/html_node/Proceeding-with-the-Installation.html
[5]: https://guix.gnu.org/en/manual/en/html_node/After-System-Installation.html#After-System-Installation
[6]: https://guix.gnu.org/manual/devel/en/html_node/Home-Configuration.html
[7]: https://wiki.gentoo.org/wiki/Iwlwifi
