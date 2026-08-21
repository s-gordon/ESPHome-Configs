# ESPHome-Configs

Shared [ESPHome](https://esphome.io) package files for my Home Assistant devices,
consumed remotely over the `packages:` `remote_package` mechanism so devices can be
kept in sync from one place rather than copy-pasted per device.

## Layout

```
esphome/packages/config/
├── detagrid-smart-switch/          # DETA Grid Connect wall switches
│   ├── base.yaml                   # ESP8266 (esp01_1m) platform + common entities
│   ├── base-BK72XX.yaml            # Beken BK72XX, wb3s module
│   ├── base-BK72XX-cb3s.yaml       # Beken BK72XX, cb3s module
│   ├── 1-gang.yaml / 2-gang.yaml / 3-gang.yaml          # ESP8266 pinouts + entities
│   ├── 3-gang-middle-fan.yaml      # as 3-gang, middle gang exposed as a fan
│   ├── 1-gang-cb3s.yaml            # BK72XX (cb3s) pinout + entities
│   ├── DETA-Grid-Connect-Smart-Switch-2-gang_BK72XX.yaml
│   └── esphome.yaml                # example per-device config to copy locally
└── antsig/smart-wifi-ir-universal-remote/
    ├── base.yaml                   # cb3s IR blaster: transmitter, receiver, status LED
    ├── climate-daikin.yaml         # Daikin climate over IR
    └── climate-fujitsu.yaml        # Fujitsu General climate over IR
```

## How it fits together

Each device config is composed from two package files:

1. **A base file** — picks the chip/board (`esp8266:` or `bk72xx:`), sets up wifi
   fallback AP, captive portal, logger, wifi-signal/uptime sensors and a restart switch.
2. **A gang/variant file** — declares the GPIO substitutions for that hardware revision
   plus the `output` / `light` / `fan` / `binary_sensor` entities. Each physical button
   toggles its own relay locally, so the switch still works without the network.

Pin naming differs by chip family: ESP8266 variants use `GPIOxx`, Beken BK72XX variants
use `Pxx`. Pick the variant file matching the module actually inside the device —
these switches ship with different internals under the same model number.

Everything device-specific (`name`, `friendly_name`, API key, OTA password, wifi
credentials) stays in the local config on the ESPHome host; nothing secret lives here.

## Usage

Copy `esphome/packages/config/detagrid-smart-switch/esphome.yaml` into your ESPHome
config directory as `<device>.yaml`, set the substitutions, and uncomment the package
files that match your hardware. A real example, an older DETA Grid 1-gang light switch:

```yaml
substitutions:
  name: "dg-bedroom"
  friendly_name: "DetaGrid 1Gang Bedroom"
  timout_hours: 4

packages:
  remote_package:
    url: https://github.com/s-gordon/ESPHome-Configs
    ref: main
    refresh: 1d
    files:
      - esphome/packages/config/detagrid-smart-switch/base.yaml
      - esphome/packages/config/detagrid-smart-switch/1-gang.yaml

api:
  encryption:
    key: !secret apikey
  reboot_timeout:
    hours: ${timout_hours}

ota:
  - platform: esphome
    password: !secret ota_password

captive_portal: !remove

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: on
  domain: !secret wifi_domain
  ap:
    ssid: "${name} AP"
    password: !secret ap_password
    ap_timeout: ${timout_hours}h
  reboot_timeout:
    hours: ${timout_hours}
  min_auth_mode: WPA2
```

Note that the local config has the last word: the base packages give you an open
fallback AP and a captive portal, and the example above replaces the AP with a
password-protected one and drops the captive portal with `!remove`.

For the IR blaster, include `antsig/smart-wifi-ir-universal-remote/base.yaml` (which
expects `room` and `area` substitutions) plus one `climate-*.yaml` for your AC unit.

Then build and flash from the ESPHome builder / dashboard as usual — no external
tooling beyond what ESPHome ships. Devices already running these configs update OTA.

## Devices covered

| Device                                        | Chip          | Files                                              |
| --------------------------------------------- | ------------- | -------------------------------------------------- |
| DETA Grid Connect smart switch, 1/2/3 gang    | ESP8266       | `base.yaml` + `N-gang.yaml`                        |
| DETA Grid Connect smart switch, 3 gang w/ fan | ESP8266       | `base.yaml` + `3-gang-middle-fan.yaml`             |
| DETA Grid Connect 2 gang (6912HAMBK)          | BK72XX / wb3s | `base-BK72XX.yaml` + `DETA-...-2-gang_BK72XX.yaml` |
| DETA Grid Connect 1 gang                      | BK72XX / cb3s | `base-BK72XX-cb3s.yaml` + `1-gang-cb3s.yaml`       |
| Antsig smart wifi IR universal remote         | BK72XX / cb3s | `antsig/.../base.yaml` + a `climate-*.yaml`        |

## Licence

MIT — see [LICENSE](LICENSE).
