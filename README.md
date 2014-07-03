prom2json
=========

A tool to scrape a Prometheus client and dump the result as JSON.

# Background

(Pre-)historically, Prometheus clients were able to expose metrics as
JSON. For various reasons, the JSON exposition format was deprecated.

Usually, scraping of a Prometheus client is done by the Prometheus
server, which preferably happens with the protocol buffer
format. Sometimes, a human being needs to inspect what a Prometheus
clients exposes. In that case, the text format is used (which is
otherwise meant to allow simplistic _clients_ like shell scripts to
expose metrics to a Prometheus _server_).

However, some users wish to scrape Prometheus clients with programs
other than the Prometheus server. Those programs would usually use the
protocol buffer format, but for small _ad hoc_ programs, that is too
much of an (programming) overhead. JSON comes in handy for these
use-cases, as many languages offer tooling for JSON parsing.

To avoid maintaining a JSON format in all client libraries, the
`prom2json` tool has been created, which scrapes a Prometheus client
in protocol buffer or text format and dumps the result as JSON to
`stdout`.

# Usage

Installing and building:

    $ go get github.com/prometheus/prom2json
    $ go install github.com/prometheus/prom2json

Running:

    $ prom2json http://my-prometheus-client.example.org:8080/metrics

This will dump the JSON to `stdout`. Note that the dumped JSON is
_not_ using the deprecated JSON format as specified in the
[Prometheus exposition format
reference](https://docs.google.com/document/d/1ZjyKiKxZV83VI9ZKAXRGKaUKK2BIWCT7oiGBKDBpjEY/edit?usp=sharing). The
created JSON uses a format much closer in structure to the procotol
buffer format. It is only used by the `prom2json` tool and has no
significance elsewhere. See below for a description.

A typical use-case is to pipe the JSON format into a tool like `jq` to
run a query over it. That looked like the following when the clients
still supported the deprecated JSON format:

    $ curl http://my-prometheus-client.example.org:8080/metrics | jq .

Now simply use `prom2json` instead of `curl` (and change the query
syntax according to the changed JSON format generated by `prom2json`):

    $ prom2json http://my-prometheus-client.example.org:8080/metrics | jq .

TODO: come up with a nifty query, not just ".".

# JSON format

```json
[
  {
    "name": "http_request_duration_microseconds",
    "help": "The HTTP request latencies in microseconds.",
    "type": "SUMMARY",
    "metrics": [
      {
        "labels": {
          "method": "get",
          "handler": "prometheus",
          "code": "200"
        },
        "quantiles": {
          "0.99": 67542.292,
          "0.9": 23902.678,
          "0.5": 6865.718
        },
        "count": 743,
        "sum": 6936936.447000001
      },
      {
        "labels": {
          "method": "get",
          "handler": "prometheus",
          "code": "400"
        },
        "quantiles": {
          "0.99": 3542.9,
          "0.9": 1202.3,
          "0.5": 1002.8
        },
        "count": 4,
        "sum": 345.01
      }
    ]
  },
  {
    "name": "roshi_select_call_count",
    "help": "How many select calls have been made.",
    "type": "COUNTER",
    "metrics": [
      {
        "value": 1063110
      }
    ]
  }
]
```