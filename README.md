![guix](https://upload.wikimedia.org/wikipedia/commons/8/81/Guix_logo.svg)

scheme scm configuration files for guix and the xps13 9343

assumes two things,
 1. **a guix iso with non-free drivers** and iwlwifi kernel module
 2. **a pre-existing disk partition** with swap, efi and root
    * /dev/sda1 efi
    * /dev/sda2 swap my-swap
    * /dev/sda3 ext4 my-root


Setup a network connection.

_wifi.config_
```console
network={
  ssid="ssid-name"
  key_mgmt=WPA-PSK
  psk="unencrypted passphrase"
}
```

```console
rfkill unblock all
ifconfig -a
wpa_supplicant -c wifi.conf -i wlan0 -B
dhclient -v wlan0
```

Format "root" and "swap" with labels, then mount them. The root label allows guix configuration to mount by label rather than random UUID. (should investigated if this can be done with swap)
```console
mkfs.ext4 -L my-root /dev/sda3
mount LABEL=my-root /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
mkswap -L my-swap /dev/sda2
swapon /dev/sda2
mkdir -p /mnt/home

```

Begin installation, initialize non-free channels
``` console
herd start cow-store /mnt
git clone https://github.com/iambumblehead/guix-xps13-9343
mkdir -p ~/.config/guix
cp guix-xps13-9343/.config/guix/channels.scm ~/.config/guix/
guix pull # takes a long time
hash guix
```

Install system config-bare. This will define a small environment that boots with an operational wifi setup and the user (you) can incrementally add guix home coniguration to complete the rest of system.  [gnu manual](https://guix.gnu.org/manual/en/html_node/Proceeding-with-the-Installation.html)
``` console
mkdir /mnt/etc
cp guix-xps13-9343/config-bare.scm /mnt/etc/config.scm
emacs -nw /mnt/etc/config.scm # edit root mount point uuid
guix system init /mnt/etc/config.scm /mnt
```
