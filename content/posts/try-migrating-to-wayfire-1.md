---
layout: post
title: Try migrating to wayfire 1
date: 2023-02-22 19:48:29
tags:
    - linux
---

## Preface

Recently, I want to migrate to `windows manager`, and I attempt to leran `wayfire`. In the past, I have heard and tried wayfire, but I didn't migrate to it because I need a GUI file manager.

Now, I want to change my habit to adapt `tui file manager` for focusing on work. Another reason is that `DE` takes up too much memory and has some small bugs from time to time.

So, I chose wayfire, a cascading window manager, not tiling, I prefer cascading to tiling.

## About [Wayfire](https://github.com/WayfireWM/wayfire)

Acording to its self introduction:

> Wayfire is a 3D Wayland compositor, inspired by Compiz and based on wlroots.

> It aims to create a customizable, extendable and lightweight environment without sacrificing its appearance.

![wayfire](https://camo.githubusercontent.com/d98347d40fc6e05519d9ff78c897457f3a02fcbe8194b7e18e0a1e0f93bf7d24/68747470733a2f2f696d672e796f75747562652e636f6d2f76695f776562702f3250744e7a78447378594d2f6d617872657364656661756c742e77656270)

It bases `Wayland` protocol, which is more modern than `X11`, more information can be found [here](https://wayland.freedesktop.org/architecture.html)

## Usage

The official provides a very detailed [tutorial](https://github.com/WayfireWM/wayfire/wiki/Tutorial), so I won’t talk about the basic tutorial here, just mention some problems and points encountered in the setup.

I suggest you read and complete the official getting started guide first

### The keybind of `PRTSC` ?

You need to use `KEY_SYSRQ` instead of `KEY_PRINT` in default config! About more, you can learn from [here](https://github.com/WayfireWM/wayfire/issues/745)!

### `WayBar` start slowly in `Wayfire`?

It seems to be because of `xdg-desktop-portal`, it wakes up by dbus by default, but dbus tends to be slow!

We can add this in wayfire section `autostart`

```ini
dbus-update-activation-environment --systemd DISPLAY WAYLAND_DISPLAY
```

More infomation, [here](https://github.com/swaywm/sway/issues/5732)

## Other component tools

For launcher/menu:

-   [wofi](https://hg.sr.ht/~scoopta/wofi)

For terminal emulator:

-   [foot](https://codeberg.org/dnkl/foot)
-   [alacritty](https://github.com/alacritty/alacritty)
-   [wezterm](https://github.com/wez/wezterm)

For wayland bar:

-   [waybar](https://github.com/Alexays/Waybar)

For graphical notification daemon:

-   [mako](https://github.com/emersion/mako)

For logout menu:

-   [wlogout](https://github.com/ArtsyMacaw/wlogout)

For Idle management daemon:

-   [swayidle](https://github.com/swaywm/swayidle)

For screen locker:

-   [swaylock](https://github.com/swaywm/swaylock)

For auto-reload profile on hotplug:

-   [kanshi](https://wayland.emersion.fr/kanshi/)

For Day/night gamma adjustments:

-   [wlsunset](https://sr.ht/~kennylevinsen/wlsunset/)

For Screenshots:

-   [grim](https://wayland.emersion.fr/grim/)
-   [slurp](https://wayland.emersion.fr/slurp/)

For volume control:

-   [amixer](https://alsa-project.org)

For Screen brightness control:

-   [light](https://haikarainen.github.io/light/)

For Wallpaper:

-   [swaybg](https://github.com/swaywm/swaybg)
-   [wallpaper](https://wall.alphacoders.com/)

For more other programs, you can found [here](https://wiki.archlinux.org/title/List_of_applications/Other).
