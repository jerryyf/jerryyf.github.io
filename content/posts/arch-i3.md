---
title: "Basic i3 Arch Linux Setup"
date: 2023-03-22
lastmod: 2023-06-21
toc: true
tags: ["linux", "arch", "i3"]
---

![arch-ss.png](/arch-ss.png)

After I botched my Manjaro i3 installation and some further [readings](https://manjarno.snorlax.sh/), I dove deeper into other Arch-based alternatives. Specifically, I wanted a pre-configured tiling window manager setup out of the box as I don't want to spend a lot of time customising and optimising my system. I tried out a couple distros:

- [ArcoLinux](https://arcolinux.com/) is a great distro to get started with ricing Linux. There are different ISOs, suited for different levels of customisation and technical knowledge. ArcoLinuxD comes with a large range of configured desktop environments including GNOME, XFCE, Budgie, Openbox, i3, BSPWM, DWM and more, as well as their custom "tweak tool". Good for getting started and playing around with.
- [EndeavourOS](https://endeavouros.com/) comes with a good range of fully themed yet minimal desktop environments including GNOME, XFCE, KDE Plasma, i3 and Sway. I found it to be a lightweight option with a good balance of customisation and functionality. The out of the box i3 config has some interesting navigation keybindings though - `jkbo` - I change this to `hjkl` for consistency with Vim keybindings.

Here I document my basic setup process for any installation. Please note that this is what works for me and can differ depending on hardware.

## Backups

Having a systematic backup approach is always important, especially in the case of Arch not being known to be the most stable distro. Tools I use:

- `timeshift` for system snapshots - using RSYNC it is lightning fast
- `restic` for home directory backups, also using RSYNC.

Next in `~/.config/i3/`, I change a few keybindings such as `hjkl`. Now a few hardware-specific configuations need to be addressed.

## Brightness adjustment script

Adapted from https://gitlab.com/Nmoleo/i3-volume-brightness-indicator to use `brightnessctl` because xbacklight does not work for me.

```shell
# Uses regex to get brightness from brightnessctl
function get_brightness {
    brightnessctl | grep -Po '[0-9]{1,3}%'
}

# In main function:
case $1 in
    brightness up)
    # Increases brightness and displays the notification
    brightnessctl set $brightness_step%+
    show_brightness_notif
    ;;

    brightness_down)
    # Decreases brightness and displays the notification
    brightnessctl set $brightness_step%-
    show_brightness_notif
    ;;
esac
```

Then in i3's config, call the script functions like so:

```shell
bindsym XF86MonBrightnessUp exec --no-startup-id ~/.config/i3/scripts/volume_brightness.sh brightness_up
bindsym XF86MonBrightnessDown exec --no-startup-id ~/.config/i3/scripts/volume_brightness.sh brightness_down
```

## Internal mic mute button

Dependencies: `amixer`.

```shell
bindsym XF86AudioMicMute exec amixer sset Capture toggle
```

## Touchpad configuration

To enable natural scrolling and tapping on X11 start, in `/usr/share/X11/xorg.conf.d/30-touchpad.conf`:

```shell
Section "InputClass"
    Identifier "devname"
    Driver "libinput"
    MatchIsTouchPad "on"
    Option "Tapping" "on"
    Option "Natural Scrolling" "true"
EndSection

```

## Make Firefox scroll smoothly with the touchpad

By default the scrolling experience is not smooth. This command should fix that:

```shell
echo export MOZ_USE_XINPUT2=1 | sudo tee /etc/profile.d/use-xinput2.sh
```

## Missing fonts

Most of the time special fonts will be missing. These are the packages to install to fix that:

```shell
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
```

## Enable TRIM on SSD drive

Optional, helps maintain SSD performance.

```shell
sudo systemctl enable fstrim.timer
sudo systemctl start fstrim.timer
```

## Syncthing

[Syncthing](https://syncthing.net/) is an excellent open source P2P file sync application I use as a Google Drive/OneDrive/Dropbox alternative. It's exceedingly fast when all your devices are on the same network, and even if they're not there exist relays over TLS for when abroad. It also features encryption for untrusted devices.

```bash
sudo pacman -S syncthing
systemctl enable syncthing@USER.service
systemctl start syncthing@USER.service
```

Don't forget to enable staggered file versioning on important folders.

## Conclusion

With this I have a basic functioning i3 setup. I find that it's hard to go without a tiling window manager anymore, it just makes my workflow much faster and fluid especially on a laptop without a mouse.
