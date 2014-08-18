# fluent-agent-hydra

A Fluentd log agent.

[![Build status](https://api.travis-ci.org/fujiwara/fluent-agent-hydra.svg?branch=master)](https://travis-ci.org/fujiwara/fluent-agent-hydra)

This agent is inspired by [fluent-agent-lite](https://github.com/tagomoris/fluent-agent-lite).

## Features

- Tailing log files (like in_tail)
-- enable to handle multiple files in a single process.
- Forwarding messages to external fluentd (like out_forward)
-- multiple fluentd server can be used. When primary server is down, messages will sent to secondary server.
- Receiving a fluentd's forward protocol messages via TCP (like in_forward)
-- includes simplified on-memory queue.
- Stats monitor httpd server
-- serve an agent stats by JSON format.

## Installation

```
go get github.com/fujiwara/fluent-agent-hydra/cmd/fluent-agent-hydra/
```

## Usage

### Using command line arguments

```
fluent-agent-hydra [options] TAG TARGET_FILE PRIMARY_SERVER SECONDARY_SERVER
```

Options

* -f="message": fieldname of fluentd log message attribute (DEFAULT: message)
* -monitor="127.0.0.1:24223": monitor httpd daemon address (DEFAULT: -)

Usage example

```
fluent-agent-hydra -f msg tagname /path/to/foo.log fluentd.example.com:24224 127.0.0.1:24224
```

* Field name: "msg"
* Filename: "/path/to/foo.log"
* Primary server: "fluentd.example.com:24224"
* Secondary server: "127.0.0.1:24224"

### Using configuration file

```
fluent-agent-hydra -c /path/to/config.toml
```

A example of config.toml

```toml
TagPrefix = "foo"     # "foo.tag1", "foo.tag2"
FieldName = "message" # default

# tailing log file (in_tail)
[[Logs]]
Tag  = "tag1"
File = "/path/to/foo.log"

[[Logs]]
Tag  = "tag2"
File = "/path/to/bar.log"
FieldName = "msg"

# forwarding fluentd server (out_forward)
[[Servers]]
Host = "fluentd.example.com"
Port = 24224

[[Servers]]
Host = "fluentd-backup.example.com"
Port = 24225

# receive fluentd forward protocol daemon (in_forward)
[Receiver]
Host = "127.0.0.1"
Port = 2422

# stats monitor http daemon
[Monitor]
Host = "127.0.0.1"
Port = 24223
```

## Stats monitor

`curl -s [Monitor.Host]:[Monitor.Port] | jq .`

example response.

```json
{
  "receiver": {
    "buffered": 0,
    "disposed": 0,
    "messages": 123,
    "current_connections": 1,
    "total_connections": 10,
    "address": "[::]:24224"
  },
  "servers": [
    {
      "error": "",
      "alive": true,
      "address": "192.168.1.11:24224"
    },
    {
      "error": "[2014-08-18 18:15:19.121265262 +0900 JST] dial tcp 192.168.1.12:24224: i/o timeout",
      "alive": false,
      "address": "192.168.1.12:24224"
    }
  ],
  "files": {
    "/var/log/nginx/error.log": {
      "error": "",
      "position": 95039,
      "tag": "nginx.error"
    },
    "/var/log/nginx/access.log": {
      "error": "",
      "position": 112093,
      "tag": "nginx.access"
    }
  },
  "sent": {
    "nginx.error": {
      "bytes": 1050,
      "messages": 5
    },
    "nginx.access": {
      "bytes": 19480,
      "messages": 48
    }
  }
}
```

## Thanks to

* `fluent/fluent.go, utils.go` imported and modified from [github.com/t-k/fluent-logger-golang](https://github.com/t-k/fluent-logger-golang).
* `fluent/server.go` imported and modified from [github.com/moriyoshi/ik](https://github.com/moriyoshi/ik/).

## Author

Fujiwara Shunichiro <fujiwara.shunichiro@gmail.com>

## Licence

Copyright 2014 Fujiwara Shunichiro.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
