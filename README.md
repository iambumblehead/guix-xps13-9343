![guix](https://upload.wikimedia.org/wikipedia/commons/8/81/Guix_logo.svg)

scheme scm configuration files for guix and the xps13 9343

assumes two things,
 1. **a guix iso with non-free drivers** and iwlwifi kernel module
 2. **a pre-existing disk partition** with swap, efi and root
    * /dev/sda1 efi
    * /dev/sda2 swap my-swap
    * /dev/sda3 ext4 my-root



_wifi.config_
```console
network={
  ssid="ssid-name"
  key_mgmt=WPA-PSK
  psk="unencrypted passphrase"
}
```

Setup a network connection.
```console
rfkill unblock all
ifconfig -a
wpa_supplicant -c wifi.conf -i wlan0 -B
dhclient -v wlan0
```

Format and label "root" and "swap". Mount them. The config will mount root using the label, rather than UUID. (todo: investigate if this can be done with swap)
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
```console
herd start cow-store /mnt
git clone https://github.com/iambumblehead/guix-xps13-9343
mkdir -p ~/.config/guix
cp guix-xps13-9343/.config/guix/channels.scm ~/.config/guix/
guix pull # takes a long time
hash guix
```

Install system config-bare. A small environment is defined with an operational wifi setup. After reboot, one can incrementally update guix home to complete the rest of system. Includes git, emacs and networking tools  [gnu manual][4]
```console
mkdir /mnt/etc
cp guix-xps13-9343/config-bare.scm /mnt/etc/config.scm
emacs -nw /mnt/etc/config.scm # edit root mount point uuid
guix system init /mnt/etc/config.scm /mnt
reboot
```

Use the not-root user to run guix reconfigure and guix pull  [gnu manual][5]
```console
# Set the password for your root account
passwd
# Set the password for your user
passwd <your username>
# Logout and back in
exit
```

[4]: https://guix.gnu.org/manual/en/html_node/Proceeding-with-the-Installation.html
[5]: https://guix.gnu.org/en/manual/en/html_node/After-System-Installation.html#After-System-Installation
