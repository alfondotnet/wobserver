# Wobserver

[![Hex.pm](https://img.shields.io/hexpm/v/wobserver.svg "Hex")](https://hex.pm/packages/wobserver)
[![Hex.pm](https://img.shields.io/badge/docs-v0.1.4-brightgreen.svg "Docs")](https://hexdocs.pm/wobserver)
[![Hex.pm](https://img.shields.io/hexpm/l/wobserver.svg "License")](LICENSE)

Web based metrics, monitoring, and observer.

<a href="http://imgur.com/a/IqO6O" target="_blank"><img src="http://i.imgur.com/BkwHIpv.gif" alt="Click to see more images." width="210" height="250" /></a>

[_Click to view images_](http://imgur.com/a/IqO6O)

**Functionality:**

* Drop-in monitoring though web interface.
* Metrics endpoint (`/metrics`) for system monitoring.
  (Default: [Prometheus](https://prometheus.io/))
* Monitoring automation through JSON API.
* Node management and discovery behind firewalls and load balancers.
* Easy to extend:
    * Add custom metrics and pages for your project, just by adding them in the config.
    * Just 3 lines of code to add pages/metrics for your library, when users have `:wobserver` installed.
      (See [how](#library-integration).)

## Table of contents

* [wobserver](#wobserver)
* [Table of contents](#table-of-contents)
* [Installation](#installation)
    * [Hex](#hex)
    * [Build manually](#build-manually)
* [Usage](#usage)
    * [Browser](#browser)
    * [API](#api)
        * [Remote nodes](#remote-nodes)
        * [System](#system)
        * [Allocators](#allocators)
        * [Application](#application)
        * [Process](#process)
        * [Ports](#ports)
        * [Table view](#table-view)
    * [Metrics](#usage-metrics)
* [Configuration](#configuration)
    * [Port](#port)
    * [Mode](#mode)
        * [Standalone](#mode-standalone)
        * [Plug](#mode-plug)
    * [Node Discovery](#node-discovery)
    * [Metrics](#configure-metrics)
        * [Add Metrics](#add-metrics)
        * [Formatting](#formatting-metrics)
    * [Pages](#pages)
        * [Config](#pages-app)
        * [Dynamically](#pages-dynamic)
* [Library Integration](#library-integration)
* [Improvements](#improvements)
* [license](#License)


## Installation

### Hex

Add Wobserver as a dependency to your `mix.exs` file:

```elixir
def deps do
  [{:wobserver, "~> 0.1"}]
end
```

and add it to your list of applications:

```elixir
def application do
  [applications: [:wobserver]]
end
```

Then run `mix deps.get` in your shell to fetch the dependencies.

__Note:__ Check out [plug mode](#mode-plug) to integrate with a *Phoenix* or other web application. (Prevents startup of separate web server.)

### Build manually

Run the following commands to build the project:
```bash
$ npm install
$ gulp build
$ mix deps.get
```

## Usage

### Browser
To view the web interface just enter `http://<host>[:<port>]/` in the browser and it should show the `:wobserver` interface.
The default port is 4001, but the port can be changed in the configuration.

A sample interface can be viewed [here](http://imgur.com/a/IqO6O).

### API

The API can be accessed by calling `http://<host>[:<port>]/api/`.
The index will return `404`, but specific endpoints should return results.

#### <a name="remote-nodes"></a> Remote nodes

The API provides a list of remote nodes by calling `http://<host>[:<port>]/api/nodes`.

The API of remote nodes can be accessed by calling the API endpoint and prefixing the node name, host, or host:port.

For example considering the following node list:
```json
[
  {
    "port": 4001,
    "name": "node_prime",
    "local?": true,
    "host": "192.168.5.55"
  },
  {
    "port": 80,
    "name": "remote",
    "local?": false,
    "host": "80.23.1.165"
  }
]
```

The following calls would all work for the first node:
*(`local` is a reserved name that always points to the local node.*)
```bash
http://<host>[:<port>]/api/local/system
http://<host>[:<port>]/api/node_prime/system
http://<host>[:<port>]/api/192.168.5.55/system
http://<host>[:<port>]/api/192.168.5.55:4001/system
```

And these calls would work for the second node:
```bash
http://<host>[:<port>]/api/remote/system
http://<host>[:<port>]/api/80.23.1.165/system
http://<host>[:<port>]/api/80.23.1.165:80/system
```

#### <a name="system"></a> System
The API provides a list of system information by calling `http://<host>[:<port>]/api/system`.

The `scheduler` is a list of load values (0-1) for each scheduler.

Example:
```json
{
  "statistics": {
    "uptime": 459876,
    "process_total": 122,
    "process_running": 0,
    "process_max": 262144,
    "output": 1259201,
    "input": 12945380
  },
  "scheduler": [
    0.0037370416873916392,
    0.0003088661849770247,
    0.0003072993680801981,
    0.00030274231847091137,
    0.0004706952361156354,
    0.00028556537348788645,
    0.00025471141618606366,
    0.0002522242536713918
  ],
  "memory": {
    "total": 30275576,
    "process": 5242800,
    "ets": 886544,
    "code": 13635797,
    "binary": 288744,
    "atom": 594561
  },
  "cpu": {
    "schedulers_online": 8,
    "schedulers_available": 8,
    "schedulers": 8,
    "logical_processors_online": 8,
    "logical_processors_available": "unknown",
    "logical_processors": 8
  },
  "architecture": {
    "wordsize_internal": 8,
    "wordsize_external": 8,
    "threads": true,
    "thread_pool_size": 10,
    "system_architecture": "x86_64-apple-darwin15.6.0",
    "smp_support": true,
    "otp_release": "19",
    "kernel_poll": false,
    "erts_version": "8.2",
    "elixir_version": "1.4.0"
  }
}
```

#### <a name="allocators"></a> Allocators
The API provides a list of allocators and their size by calling `http://<host>[:<port>]/api/allocators`.

Example:
```json
[
  {
    "type": "sl_alloc",
    "carrier": 294912,
    "block": 664
  },
  {
    "type": "std_alloc",
    "carrier": 1081344,
    "block": 498184
  },
  {
    "type": "ll_alloc",
    "carrier": 35913728,
    "block": 26080144
  },
  {
    "type": "eheap_alloc",
    "carrier": 9830400,
    "block": 2634720
  },
  {
    "type": "ets_alloc",
    "carrier": 3178496,
    "block": 890880
  },
  ...
]
```

#### <a name="application"></a> Application
The API provides a list of applications and their descriptions by calling `http://<host>[:<port>]/api/application`.

The information for a specific application, including the process hierarchy can be found by calling `http://<host>[:<port>]/api/application/<application-name>`.

Example:
`http://localhost:4001/api/application`
```json
[
  {
    "version": "0.1.0",
    "name": "wobserver",
    "description": "Web based metrics, monitoring, and observer."
  },
  {
    "version": "1.3.0",
    "name": "plug",
    "description": "A specification and conveniences for composable modules between web applications"
  },
  {
    "version": "1.1.0",
    "name": "cowboy",
    "description": "Small, fast, modular HTTP server."
  },
  {
    "version": "1.2.1",
    "name": "ranch",
    "description": "Socket acceptor pool for TCP protocols."
  },
  {
    "version": "0.6.1",
    "name": "credo",
    "description": "A static code analysis tool for the Elixir language with a focus on code consistency and teaching."
  },
  {
    "version": "0.2.0",
    "name": "bunt",
    "description": "256 color ANSI coloring in the terminal"
  },
  {
    "version": "1.6.5",
    "name": "hackney",
    "description": "simple HTTP client"
  },
  {
    "version": "1.4.0",
    "name": "logger",
    "description": "logger"
  },
  ...
]
```
`http://localhost:4001/api/application/elixir`
```json
{
  "pid": "#PID<0.59.0>",
  "name": "<0.59.0>",
  "meta": {
    "status": "waiting",
    "init": "proc_lib.init_p/5",
    "current": "application_master.main_loop/2",
    "class": "application"
  },
  "children": [
    {
      "pid": "#PID<0.60.0>",
      "name": "<0.60.0>",
      "meta": {
        "status": "waiting",
        "init": "application_master.start_it/4",
        "current": "application_master.loop_it/4",
        "class": "unknown"
      },
      "children": [
          {
            "pid": "#PID<0.61.0>",
            "name": "elixir_sup",
            "meta": {
              "status": "waiting",
              "init": "proc_lib.init_p/5",
              "current": "gen_server.loop/6",
              "class": "supervisor"
            },
            "children": [
              {
              "pid": "#PID<0.62.0>",
              "name": "elixir_config",
              "meta": {
                "status": "waiting",
                "init": "proc_lib.init_p/5",
                "current": "gen_server.loop/6",
                "class": "gen_server"
              },
              "children": []
            },
            {
              "pid": "#PID<0.63.0>",
              "name": "elixir_code_server",
              "meta": {
                "status": "waiting",
                "init": "proc_lib.init_p/5",
                "current": "gen_server.loop/6",
                "class": "gen_server"
              },
              "children": []
            }
          ]
        }
      ]
    }
  ]
}
```

#### <a name="process"></a> Process
The API provides a list of processes and their basic information by calling `http://<host>[:<port>]/api/process`.

The information for a specific process, including a links, memory usage, and state can be found by calling `http://<host>[:<port>]/api/application/<process-name>`.

The process name can be given as pid, name, or short pid.

So all the following are valid:
```bash
http://localhost:4001/api/process/<0.247.0>
http://localhost:4001/api/process/#PID<0.247.0>   # Rememeber to url encode # -> %23
http://localhost:4001/api/process/Wobserver.Supervisor
```

Example:
`http://localhost:4001/api/process`
```json
{
  "processes": [
    {
      "reductions": 162714,
      "pid": "#PID<0.247.0>",
      "nr1": "0",
      "message_queue_length": 0,
      "memory": 11888,
      "init": "timer_server",
      "current": "gen_server.loop/6"
    },
    {
      "reductions": 95,
      "pid": "#PID<0.243.0>",
      "nr1": "0",
      "message_queue_length": 0,
      "memory": 2792,
      "init": "erlang.apply/2",
      "current": "io.execute_request/2"
    },
    {
      "reductions": 954,
      "pid": "#PID<0.242.0>",
      "nr1": "0",
      "message_queue_length": 0,
      "memory": 16808,
      "init": "Elixir.IEx.Evaluator.init/4",
      "current": "Elixir.IEx.Evaluator.loop/3"
    },
    ...
  ]
}
```
`http://localhost:4001/api/process/<0.247.0>`
```json
{
  "trap_exit": true,
  "state": "[]",
  "relations": {
    "links": [
      "#PID<0.53.0>"
    ],
    "group_leader": "#PID<0.33.0>",
    "ancestors": [
      "kernel_safe_sup",
      "kernel_sup",
      "#PID<0.34.0>"
    ]
  },
  "registered_name": "timer_server",
  "priority": "normal",
  "pid": "#PID<0.247.0>",
  "meta": {
    "status": "waiting",
    "init": "proc_lib.init_p/5",
    "current": "gen_server.loop/6",
    "class": "gen_server"
  },
  "message_queue_len": 0,
  "memory": {
    "total": 0,
    "stack_size": 9,
    "stack_and_heap": 1974,
    "heap_size": 1598,
    "gc_min_heap_size": 233,
    "gc_full_sweep_after": 65535
  },
  "error_handler": "error_handler"
}
```

#### <a name="ports"></a> Ports
The API provides a list of ports and their owners by calling `http://<host>[:<port>]/api/ports`.

Example:
`http://localhost:4001/api/ports`
```json
[
  {
    "output": 0,
    "os_pid": "undefined",
    "name": "forker",
    "links": [],
    "input": 0,
    "id": 0,
    "connected": "#PID<0.0.0>"
  },
  {
    "output": 3,
    "os_pid": "undefined",
    "name": "efile",
    "links": [
      "#PID<0.4.0>"
    ],
    "input": 46,
    "id": 8,
    "connected": "#PID<0.4.0>"
  },
  {
    "output": 18810,
    "os_pid": "undefined",
    "name": "efile",
    "links": [
      "#PID<0.44.0>"
    ],
    "input": 23874,
    "id": 4680,
    "connected": "#PID<0.44.0>"
  },
  ...
]
```

#### <a name="table-view"></a> Table view
The API provides a list of tables and their details by calling `http://<host>[:<port>]/api/table`.

The information for a specific details, including a the actual data can be found by calling `http://<host>[:<port>]/api/table/<table-name>`.

Example:
`http://localhost:4001/api/table`
Example:
```json
[
  {
    "type": "set",
    "size": 0,
    "protection": "protected",
    "owner": "#PID<0.247.0>",
    "name": "timer_interval_tab",
    "meta": {
      "write_concurrency": false,
      "read_concurrency": false,
      "compressed": false
    },
    "memory": 304,
    "id": "timer_interval_tab"
  },
  {
    "type": "ordered_set",
    "size": 7,
    "protection": "protected",
    "owner": "#PID<0.247.0>",
    "name": "timer_tab",
    "meta": {
      "write_concurrency": false,
      "read_concurrency": false,
      "compressed": false
    },
    "memory": 304,
    "id": "timer_tab"
  },
  {
    "type": "set",
    "size": 7,
    "protection": "public",
    "owner": "#PID<0.228.0>",
    "name": "workstore",
    "meta": {
      "write_concurrency": false,
      "read_concurrency": false,
      "compressed": false
    },
    "memory": 5138,
    "id": 417840
  },
  ...
]
```
`http://localhost:4001/api/table/timer_interval_tab`
```json
{
  "type": "set",
  "size": 0,
  "protection": "protected",
  "owner": "#PID<0.247.0>",
  "name": "timer_interval_tab",
  "meta": {
    "write_concurrency": false,
    "read_concurrency": false,
    "compressed": false
  },
  "memory": 304,
  "id": "timer_interval_tab",
  "data": []
}

```

### Metrics
Metrics are available by calling `http://<host>[:<port>]/metrics`.

The metrics are by default formatted for [Prometheus](https://prometheus.io/), but can be configured to work with any system.
An explanation of how to configure the metrics format and how to add metrics to the output will be added later.

`http://localhost:4001/metrics`
```bash
# HELP erlang_vm_used_memory_bytes Memory usage of the Erlang VM.
# TYPE erlang_vm_used_memory_bytes gauge
erlang_vm_used_memory_bytes{node="10.74.181.35",type="atom"} 553593
erlang_vm_used_memory_bytes{node="10.74.181.35",type="binary"} 359552
erlang_vm_used_memory_bytes{node="10.74.181.35",type="code"} 13533686
erlang_vm_used_memory_bytes{node="10.74.181.35",type="ets"} 1899472
erlang_vm_used_memory_bytes{node="10.74.181.35",type="process"} 6048552
# HELP erlang_vm_used_io_bytes IO counter for the Erlang VM.
# TYPE erlang_vm_used_io_bytes counter
erlang_vm_used_io_bytes{node="10.74.181.35",type="input"} 11301316
erlang_vm_used_io_bytes{node="10.74.181.35",type="output"} 618157
```

## Configuration
### Port
The port can be set in the config by setting `:port` for `:wobserver` to a valid number.

#### Example config
```elixir
config :wobserver,
  port: 80
```

### Mode
Wobserver runs by default in `:standalone` mode.
This means that `:wobserver` will start its own `:cowboy` listeners on a separate port.
Standalone mode is ideal for drop-in web viewing, but might not be ideal if another part of the application is already running an web server.
It is possible to enable `:plug` mode to prevent `:wobserver` from starting `:cowboy` to handle web requests.

#### <a name="mode-standalone"></a> Standalone
Standalone mode is the default operating mode.
A `:cowboy` (ranch) listener will be started with 10 accepters and a websocket.
Set `mode` to `:standalone` in the `:wobserver` configuration to force standalone mode.

Example:
```elixir
config :wobserver,
  mode: :standalone
```

#### <a name="mode-plug"></a> Plug
Plug mode prevents `:wobserver` from starting `:cowboy` (ranch).
Set `mode` to `:plug` in the `:wobserver` configuration to use plug mode.

Add the following line of code to the application's router to forward requests to `:wobserver`:
```elixir
  forward "/wobserver", to: Wobserver.Web.Router
```

Add the following option to you `:cowboy` child_spec to enable use of the `:wobserver` websocket:
```elixir
dispatch: [
    {:_, [
      {"/ws", Wobserver.Web.Client, []},
      {:_, Cowboy.Handler, {<your own router>, []}}
    ]}
  ],
```

Example:

__config.exs__
```elixir
config :wobserver,
  mode: :plug
```
__my_router.ex__
```elixir
defmodule MyApp.MyRouter do
  use Plug.Router

  plug :match
  plug :dispatch

  forward "/wobserver", to: Wobserver.Web.Router
end
```
__application.ex__
```elixir
defmodule MyApp.Application do
  use Application

  alias Plug.Adapters.Cowboy

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    options = [
      dispatch: [
        {:_, [
          {"/wobserver/ws", Wobserver.Web.Client, []},
          {:_, Cowboy.Handler, {MyApp.MyRouter, []}}
        ]}
      ],
    ]

    children = [
      Cowboy.child_spec(:http, MyApp.MyRouter, [], options)
    ]

    opts = [strategy: :one_for_one, name: MyApp.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```


### Node Discovery
The method used can be set in the config file by setting:
```elixir
config :wobserver,
  discovery: :none
```

The following methods can be used: (default: `:none`)

  - `:none`, just returns the local node.
  - `:dns`, use DNS to search for other nodes.
    The option `discovery_search` needs to be set to filter entries.
  - `:custom`, a function as String.

#### Example config
No discovery: *(default)*
```elixir
config :wobserver,
  port: 80,
  discovery: :none
```

Using dns as discovery service:
```elixir
config :wobserver,
  port: 80,
  discovery: :dns,
  discovery_search: "google.nl"
```

Using a custom function:
```elixir
config :wobserver,
  port: 80,
  discovery: :custom,
  discovery_search: &MyApp.CustomDiscovery.discover/0
```

Using an anonymous function:
```elixir
config :wobserver,
  port: 80,
  discovery: :custom,
  discovery_search: fn -> [] end
```

Both the custom and anonymous functions can be given as a String, which will get evaluated.

### <a name="configure-metrics"></a> Metrics
#### <a name="add-metrics"></a> Add Metrics
#### Config
Metrics and metric generators can be added by setting them in the configuration.

To add custom metrics set the `:metrics` option.
The `:metrics` option must be a keyword list with the following keys:
  * `additional`, for a keyword list with additional metrics.
  * `generators`, for a list of metric generators.

The following settings are accepted for `additional`:
  - `keyword` list, the key is the name of the metric and the value is the metric data.

The following inputs are accepted for metric generators:
  - `list` of callable functions.
    Every function should return a keyword list with as key the name of the metric and as value the metric data.

For more information about how to format metric data see: [`Wobserver.Util.Metrics.Formatter.format_all/1`](https://hexdocs.pm/wobserver/Wobserver.Util.Metrics.Formatter.html#format_all/1).

For example this configuration:
```elixir
config :wobserver,
  metrics: [
    additional: [
      example: {fn -> [red: 5] end, :gauge, "Description"},
    ],
    generators: [
      "&MyApp.generator/0",
      fn -> [bottles: {fn -> [wall: 8, floor: 10] end, :gauge, "Description"}] end
      fn -> [server: {"MyApp.Server.metrics/0", :gauge, "Description"}] end
    ]
  ]
```

#### Dynamically
Metrics and metric generators can also be added dynamically at runtime.

To register a metric you need to pass a keyword list to `Wobserver.register` with the same data as you would set in the configuration file.

For example:
```elixir
Wobserver.register :metric, [example: {fn -> [red: 5] end, :gauge, "Description"}]
```

To register a metric generator you need to pass a list of functions to `Wobserver.register`.

For example:
```elixir
Wobserver.register :metric, [&MyLibrary.Metrics.generate/0]
```

#### <a name="formatting-metrics"></a> Formatting
A custom formatter can be created for output of metrics by implementing the `Wobserver.Util.Metrics.Formatter` behavior.
This custom formatter can be enabled in the configuration file by setting `metric_format`.

For example this configuration:
```elixir
config :wobserver,
  metric_format: JsonFormatter
```

And this simple JSON formatter:
```elixir
defmodule SimpleJsonFormatter do
  @behaviour Wobserver.Util.Metrics.Formatter

  def format_data(name, data, type, help) do
    formatted_data =
      data
      |> Enum.map(fn {value, labels} ->
           %{value: value, labels: Enum.into(labels, %{})}
         end)

    %{
      name: name,
      type: type,
      description: help,
      data: formatted_data
    }
    |> Poison.encode!
  end

  def combine_metrics(metrics) do
    "[" <> Enum.join(metrics,",") <> "]"
  end

  def merge_metrics(metrics) do
    "[" <> Enum.join(metrics,",") <> "]"
  end
end
```

Produce the following output:
```json
[
  [
    {
      "type": "gauge",
      "name": "erlang_vm_used_memory_bytes",
      "description": "Memory usage of the Erlang VM.",
      "data": [
        {
          "value": 654241,
          "labels": {
            "type": "atom",
            "node": "192.168.1.88"
          }
        },
          {
          "value": 503464,
          "labels": {
            "type": "binary",
            "node": "192.168.1.88"
          }
        },
        {
          "value": 14459399,
          "labels": {
            "type": "code",
            "node": "192.168.1.88"
          }
        },
        {
          "value": 2073072,
          "labels": {
            "type": "ets",
            "node": "192.168.1.88"
          }
        },
        {
          "value": 6008488,
          "labels": {
            "type": "process",
            "node": "192.168.1.88"
          }
        }
      ]
    },
    {
      "type": "counter",
      "name": "erlang_vm_used_io_bytes",
      "description": "IO counter for the Erlang VM.",
      "data": [
        {
          "value": 29523254,
          "labels": {
            "type": "input",
            "node": "192.168.1.88"
          }
        },
        {
          "value": 9960593,
          "labels": {
            "type": "output",
            "node": "192.168.1.88"
          }
        }
      ]
    }
  ]
]
```

### <a name="pages"></a> Pages
Pages are custom views in the web interface and endpoints in the JSON API for an application or library.

There are two ways to add a custom page:
* *config*, set a list of custom pages in the mix config.
* *registration*, call `Wobserver.register/2` and dynamically add pages.

#### <a name="pages-config"></a> Config
Adding more pages to `:wobserver` can be done by setting the `:pages` option.

The `:pages` option must be a list of page data.

The page data can be formatted as:
  * `{title, command, callback}`
  * `{title, command, callback, options}`
  * a `map` with the following fields:
      * `title`
      * `command`
      * `callback`
      * `options` (optional)

For more information and types see: [`Wobserver.Page.register/1`](https://hexdocs.pm/wobserver/Wobserver.Page.html#register/1).

Example:
```elixir
config :wobserver,
  pages: [
    {"Example", :example, fn -> %{x:  9} end}
  ]
```
#### <a name="pages-dynamic"></a> Dynamically
Dynamically register a page with `:wobserver` by calling [`Wobserver.register/2`](https://hexdocs.pm/wobserver/Wobserver.html#register/2).

The following inputs are accepted:
  * `{title, command, callback}`
  * `{title, command, callback, options}`
  * a `map` with the following fields:
      * `title`
      * `command`
      * `callback`
      * `options` (optional)

The fields are used as followed:
  * `title`, the name of the page. Is used for the web interface menu.
  * `command`, single atom to associate the page with.
  * `callback`, function to be evaluated, when the a api is called or page is viewd.
                The result is converted to JSON and displayed.
  * `options`, options for the page.

The following options can be set:
  * `api_only` (`boolean`), if set to true the page won't show up in the web interface, but will only be available as API.
  * `refresh` (`float`, 0-1), sets the refresh time factor. Used in the web interface to refresh the data on the page. Set to `0` for no refresh.

Example:
```elixir
Wobserver.register(:page, {"My App", :my_app, fn -> %{data: 123} end})
```

## Library Integration
Integrating a library with `:wobserver` is done by calling [`Wobserver.register/2`](https://hexdocs.pm/wobserver/Wobserver.html#register/2), when the library loads, and dynamically adding pages and metrics.

### Code
To safely integrate with `:wobserver` use the following code:
```elixir
if Code.ensure_loaded(Wobserver) == {:module, Wobserver} do
  Wobserver.register :page, {"My Library", :my_library, fn -> %{data: 123} end}
  Wobserver.register :metric, [&MyLibrary.Metrics.generate/0]
end
```
The above code will make sure that the library only calls register, when `:wobserver` is loaded.
This will prevent the library from trying to register, when `:wobserver` is not installed.

For an implementation see the `:task_bunny` library: [TaskBunny](https://github.com/shinyscorpion/task_bunny), [`lib/task_bunny.ex`](https://github.com/shinyscorpion/task_bunny/blob/master/lib/task_bunny.ex).

### Remove Warnings
The above code will generate warnings while compiling the library.
```bash
warning: function Wobserver.register/2 is undefined (module Wobserver is not available)
```

There are two options to remove those warnings.

#### Edit mix.exs
The `mix.exs` file can be edited to excluded `Wobserver` from reference checks.

To do this add the following line to `project/0` in your mix file:
```elixir
  xref: [exclude: [Wobserver]]
```

#### Kernel.apply
The code can be rewritten to use [`Kernel.apply/3`](https://hexdocs.pm/elixir/Kernel.html#apply/3).
The following code will be less readable and slightly slower, but will not generate warnings.
```elixir
if Code.ensure_loaded(Wobserver) == {:module, Wobserver} do
  apply Wobserver, :register, [:page, {"My Library", :my_library, fn -> %{data: 123} end}]
  apply Wobserver, :register, [:metric, [&MyLibrary.Metrics.generate/0]]
end
```

## Improvements
  - Cleanup namespaces.
  - Cleanup readme, condense sample output.
  - Overhaul web interface (make fancier/pleasant)

## License

Wobserver source code is released under [the MIT License](LICENSE).
Check [LICENSE](LICENSE) file for more information.
