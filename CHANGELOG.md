# Changelog

## [2.0.0](https://github.com/s-gordon/ESPHome-Configs/compare/v1.0.0...v2.0.0) (2026-08-21)


### ⚠ BREAKING CHANGES

* entity names, file paths and substitution names have all changed. Device configs must have their files: list rewritten to the new paths, and Home Assistant entity IDs will move -- automations, scripts and dashboards referencing the old ids need updating. Pin ref: to v1.0.0 to stay on the old scheme.

### Features

* normalise naming across packages ([6ed0116](https://github.com/s-gordon/ESPHome-Configs/commit/6ed0116836d7adb00926a8816e759b3828e440f1))


### Fixes

* modernise ota syntax and drop unresolvable sensor reference ([b278b1e](https://github.com/s-gordon/ESPHome-Configs/commit/b278b1e1ca779b94ce5f3135f4d446eb61a75748))
