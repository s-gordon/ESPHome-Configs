# Changelog

## [3.0.0](https://github.com/s-gordon/ESPHome-Configs/compare/v2.0.0...v3.0.0) (2026-08-21)


### ⚠ BREAKING CHANGES

* each device gains a controllable `Status` light entity, and configs on ESPHome older than 2026.7.4 will now be rejected outright.
* the antsig package directory and base filename have changed, so device configs including them must have their files: list updated. Configs using a climate package must now define a temperature sensor with `id: temperature`, or validation will fail.

### Features

* :truck: changes formatting of antsig IR blasters to match DG switches ([8975868](https://github.com/s-gordon/ESPHome-Configs/commit/897586819e53a1600d70b845a0e9517f776ca9b9))
* expose button entities on the wb3s 2-gang variant ([2ead345](https://github.com/s-gordon/ESPHome-Configs/commit/2ead345caffc55a91a197e4897167f74bcf2ad0b))
* expose status LED as a light and declare min_version ([873c761](https://github.com/s-gordon/ESPHome-Configs/commit/873c761670d9b3535c0429582415107560e308bb))
* normalise antsig paths and restore climate temperature sensor ([03a6090](https://github.com/s-gordon/ESPHome-Configs/commit/03a609017fa654dddaecd97e366b32db317d5ae9))


### Fixes

* pin antsig bk72xx framework to the recommended LibreTiny version ([6ef0b59](https://github.com/s-gordon/ESPHome-Configs/commit/6ef0b59eb9cc9e771fed7421ea29c6729f93b0f0))
* pin bk72xx framework to the recommended LibreTiny version ([665b67d](https://github.com/s-gordon/ESPHome-Configs/commit/665b67d627dae69c90be339152f5c74babcb8ed1))
* repair antsig friendly_name substitution and catch it in CI ([eac2674](https://github.com/s-gordon/ESPHome-Configs/commit/eac2674d14d6c63baa59e0f1de1a1dd1ec17f535))


### Documentation

* record the status LED trade-off and how to switch back ([b2ab282](https://github.com/s-gordon/ESPHome-Configs/commit/b2ab2823f718cb8330dd202581a840cc43b70efc))

## [2.0.0](https://github.com/s-gordon/ESPHome-Configs/compare/v1.0.0...v2.0.0) (2026-08-21)


### ⚠ BREAKING CHANGES

* entity names, file paths and substitution names have all changed. Device configs must have their files: list rewritten to the new paths, and Home Assistant entity IDs will move -- automations, scripts and dashboards referencing the old ids need updating. Pin ref: to v1.0.0 to stay on the old scheme.

### Features

* normalise naming across packages ([6ed0116](https://github.com/s-gordon/ESPHome-Configs/commit/6ed0116836d7adb00926a8816e759b3828e440f1))


### Fixes

* modernise ota syntax and drop unresolvable sensor reference ([b278b1e](https://github.com/s-gordon/ESPHome-Configs/commit/b278b1e1ca779b94ce5f3135f4d446eb61a75748))
