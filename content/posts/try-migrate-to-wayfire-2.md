---
layout: post
title: Try migrating to wayfire 2
date: 2023-02-24 23:20:28
tags:
    - linux
---

## Preface

I have already migrate to wayfire, but I also have some other minor issues when using.

## Question

The following records the problems I encountered during use.

### Nm-connection-editor doesn't save password ?

`nm-connection-editor` depends on gnome-keyring, but it's a bit heavy for `wm`, so I'm looking for a lighter alternative.

The solution now is to install two package called `gnome-keyring` and `libsecret`, and add `gnome-keyring-daemon --start` to sectin `autostart`.

Then, everything about network will work fines.

### Prevent multiple startups when using wofi ?

You can use the following command to start wofi, its function is to find wofi in the process now, if not found, start a wofi instance!

```sh
pgrep -x wofi >/dev/null 2>&1 || wofi
```

### Lightdm cannot start wayfire normally ?

When try to login into `wayfire` wayland session through `Lightdm`, it loads for some time and then goes back to login screen

Acorrding to this [issue](https://github.com/canonical/lightdm/issues/63) and this [article](https://blog.lilydjwg.me/2021/11/15/wayfire-migration-progress.215972.html)

We can add this looping block into `.xprofile`:

```sh
if [ -n "$XDG_VTNR" ] ; then
    echo "Waiting for VT $XDG_VTNR:"
    for ((i=0;i<100;i++)) ; do
      CURRENT="$(</sys/devices/virtual/tty/tty0/active)"
      echo  "  $(date +%s.%N) using $CURRENT"
      if [[ "$CURRENT" = "tty$XDG_VTNR" ]]; then
          break
      fi
      sleep 0.01
    done
fi
```

That is because lightdm runs under `X11`, and then it will attempt to start `wayland.` But `X11` takes a finite amount of time to release the video driver. when wayland start, the video driver has not been released yet, then wayland will not found vedio output and will be suspended, so timeout, return lightdm.

More infomation, can be found [here](https://github.com/WayfireWM/wayfire/issues/1479).

### About volume control

We can use `pavucontrol` to control pulseaudio, which is a tui tool.

we should note some device which has mutiple output interfaces, we should select proper interface.

For `pipewire`, we can use `pamixer`, this can control volume by console.

!!!!!!!!!!
More to be done!
!!!!!!!!!!
