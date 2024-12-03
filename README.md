# Tado° Prometheus Exporter

[![Push on branch](https://github.com/acoster/tado-exporter/actions/workflows/master.yml/badge.svg)](https://github.com/acoster/tado-exporter/actions/workflows/master.yml)

This is a Prometheus exporter for [tado°](https://www.tado.com/) thermostats, based on [eko/tado-exporter](https://github.com/eko/tado-exporter)

## Prerequisites

In case you want to develop on this project, you will need:

* [Rust](https://www.rust-lang.org/)

## Installation

### Using Docker

The exporter is also available as a [Docker image](https://ghcr.io/acoster/tado-exporter).
You can run it using the following example and pass configuration environment variables:

```
$ docker run \
  -e 'EXPORTER_USERNAME=your-username@acme.tld' \
  -e 'EXPORTER_PASSWORD=your-password' \
  -p '9898:9898' \
  ghcr.io/acoster/tado-exporter:master
```

### From sources

Optionally, you can download and build it from the sources. You have to retrieve the project sources by using one of the following way:
```bash
$ git clone https://github.com/acoster/tado-exporter
```

Then, just build the binary:

```
$ cargo build --release
```

## Usage

In order to run the exporter, type the following command (arguments are optional):

```bash
$ export EXPORTER_TICKER=10
$ export EXPORTER_USERNAME="my-username@acme.tld"
$ export EXPORTER_PASSWORD="your-password"
$ ./tado-exporter
--- tado° exporter configuration ---
Ticker seconds: 10
Username: my-username@acme.tld
Password: your-password
Client secret: wZaRN7rpjn3FoNyF5IFuxg9uMzYJcvOoQ8QWiIqS3hfk6gLhVlG57j5YNoZL2Rtc
------------------------------------
[2020-02-29T08:56:19Z INFO  tado_exporter] starting tado° exporter on address: V4(0.0.0.0:9898)
[2020-02-29T08:56:19Z INFO  tado_exporter] waiting for the first tick in 10 seconds...
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::client] retrieving zone details for Office...
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::client] retrieving zone details for Kitchen...
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::client] retrieving zone details for Living Room...
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::client] retrieving zone details for Room...
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Office -> setting temperature (celsius): 23
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Office -> setting temperature (fahrenheit): 73.4
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Office -> sensor temperature (celsius): 23.75
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Office -> sensor temperature (fahrenheit): 74.75
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Office -> sensor humidity: 40.1%
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Office -> heating power: 38%
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Kitchen -> setting temperature (celsius): 22
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Kitchen -> setting temperature (fahrenheit): 71.6
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Kitchen -> sensor temperature (celsius): 22.03
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Kitchen -> sensor temperature (fahrenheit): 71.65
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Kitchen -> sensor humidity: 42.7%
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Kitchen -> heating power: 0%
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Living Room -> setting temperature (celsius): 22
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Living Room -> setting temperature (fahrenheit): 71.6
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Living Room -> sensor temperature (celsius): 22.49
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Living Room -> sensor temperature (fahrenheit): 72.48
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Living Room -> sensor humidity: 42.2%
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Living Room -> heating power: 0%
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Room -> setting temperature (celsius): 20
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Room -> setting temperature (fahrenheit): 68
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Room -> sensor temperature (celsius): 21.42
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Room -> sensor temperature (fahrenheit): 70.56
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Room -> sensor humidity: 45.8%
[2020-02-29T08:56:30Z INFO  tado_exporter::tado::metrics] -> Room -> heating power: 0%
...
```

Once the exporter is running, you also have to update your `prometheus.yml` configuration to let it scrape the exporter:

```yaml
scrape_configs:
  - job_name: 'tado'
    static_configs:
      - targets: ['localhost:9898']
```

## Environment variables

These variables are used to configure the tool's behaviour:

| Environment variable name  | Description                                                                                     |
|:-------------------------:|--------------------------------------------------------------------------------------------------|
| `EXPORTER_USERNAME`       | Required. This represent your tado° account username/email                                       |
| `EXPORTER_PASSWORD`       | Required. This represent your tado° account password                                             |
| `EXPORTER_CLIENT_SECRET`  | Optional. This represent your tado° account client secret, using default value seems to work     |
| `EXPORTER_TICKER`         | Optional (default: 10). This represent the number of seconds the exporter will look for new data |
| `RUST_LOG`                | Optional (default: info). This describes the log level (see https://docs.rs/env_logger/)         |

## Prometheus metrics

All metrics have a `zone` label, set with the device's zone name.

| Metric name                  | Description                                                                                  |
|:----------------------------------------:|----------------------------------------------------------------------------------|
| `tado_activity_ac_power_value`           | This represent the value (1.0 = ON, 0.0 = OFF) of ac power for every zone        |
| `tado_activity_heating_power_percentage` | This represent the % of heating power for every zone                             |
| `tado_setting_temperature_value`         | This represent the current temperature you asked/programmed in a zone            |
| `tado_sensor_temperature_value`          | This represent the current temperature detected by sensor in a zone              |
| `tado_sensor_humidity_percentage`        | This represent the current humidity % detected by sensor in a zone               |
| `tado_sensor_window_opened`              | Set to `1.0` if windows in the zone have been detected as open, `0.0` otherwise  |

