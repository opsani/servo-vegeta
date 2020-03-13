# servo-vegeta

Opsani Servo measure plugin for the [Vegeta](https://github.com/tsenart/vegeta) load testing tool.

## Overview

This package provides support for executing a constant rate load test within an Opsani Servo using [Vegeta](https://github.com/tsenart/vegeta). Vegeta is a modern Golang based load testing utility supporting keep-alives, multiple target URLs, HTTP/2, and flexible reporting. It is capable delivering a very impressive level of load at a constant rate from a simple CLI utility. Metrics are reported to Opsani based on the JSON output of the `vegeta report` command.

## Configuration

`servo-vegeta` is configured as part of the `config.yaml` file deployed within a Servo assembly. Configuration is provided under the `vegeta` top-level configuration key. All options defined directly map to options for the `vegeta attack` command and links are provided to the  relevant docs on the Vegeta repository README. Required options are given in **bold** formatting.

* [**`rate`**](https://github.com/tsenart/vegeta#-rate) - Specifies the request rate per time unit to issue against the targets. Given in the format of `request/time unit`. Examples: `50/1s`, `3000/1m`. A value of `0` will cause requests to be issued as fast as possible.
* [**`duration`**](https://github.com/tsenart/vegeta#-duration) - Specifies the amount of time to issue requests to the targets. Examples: `30s`, `10m`.
* **`target`** - Specifies a single formatted Vegeta target to load against. This option is exclusive of the `target` option and will provide a target Vegeta via stdin. Default: `None`. Example: `GET https://localhost:8080/`
* [**`targets`**](https://github.com/tsenart/vegeta#-targets) - Specifies the file from which to read targets. See the `format` option to learn about available target formats. This option is exclusive of the `target` option and one or the other must be provided. Default: `stdin`. This file must be mapped into the Servo assembly container.
* [`format`](https://github.com/tsenart/vegeta#-format) - Specifies the format of the `targets` input. Valid values are `http` and `json`. Refer to the Vegeta docs for details. Default: `http`.
* [`connections`](https://github.com/tsenart/vegeta#-connections) - Specifies the maximum number of idle open connections per target host. Default: `10000`.
* [`workers`](https://github.com/tsenart/vegeta#-workers) - Specifies the initial number of workers used in the attack. The workers will automatically increase to achieve the target request rate, up to `max-workers`. Default: `10`.
* [`max-workers`](https://github.com/tsenart/vegeta#-max-workers) - The maximum number of workers used to sustain the attack. This can be used to control the concurrency of the attack to simulate a target number of clients. Default: `18446744073709551615`.
* [`max-body`](https://github.com/tsenart/vegeta/blob/master/README.md#-max-body) - Specifies the maximum number of bytes to capture from the body of each response. Remaining unread bytes will be fully read but discarded. Default: `-1`.
* [`http2`](https://github.com/tsenart/vegeta#-http2) - Specifies whether to enable HTTP/2 requests to servers which support it. Default: `true`.
* [`keepalive`](https://github.com/tsenart/vegeta#-keepalive) - Specifies whether to reuse TCP connections between HTTP requests. Default: `true`.
* [`insecure`](https://github.com/tsenart/vegeta#-insecure) - Specifies whether to ignore invalid server TLS certificates. Default: `false`.

### Example Configuration

```yaml
vegeta:
    rate: 3000/m
    duration: 10m
    target: GET https://localhost:8080/
    workers: 500
    max-workers: 5000
    interactive: true
```

### Metrics

Metrics are collected and reported from the output of a `vegeta report` invocation in JSON format. Reporting is described in detail on the [Vegeta repository](https://github.com/tsenart/vegeta#report--typetext). The following metrics are collected and reported to Opsani:

* `throughput` - Success only.
* `error_rate`
* `latencies` - Exploded from the full payload below.

All normalized to human readable. Print out the ending report to stderr. Document all the units that are being sent. In the `DESCRIBE` packet send all the explicit units.

Vegeta metrics example:

```json
{
  "latencies": {
    "total": 237119463,
    "mean": 2371194,
    "50th": 2854306,
    "90th": 3228223,
    "95th": 3478629,
    "99th": 3530000,
    "max": 3660505,
    "min": 1949582
  },
  "buckets": {"0":9952,"1000000":40,"2000000":6,"3000000":0,"4000000":0,"5000000":2},
  "bytes_in": {
    "total": 606700,
    "mean": 6067
  },
  "bytes_out": {
    "total": 0,
    "mean": 0
  },
  "earliest": "2015-09-19T14:45:50.645818631+02:00",
  "latest": "2015-09-19T14:45:51.635818575+02:00",
  "end": "2015-09-19T14:45:51.639325797+02:00",
  "duration": 989999944,
  "wait": 3507222,
  "requests": 100,
  "rate": 101.01010672380401,
  "throughput": 101.00012489812,
  "success": 1,
  "status_codes": {
    "200": 100
  },
  "errors": []
}
```

## Testing

1. Create `config.yaml` file with load configuration and `target`.
2. Install dependencies via [Poetry](https://github.com/python-poetry/poetry): `$ poetry install`
3. Run the plugin standalone: `$ echo '{}' | poetry run ./measure any-string`

## License

BSD 3-Clause. See `LICENSE` file.
