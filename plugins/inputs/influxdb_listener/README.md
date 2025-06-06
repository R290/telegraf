# InfluxDB Listener Input Plugin

This plugin listens for requests sent according to the
[InfluxDB HTTP v1 API][influxdb_http_api]. This allows Telegraf to serve as a
proxy/router for the `/write` endpoint of the InfluxDB HTTP API.

> [!NOTE]
> This plugin was previously known as `http_listener`. If you wish to
> send general metrics via HTTP it is recommended to use the
> [`http_listener_v2`][http_listener_v2] instead.

The `/write` endpoint supports the `precision` query parameter and can be set
to one of `ns`, `u`, `ms`, `s`, `m`, `h`.  All other parameters are ignored and
defer to the output plugins configuration.

> [!IMPORTANT]
> When chaining Telegraf instances using this plugin, `CREATE DATABASE` requests
> receive a `200 OK` response with message body `{"results":[]}` but they are
> not relayed. The configuration of the output plugin ultimately submits data
> to InfluxDB determines the destination database.

⭐ Telegraf v1.9.0
🏷️ datastore
💻 all

[influxdb_http_api]: https://docs.influxdata.com/influxdb/v1.8/guides/write_data/
[http_listener_v2]: /plugins/inputs/http_listener_v2/README.md

## Service Input <!-- @/docs/includes/service_input.md -->

This plugin is a service input. Normal plugins gather metrics determined by the
interval setting. Service plugins start a service to listen and wait for
metrics or events to occur. Service plugins have two key differences from
normal plugins:

1. The global or plugin specific `interval` setting may not apply
2. The CLI options of `--test`, `--test-wait`, and `--once` may not produce
   output for this plugin

## Global configuration options <!-- @/docs/includes/plugin_config.md -->

In addition to the plugin-specific configuration settings, plugins support
additional global and plugin configuration settings. These settings are used to
modify metrics, tags, and field or create aliases and configure ordering, etc.
See the [CONFIGURATION.md][CONFIGURATION.md] for more details.

[CONFIGURATION.md]: ../../../docs/CONFIGURATION.md#plugins

## Configuration

```toml @sample.conf
# Accept metrics over InfluxDB 1.x HTTP API
[[inputs.influxdb_listener]]
  ## Address and port to host HTTP listener on
  service_address = ":8186"

  ## maximum duration before timing out read of the request
  read_timeout = "10s"
  ## maximum duration before timing out write of the response
  write_timeout = "10s"

  ## Maximum allowed HTTP request body size in bytes.
  ## 0 means to use the default of 32MiB.
  max_body_size = 0

  ## Set one or more allowed client CA certificate file names to
  ## enable mutually authenticated TLS connections
  tls_allowed_cacerts = ["/etc/telegraf/clientca.pem"]

  ## Add service certificate and key
  tls_cert = "/etc/telegraf/cert.pem"
  tls_key = "/etc/telegraf/key.pem"

  ## Optional tag name used to store the database name.
  ## If the write has a database in the query string then it will be kept in this tag name.
  ## This tag can be used in downstream outputs.
  ## The default value of nothing means it will be off and the database will not be recorded.
  ## If you have a tag that is the same as the one specified below, and supply a database,
  ## the tag will be overwritten with the database supplied.
  # database_tag = ""

  ## If set the retention policy specified in the write query will be added as
  ## the value of this tag name.
  # retention_policy_tag = ""

  ## Optional username and password to accept for HTTP basic authentication
  ## or authentication token.
  ## You probably want to make sure you have TLS configured above for this.
  ## Use these options for the authentication token in the form
  ##   Authentication: Token <basic_username>:<basic_password>
  # basic_username = "foobar"
  # basic_password = "barfoo"

  ## Optional JWT token authentication for HTTP requests
  ## Please see the documentation at
  ##   https://docs.influxdata.com/influxdb/v1.8/administration/authentication_and_authorization/#authenticate-using-jwt-tokens
  ## for further details.
  ## Please note: Token authentication and basic authentication cannot be used
  ##              at the same time.
  # token_shared_secret = ""
  # token_username = ""

  ## Influx line protocol parser
  ## 'internal' is the default. 'upstream' is a newer parser that is faster
  ## and more memory efficient.
  # parser_type = "internal"
```

## Metrics

Metrics are created from InfluxDB Line Protocol in the request body.

## Example Output

Using

```sh
curl -i -XPOST 'http://localhost:8186/write' --data-binary 'cpu_load_short,host=server01,region=us-west value=0.64 1434055562000000000'
```

will produce the following metric

```text
cpu_load_short,host=server01,region=us-west value=0.64 1434055562000000000
```
