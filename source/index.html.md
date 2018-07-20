---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby
  - python
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Contagion is a forward genetic simulator designed to model evolution occurring at multiple frames of reference, such as the evolution of viruses and other pathogens.

There are two ways to set-up a simulation in Contagion: writing down simulation parameters to a configuration file or interactively set-up the simulation using the Contagion command-line interface (Contagion CLI). 

Below you can find instructions how to set-up and run simulations in Contagion either by using a configuration file or the Contagion CLI. You can view configuration examples in the dark area to the right, and you can switch between the configuration file or the Contagion CLI guide using the tabs in the top right.

Contagion is free and open-source software written in Go and is distributed under the Apache License. For specific scenarios not currently implemented in the program, users are encouraged to use Contagion as the basis for their simulations. Contagion is built as a set of functional modules that can be readily adapted for many different simulation objectives. For more information about using Contagion as a Go package, head over to the [Contagion GoDoc](https://godoc.org/github.com/kentwait/contagion) page.

# Quickstart

> Use this configuration to create an SIR simulation without mutation in a fully-connected population of 10 hosts.

```toml
[simulation]
num_generations = 100
num_instances = 1
host_popsize = 10
epidemic_model = "sir"
coinfection = false
pathogen_path = "examples/pathogens.quickstart.fa"
host_network_path = "examples/net.quickstart.txt"
num_sites = 1
expected_characters = ["U", "P"]

[logging]
log_freq = 1
log_path = "examples/run.log"

[[intrahost_model]]
model_name = "no_mutation_no_selection"
host_ids = [0,1,2,3,4,5,6,7,8,9]
mutation_rate = 0.0
transition_matrix = [
  [ 0.0e+00, 0.0e+00 ],
  [ 0.0e+00, 0.0e+00 ],
]
recombination_rate = 0.0
replication_model = "constant"
max_pop_size = 1000000
infected_duration = 10

[[fitness_model]]
model_name = "additive"
host_ids = [
 0
]
fitness_model = "additive"
fitness_model_path = "examples/fitpop/fm.txt"

[[transmission_model]]
model_name = "poisson"
host_ids = [
 0
]
mode = "poisson"
transmission_prob = 0.0
transmission_size = 10.0

# [[stop_condition]]
# condition = "allele_fixloss"
# sequence = "P"
# position = 0
```

```shell
```

# Configuration

The configuration file is the main method to set-up and control the behavior of a simulation. The configuration file is divided into different sections grouped by the role of the parameters. 

There are two types of parameters in Contagion - global and local parameters. Global parameters affect the entire simulation, such as the size and the duration of the simulation, and how the data generated from the simulation is logged. Global parameters can only be declared once and have only one constant value for the entire simulation. Parameters in the `simulation` and the `logging` sections of the configuration file are global parameters.

Parameters that would only affect a part of the simulation are called local parameters. For example, you want to simulate the evolution of pathogens across two different host species where viral mutation rate is higher in one compared to the other. Under this scenario, the population of hosts in the simulation need to be divided into two groups and each group requires a distinct viral mutation rate parameter. Thus, mutation rate is a local parameter that can be configured to have multiple values, each affecting only a particular subset of the population. Parameters in the `intrahost_model`, `fitness_model`, `transmission_model`, and `stop_condition` sections are local parameters. Note that while local parameters can be set to affect only a portion of the simulation, these parameters can also be used to assign a global behavior to the simulation. 

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

