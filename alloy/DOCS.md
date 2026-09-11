# Grafana Alloy for Home Assistant

Ship Home Assistant OS logs to a remote [Loki](https://grafana.com/oss/loki/) instance using [Grafana Alloy](https://grafana.com/docs/alloy/latest/).

This add-on replaces the deprecated Promtail add-on, which is incompatible with modern HAOS versions (11+) due to systemd 252+ compact journal format changes.

## Configuration

### Required

- **loki_url**: The full URL to your Loki push endpoint (e.g., `http://192.168.1.45:3100/loki/api/v1/push`)

### Optional

- **log_level**: Alloy log verbosity (`debug`, `info`, `warn`, `error`). Default: `info`
- **additional_config**: Extra Alloy config blocks to append (advanced users)
- **label_overrides**: JSON object (as a string) overriding the default label names/values — see below. Default: none
- **disable_reporting**: Disable Alloy's built-in anonymous usage reporting to `stats.grafana.com` (passes `--disable-reporting` to the Alloy binary). Default: `true`. Set to `false` to opt back into upstream telemetry.

## Labels

All journal entries are shipped to Loki with these labels:

| Label | Source | Overridable via `label_overrides` |
|-------|--------|---|
| `job` | `systemd-journal` (static) | `job` — replaces the **value** |
| `unit` | systemd unit name | `unit` — renames the **label key** |
| `hostname` | machine hostname | `hostname` — renames the **label key** |
| `syslog_identifier` | process identifier | `syslog_identifier` — renames the **label key** |
| `transport` | journal transport type | `transport` — renames the **label key** |
| `container_name` | Docker container name (for add-ons) | `container_name` — renames the **label key** |
| `level` | log priority (debug, info, warning, error, etc.), reparsed from message content for container-sourced entries — see below | `level` — renames the **label key** |

### Overriding labels

`label_overrides` is a JSON object entered as a single string (same input pattern as `additional_config`), e.g.:

```json
{"job": "home-assistant", "hostname": "host"}
```

This does two different things depending on the key, because `job` isn't derived from a journal field the way the others are:

- `job` is a static value on the journal source — overriding it **replaces the value** (e.g. Loki streams show `job="home-assistant"` instead of `job="systemd-journal"`), useful for matching a `job` naming scheme already used by other Loki sources on the same instance.
- Every other key (`unit`, `hostname`, `syslog_identifier`, `transport`, `container_name`, `level`) is derived from a journal field via a relabel rule — overriding it **renames the label itself** (e.g. `{"hostname": "host"}` makes entries carry a `host` label instead of `hostname`), useful for matching a label naming scheme used by other log shippers (Promtail, syslog-ng, other Alloy instances) writing to the same Loki instance.

Target label names must match `^[a-zA-Z_][a-zA-Z0-9_]*$` (standard Prometheus/Loki label name rules) — the add-on fails to start with a clear error instead of generating a broken pipeline if one doesn't.

### Container log level correction

dockerd logs every container's stdout at journal priority `info` and stderr at `err`, regardless of what the container actually wrote — so any add-on/container that logs to stderr (a common pattern, not an error indicator) had every single line mislabeled `level=error` upstream. For any entry carrying a `container_name` label (or your renamed equivalent), this add-on now:

1. drops the priority-derived `level` label,
2. re-parses the real level from the line's own text (case-insensitive match on `trace`/`debug`/`info`/`notice`/`warning`/`warn`/`error`/`err`/`critical`/`crit`/`fatal`, normalized to `warning`/`error`/`critical`/etc.), and
3. leaves lines with no recognizable level token unlabeled, so Loki's own `detected_level` can take over instead.

Native (non-container) journal entries are unaffected — they keep the accurate priority-based level. This targets the same root cause as upstream issue/PR discussion around container log levels, generalized here to respect `label_overrides` if you've renamed `container_name` or `level`.

## Debug UI

The Alloy debug UI is available at `http://<haos-ip>:12345` when the add-on is running. Use it to inspect component health, view the pipeline DAG, and troubleshoot issues.

## Advanced: Additional Config

The `additional_config` option lets you append raw Alloy config blocks. For example, to also scrape a file:

```
local.file_match "extra" { path_targets = [{__path__ = "/config/home-assistant.log"}] }
loki.source.file "extra" { targets = local.file_match.extra.targets forward_to = [loki.write.loki.receiver] }
```

Note: This is injected as-is into the config file. Syntax errors will prevent Alloy from starting.

## Troubleshooting

- **No logs in Loki**: Check that `loki_url` is reachable from HAOS. Try `ping <loki-host>` from the SSH add-on.
- **Add-on crashes on start**: Check the add-on log for Alloy config errors. Set `log_level: debug` for verbose output.
- **"timestamp too old" in Loki**: Normal on first start. Alloy reads the full journal history; Loki rejects entries outside its retention window. Resolves in 1-2 minutes.

## Support

Report issues at: https://github.com/ecohash-co/ha-addon-alloy/issues
