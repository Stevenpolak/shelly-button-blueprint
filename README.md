# Shelly Dimmer Webhook Automation for Home Assistant

A standalone Home Assistant automation for the Shelly Dimmer 1/2 (Gen 1) that handles short press and hold-to-dim via Shelly URL actions (webhooks).
Based upon the beautiful blueprint for [Shelly i4 by Ltek](https://gist.github.com/Ltek/27a83afea0a073834052bd055995da4b)
Built because the popular Shelly i4 blueprint does not work on Gen 1 devices. Gen 1 fires different `shelly.click` event names and, more critically, never fires `btn_up`, which breaks hold-to-dim completely. Using webhooks instead gives back the button release signal via `btn1_off_url`.

---

## Requirements

- Shelly Dimmer 1 or Dimmer 2 (Gen 1)
- Home Assistant with the attached Automation installed
- Button configured as **Detached** in the Shelly firmware (required for short/long press URLs to fire)

---

## What it does

| Action | Result |
|---|---|
| Short press | Toggle light |
| Hold | Dims up or down (direction based on current brightness) |
| Release | Stops dimming |

Dimming direction is decided once at the start of each hold: if the light is below the boundary percentage it dims up, otherwise it dims down. The loop floors at 10% and will not go lower.

---

## Shelly setup

In the Shelly web UI, set the button type for SW1 to Detached. Without this, `btn1_shortpush_url` and `btn1_longpush_url` will not fire.

Then go to `http://<shelly-ip>/settings/actions` and set the following, replacing `<HA_IP>` with your Home Assistant's local IP:

```
btn1_shortpush_url  ->  http://<HA_IP>:8123/api/webhook/shelly_dimmer_btn1_short
btn1_longpush_url   ->  http://<HA_IP>:8123/api/webhook/shelly_dimmer_btn1_long
btn1_off_url        ->  http://<HA_IP>:8123/api/webhook/shelly_dimmer_btn1_off
```

All webhook triggers use `local_only: true`, so no authentication is needed and nothing is exposed outside your network.

---

## Dimming configuration

All settings are in the `variables:` block at the bottom of the YAML.

| Variable | Default | Description |
|---|---|---|
| `dim_target` | your light entity | Light or group to control |
| `dim_step` | `6` | Brightness change per cycle, in percent |
| `dim_delay` | `0.12` | Seconds between each step |
| `dim_transition` | `0` | Light fade per step in seconds. Must be less than `dim_delay` or steps overlap. Leave at 0 for Gen 1 hardware dimmers |
| `dim_boundary` | `50` | Brightness percentage at which dim direction flips |

---

## Notes

- Double press is not available. Gen 1 Shelly webhooks do not expose a double-press URL.
- The dim stop mechanism relies on `mode: restart`. When the button is released, the `btn1_off` webhook fires a new automation instance, which kills the running dim loop.
- If dimming feels sluggish, lower `dim_delay`. If commands are being dropped, raise it slightly.
- Tested on Shelly Dimmer 2 running firmware 1.14.x.
