# Changelog

## 1.2.0 - 2026-09-11

### Fixed
- Container-sourced log lines (anything with a `container_name` label) no longer inherit journal priority as their `level` — dockerd tags all container stderr as priority `err` regardless of actual content, mislabeling routine info-level output as errors. The level is now re-parsed from the line's own text; unrecognized lines are left unlevelled so Loki's `detected_level` applies instead. Native (non-container) journal entries are unaffected. Ported from upstream PR [#11](https://github.com/ecohash-co/ha-addon-alloy/pull/11) (closed, unmerged), generalized to respect `label_overrides` renames of `container_name`/`level`.

## 1.1.0 - 2026-09-11

### Added
- `label_overrides` option: override the static `job` label value, and/or rename any of the journal-derived label keys (`unit`, `hostname`, `syslog_identifier`, `transport`, `container_name`, `level`) to match another naming scheme already in use on your Loki instance. Fails loudly on an invalid label name instead of generating a broken pipeline. Fixes upstream [#13](https://github.com/ecohash-co/ha-addon-alloy/issues/13) and [#10](https://github.com/ecohash-co/ha-addon-alloy/issues/10).
- `disable_reporting` option (default `true`): passes `--disable-reporting` to the Alloy binary, silencing usage-report spam to `stats.grafana.com` on networks that block it (Pi-hole/AdGuard/strict egress). Fixes upstream [#4](https://github.com/ecohash-co/ha-addon-alloy/issues/4). Default changed from upstream's implicit "always report" behavior — set `disable_reporting: false` to opt back in.

### Changed
- Default behavior change: telemetry reporting is now off by default (see above). All other defaults are unchanged; existing configs without `label_overrides` or `disable_reporting` set generate byte-identical Alloy config aside from the reporting flag.

## 1.0.0 - 2026-02-21

### Added
- Initial release
- Grafana Alloy v1.13.1
- Systemd journal log shipping to Loki
- Journal field relabeling (unit, hostname, syslog_identifier, transport, container_name, level)
- Debug UI on port 12345
- Configurable Loki URL, log level, and additional config
- Watchdog health check via Alloy's `/-/ready` endpoint
- Support for amd64 and aarch64 architectures
