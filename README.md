# My Home Assistant

## RPI 4

## Disable USB Autosuspend

<https://www.zigbee2mqtt.io/guide/faq/#zigbee2mqtt-crashes-after-some-time>

This should help with ZigBee adapter being disconnected, or probably also SSD being disconnected.

```sh
mkdir /boot
echo "usbcore.autosuspend=-1" > /boot/cmdline.txt

reboot
```
