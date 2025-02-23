# Merge Aggregator Plugin

This plugin merges metrics of the same series and timestamp into new metrics
with the super-set of fields. A series here is defined by the metric name and
the tag key-value set.

Use this plugin when fields are split over multiple metrics, with the same
measurement, tag set and timestamp.

⭐ Telegraf v1.13.0
💻 all

## Global configuration options <!-- @/docs/includes/plugin_config.md -->

In addition to the plugin-specific configuration settings, plugins support
additional global and plugin configuration settings. These settings are used to
modify metrics, tags, and field or create aliases and configure ordering, etc.
See the [CONFIGURATION.md][CONFIGURATION.md] for more details.

[CONFIGURATION.md]: ../../../docs/CONFIGURATION.md#plugins

## Configuration

```toml @sample.conf
# Merge metrics into multifield metrics by series key
[[aggregators.merge]]
  ## Precision to round the metric timestamp to
  ## This is useful for cases where metrics to merge arrive within a small
  ## interval and thus vary in timestamp. The timestamp of the resulting metric
  ## is also rounded.
  # round_timestamp_to = "1ns"

  ## The period on which to flush & clear each aggregator. 
  ## All metrics that are sent with timestamps outside of this period will be ignored by the aggregator.
  # period = "30s"

  ## The delay before each aggregator is flushed. 
  ## This is to control how long for aggregators to wait before receiving metrics from input plugins, 
  ## in the case that aggregators are flushing and inputs are gathering on the same interval.
  # delay = "100ms"

  ## The duration when the metrics will still be aggregated by the plugin, 
  ## even though they're outside of the aggregation period. 
  ## This is needed in a situation when the agent is expected to receive late metrics 
  ## and it's acceptable to roll them up into next aggregation period.  
  # grace = "0s"

  ## If true, the original metric will be dropped by the
  ## aggregator and will not get sent to the output plugins.
  drop_original = true
```

## Example

```diff
- cpu,host=localhost usage_time=42 1567562620000000000
- cpu,host=localhost idle_time=42 1567562620000000000
+ cpu,host=localhost idle_time=42,usage_time=42 1567562620000000000
```
