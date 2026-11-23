# esphome packages

Reusable ESPHome packages, organized in three layers:

- `boards/` - board/hardware definition (MCU, pins, battery, ...); the reTerminal
  E1001/E1002 share `seeed-e10xx.yaml` (same MCU, battery and button pins), only the
  panel differs and is configured in the device files
- `default/` - generic defaults (wifi, logger, diagnostic, time, ...)
- `devices/` - device use cases, referencing a board; shared logic lives in a common
  file (e.g. `dashboard-image.yaml`) and panel specific parts (display, image format)
  in per-device files (e.g. `e1001-dashboard-image.yaml`)
- `examples/` - ready to use example configs, built by CI on every PR

## reTerminal E1001 / E1002 dashboard image

`devices/dashboard-image.yaml` contains the shared logic (image download, deep sleep,
buttons, ...). `devices/e1001-dashboard-image.yaml` (reTerminal E1001, 7.5" mono e-paper)
and `devices/e1002-dashboard-image.yaml` (reTerminal E1002, 7.3" 6-color e-paper) add the
panel specific display configuration. Together they display an image on the e-paper
display, e.g. a Home Assistant dashboard screenshot rendered by the
[Pupeteer/Puppet add-on](https://github.com/lovelylain/ha-pupeteer) on port 10000.

See `examples/e1001-dashboard-image.yaml` and `examples/e1002-dashboard-image.yaml`.

### Image URL

The image URL is not hardcoded in the config. Point the `ha_image_url_entity` substitution
to a Home Assistant `input_text` helper that contains the URL:

```yaml
substitutions:
  ha_image_url_entity: "input_text.reterminal_dashboard_image_url"
```

Example URL (Puppet add-on):

```text
http://homeassistant.local:10000/lovelace?viewport=480x800&format=bmp&theme=Graphite+E-ink+Dark&zoom=1.1&lang=de&colors=000000%2CFFFFFF&palette_colors=000000%2CFFFFFF&dithering=sierra
```

For the E1002 (color display) use `viewport=800x480` and PNG, e.g.
`http://homeassistant.local:10000/lovelace?viewport=800x480&format=png`.

The URL can be changed at runtime from Home Assistant; the last used URL is stored in
flash and restored after a reboot.

### Deep sleep

The device never enters deep sleep on its own. From Home Assistant:

1. Set the number entity `<friendly_name> Deep sleep duration` (minutes)
2. Press the button `<friendly_name> Deep sleep`

The device then sleeps for the configured duration and wakes up again. The right button
(GPIO4) can also be used to wake the device manually.

### Buttons

- Green button (GPIO3): refresh the display immediately
- Right button (GPIO4): wake up from deep sleep
- Left button (GPIO5): unused
