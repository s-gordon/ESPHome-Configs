# ESPHome-Configs

Shared [ESPHome](https://esphome.io) package files for my Home Assistant devices,
consumed remotely over the `packages:` `remote_package` mechanism so devices can be
kept in sync from one place rather than copy-pasted per device.

## Layout

```
esphome/packages/config/
├── deta/grid-connect-switch/       # DETA Grid Connect wall switches
│   ├── base-esp8266.yaml           # ESP8266 (esp01_1m) platform + common entities
│   ├── base-bk72xx-wb3s.yaml       # Beken BK72XX, wb3s module
│   ├── base-bk72xx-cb3s.yaml       # Beken BK72XX, cb3s module
│   ├── 1-gang-esp8266.yaml
│   ├── 2-gang-esp8266.yaml
│   ├── 3-gang-esp8266.yaml
│   ├── 3-gang-middle-fan-esp8266.yaml   # as 3-gang, 2nd gang exposed as a fan
│   ├── 1-gang-bk72xx-cb3s.yaml
│   ├── 2-gang-bk72xx-wb3s.yaml
│   └── example-device.yaml         # example per-device config to copy locally
└── antsig/ir-blaster/
    ├── base-bk72xx-cb3s.yaml       # transmitter, receiver, status LED
    ├── climate-daikin.yaml         # Daikin climate over IR
    └── climate-fujitsu.yaml        # Fujitsu General climate over IR
```

Every file is named `<variant>-<chip>[-<module>].yaml`, so a base and a variant pair up
when their chip suffixes match. Mixing an `-esp8266` base with a `-bk72xx-` variant is a
validation error, not a silent misconfiguration.

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

The status LED is exposed through the `light:` domain rather than the top-level
`status_led:` component, so it still indicates connection state but can also be turned
off from Home Assistant — useful on a bedroom switch. Each gang exposes both its relay (a `light`, or a `fan` on the middle gang of the
fan variant) and its physical button as a `binary_sensor`. The button is wired to
its own relay on-device, so the switch works without the network; the button entity
is exposed as well because it is the only thing that distinguishes a physical press
from an API command, and it lets press gestures be built in Home Assistant without
reflashing.

Entity names are bare (`1st`, `1st Button`, `Restart`). ESPHome already prefixes them
with the device's `friendly_name` in Home Assistant, so repeating it in `name:` would
double it up. Gangs are numbered by ordinal rather than named by position, because
position depends on how the switch was mounted.

Everything device-specific (`name`, `friendly_name`, API key, OTA password, wifi
credentials) stays in the local config on the ESPHome host; nothing secret lives here.

## Usage

Copy `esphome/packages/config/deta/grid-connect-switch/example-device.yaml` into your ESPHome
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
    ref: v2.0.0
    refresh: never
    files:
      - esphome/packages/config/deta/grid-connect-switch/base-esp8266.yaml
      - esphome/packages/config/deta/grid-connect-switch/1-gang-esp8266.yaml

api:
  encryption:
    key: !secret apikey
  reboot_timeout:
    hours: ${timout_hours}

ota:
  - platform: esphome
    password: !secret ota_password

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
fallback AP, and the example above replaces it with a password-protected one. Keep
the base's `captive_portal:` — these are in-wall switches, so pulling one out to
reach a USB port is not a realistic recovery path, and an AP with no captive portal
and no `web_server:` cannot be used to configure or monitor anything.

For the IR blaster, include `antsig/ir-blaster/base-bk72xx-cb3s.yaml` (which expects
`room` and `area` substitutions) plus one `climate-*.yaml` for your AC unit. The
climate packages read the current temperature from a sensor the device config must
supply with `id: temperature`:

```yaml
sensor:
  - platform: homeassistant
    id: temperature
    entity_id: sensor.living_room_temperature
```

Then build and flash from the ESPHome builder / dashboard as usual — no external
tooling beyond what ESPHome ships. Devices already running these configs update OTA.

## Devices covered

| Device | Chip | Base | Variant |
| --- | --- | --- | --- |
| DETA Grid Connect, 1 gang | ESP8266 | `base-esp8266.yaml` | `1-gang-esp8266.yaml` |
| DETA Grid Connect, 2 gang | ESP8266 | `base-esp8266.yaml` | `2-gang-esp8266.yaml` |
| DETA Grid Connect, 3 gang | ESP8266 | `base-esp8266.yaml` | `3-gang-esp8266.yaml` |
| DETA Grid Connect, 3 gang w/ fan | ESP8266 | `base-esp8266.yaml` | `3-gang-middle-fan-esp8266.yaml` |
| DETA Grid Connect, 1 gang | BK72XX / cb3s | `base-bk72xx-cb3s.yaml` | `1-gang-bk72xx-cb3s.yaml` |
| DETA Grid Connect, 2 gang (6912HAMBK) | BK72XX / wb3s | `base-bk72xx-wb3s.yaml` | `2-gang-bk72xx-wb3s.yaml` |
| Antsig wifi IR universal remote | BK72XX / cb3s | `antsig/ir-blaster/base-bk72xx-cb3s.yaml` | a `climate-*.yaml` |

`min_auth_mode:` is ESP8266/ESP32 only — setting it on a BK72XX device fails config
validation.

## Versioning

Releases are tagged with [SemVer](https://semver.org) and cut by `release-please` from
[Conventional Commits](https://www.conventionalcommits.org). **Pin `ref:` to a tag**
rather than tracking `main` — package changes here reach every device on its next
build, and a tag lets you roll one device forward, live with it, then move the rest.
Use `refresh: never` with a pinned tag; `refresh: 1d` only adds a network round-trip
against an immutable ref.

What the version numbers mean for a consumer:

| Bump | Means | Examples |
| --- | --- | --- |
| Major | Action needed on your side | entity `name:` changes (Home Assistant entity IDs move), substitution renames, GPIO reassignments, file renames — which break the `files:`/`!include` list |
| Minor | New things, safe to take | new device variants, new optional substitutions |
| Patch | Fixes, safe to take | typos, deprecated-syntax updates |

One version series covers every device family here, so an IR-blaster change can bump
the number even if you only use the switches. Read the [CHANGELOG](CHANGELOG.md)
before moving a major.

## Continuous integration

`.github/workflows/validate.yaml` runs `esphome config` over the fixtures in
[tests/](tests/) on every push and PR, against both the currently-deployed ESPHome
version and the latest release, plus weekly to catch upstream regressions.

The base files declare `min_version:` matching the older of those two, so a consumer
on an ESPHome older than anything CI covers fails loudly instead of mis-parsing. Bump
the two together.

CI validates configs; it does not compile them. Schema and substitution errors are
caught, but compile failures and flash overflow are not.

The fixtures pull the packages in by **local relative path**, not via
`remote_package` — so CI validates the branch under test rather than whatever is
published on `main`. When you add a device variant, add a fixture for it in the same
commit, otherwise nothing covers it. `tests/secrets.yaml` holds throwaway values and
is never used on a real device.

## Licence

MIT — see [LICENSE](LICENSE).
