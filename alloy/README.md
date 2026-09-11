# Grafana Alloy for Home Assistant

![Supports amd64 Architecture](https://img.shields.io/badge/amd64-yes-green.svg)
![Supports aarch64 Architecture](https://img.shields.io/badge/aarch64-yes-green.svg)

Ship Home Assistant OS systemd journal logs to Grafana Loki using Grafana Alloy.

Replaces the deprecated Promtail add-on which fails on HAOS 11+ due to systemd 252+ compact journal format incompatibility.

> This is a fork of [ecohash-co/ha-addon-alloy](https://github.com/ecohash-co/ha-addon-alloy), which appears unmaintained (no commits or merged PRs since initial release). Adds `label_overrides` (rename/override journal labels, including the static `job` label) and `disable_reporting` (off by default) — see CHANGELOG.md.

For full documentation, see the **Documentation** tab after installing.
