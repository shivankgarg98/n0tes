# Gray and yellow are too bright and blend with white

Colors are represented incorrectly when connecting to a remote Samsung 2043NW
monitor using a HDMI-VGA adapter.

## Solution

```sh
xrandr --output HDMI-1 --set "Broadcast RGB" "Limited 16:235"
```

## References

- https://wiki.archlinux.org/index.php/Xrandr#Full_RGB_in_HDMI
- https://superuser.com/q/1214434/442991
