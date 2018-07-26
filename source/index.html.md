---
title: Contagion User Guide

language_tabs: # must be one of https://git.io/vQNgJ
  - toml
  - shell

toc_footers:
  - <a href='https://github.com/kentwait/contagion'>Download Contagion</a>
  - <a href="https://twitter.com/kent">@kent</a>
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

# Contagion CLI


# Configuration file

The configuration file is the main method to set-up and control the behavior of a simulation. The configuration file follows the TOML specification and is divided into different sections grouped by the role of the parameters. 

There are two types of parameters in Contagion - global and local parameters. Global parameters affect the entire simulation, such as the size and the duration of the simulation, and how the data generated from the simulation is logged. Global parameters can only be declared once and have only one constant value for the entire simulation. Parameters in the `simulation` and the `logging` sections of the configuration file are global parameters.

Parameters that would only affect a part of the simulation are called local parameters. For example, you want to simulate the evolution of pathogens across two different host species where viral mutation rate is higher in one compared to the other. Under this scenario, the population of hosts in the simulation need to be divided into two groups and each group requires a distinct viral mutation rate parameter. Thus, mutation rate is a local parameter that can be configured to have multiple values, each affecting only a particular subset of the population. Parameters in the `intrahost_model`, `fitness_model`, `transmission_model`, and `stop_condition` sections are local parameters. Note that while local parameters can be set to affect only a portion of the simulation, these parameters can also be used to assign a global behavior to the simulation. 

# Simulation parameters

> Parameters in the simulation section:

```toml
[simulation]

epidemic_model = "sir"
coinfection = false

host_popsize = 10
host_network_path = "network.txt"

num_sites = 10000
expected_characters = ["A", "T", "C", "G"]
pathogen_path = "pathogens.fa"

num_instances = 1
```

The `simulation` section sets the type of epidemic model to run, size and shape of the host population network, the number of pathogen genetic sites to model, and the number of independent realizes to run.

To mark the start of the simulation section, the section title is enclosed by square brackets `[]` and all parameters declared after become part of the section until a new section title is encountered. This behavior is not only limited to the Contagion configuration file but is part of the TOML specification.

## epidemic_model

The `epidemic_model` parameter specifies the type of epidemic model to simulate. Contagion uses the concept of a compartmental model to determine the status of each host in the simulation. During the simulation, a host can have only one status (belong to only one compartment) at every time step. The initial (default) state is the susceptible state. This indicates that the host is not currently infected. When a pathogen infects a host, the host moves from susceptible and enters the infected state. The duration of the infection can be determined using a transition probability, a definite assigned length, or depend on the fitness of infecting pathogens within the host. In some models, infection is chronic and persists throughout the simulation time, whereas others allow the host to remove the infection.

Below are the list of epidemic models currently implemented in Contagion.

Keyword | Epidemic model | Description
------- | -------------- | -----------
si | susceptible-infected | Hosts infected with the pathogen remain infected until the end of the simulation
sis | susceptible-infected-susceptible | Hosts recover from the infection and immediately become susceptible again. After infection, all pathogens within the host is removed. However, recovered hosts can be reinfected.
sir | susceptible-infected-removed | Hosts recover from the infection. After infection, all pathogens within the host is removed. Hosts cannot be reinfected after recovery.
sirs | susceptible-infected-removed-susceptible | Similar to `sir` except that hosts can become susceptible again and may be reinfected.
sei | susceptible-exposed-infectious | Hosts infected by the pathogen do not immediately become infectious. This model introduces a lag time between infection and the ability to spread the infection.
seis | susceptible-exposed-infectious-susceptible | Similar to `sis` except a lag time exists between infection and the ability to spread the disease.
seir | susceptible-exposed-infected-removed | Similar to `sir` except a lag time exists between infection and the ability to spread the disease.
seirs | susceptible-exposed-infected-removed-susceptible | Similar to `sirs` except a lag time exists between infection and the ability to spread the disease.
endtrans | susceptible-infected-removed | Based on `sir`, this model conditions transmission of the infection at the last day of infection.

## coinfection

The `coinfection` parameter indicates whether mixed infections are allowed. Originally, pathogens can only infect hosts that are in the susceptible state. However, this behavior makes infected individuals temporarily immune to inoculation while in the infected state. To address this peculiar dynamic, coinfection allows pathogens to both infect susceptible hosts and currently infected individuals.

Value | Description
----- | -----------
true | Allows pathogens to infect susceptible and infected individuals
false | Pathogens can only be spread to susceptible individuals

## host_popsize

The `host_popsize` parameter describes the number of hosts to be modeled in the simulation. The value of this parameter must be an integer and should match the number of hosts in the host network configuration file.

## host_network_path

> Each line in the host network configuration file indicates a one-way connection between two hosts by ID. To specify an undirected connection, declare the host pair twice in the forward and reverse directions.

```text
# Ignores lines that begin with a hash
1   2
2   1
1   3
3   1
1   4
4   1
```

> Individual transmission probabilities can be specified for each connection by adding a third value:

```text
# Ignores lines that begin with a hash
1   2   0.5
2   1   0.7
1   3   0.1
3   1   0.9
1   4   0.8
4   1   0.8
```

The host network configuration file is a text file that lists the number of adjacently-connected hosts in the host population. This adjacency list is used to create an adjacency matrix and define the local neighborhood of each host in the simulation.

The `host_network_path` indicates the location of the host network configuration file. This path can be either an absolute path or the relative path. However, it is recommended to use the absolute path for clarity.

<aside class="notice">
Use absolute paths to indicate the location of the host network configuration file.
</aside>

Currently, Contagion uses a static host network and does not use dynamically changing networks.

## num_sites

The `num_sites` parameter indicates the number of genetic sites to model during the simulation. The value must be an integer greater than 0.

In Contagion, a site is the smallest evolvable unit of information. Each site holds a categorical state that can be mutated based on a given transition matrix of probabilities. The sequence of sites and their states make up the simulated genome of the pathogen.

The number of sites depends on the simulation objective. If the simulation intends to model the molecular evolution at the nucleotide level, then each site should represent each nucleotide in the sequence. On the other hand, if the simulation is concerned about codon or protein evolution, then each site should be modeled after a codon or an amino acid respectively. Another way to model evolution is to consider a site as a locus that holds an allele. Under this view, one site may be sufficient to model the dynamics of the system.

## expected_characters

**deprecated**

The `expected_characters` parameter indicates the list of characters (letters, digits, symbols) the program should expect from the sequences located in the pathogen sequence file. When the program reads the sequence file, it converts the characters into representative integers to make the simulations run more efficiently.

## pathogen_path

> The pathogen sequence file is based on the FASTA file format:

```text
# Ignores lines that begin with a hash
% U:0 P:1
>pathogen1 h:0
UUPUUPUPUUPPUUPUUPUPPPUU
>pathogen2 h:0
PUPUUPUPUPUPPUUUUPPPPPPP
```

> Contagion also allows for non-unique sequence IDs which is considered invalid FASTA formatting.

```text
# Ignores lines that begin with a hash
% U:0 P:1
>h:0
UUPUUPUPUUPPUUPUUPUPPPUU
>h:0
PUPUUPUPUPUPPUUUUPPPPPPP
```

The `pathogen_path` parameter indicate the location of the pathogen sequence file. This path can be either an absolute path or the relative path. However, it is recommended to use the absolute path for clarity.

<aside class="notice">
Use absolute paths to indicate the location of the pathogen sequence file.
</aside>

## num_instances

The `num_instances` parameter specifies the number of independent realizations of the simulation to perform. This is similar to the concept of replication. If `num_instances` is greater than 1, then the simulation will be repeated performed using the same set of input parameters. Since the processes in the simulation are stochastic, then each realized simulation may not necessarily reproduce the results from previous trials.

# Logging parameters

## log_freq

The `log_freq` parameter indicates how often data generated by the simulation will be saved to disk. Setting `log_freq` to 1 will save all the data generated for every step in the simulation. On the other hand, setting it to a value greater than 1 means that data will be saved at periodic intervals only.

## log_path 

The `log_path` parameter indicates where to save the data generated during the simulation. This path can be either an absolute path or the relative path. However, it is recommended to use the absolute path for clarity.

<aside class="notice">
Use absolute paths to indicate the location of the pathogen sequence file.
</aside>

# Host models

# Intrahost model

The intrahost model specifies the parameters that affect the behavior of pathogen within the host. The intrahost model

## model_name
## host_ids
## mutation_rate
## transition_matrix
## recombination_rate
## replication_model
## max_pop_size
## infected_duration

# Fitness model
## model_name
## host_ids
## fitness_model
## fitness_model_path

# Transmission model
## model_name
## host_ids
## mode
## transmission_prob
## transmission_size

# Stop conditions
## condition
## sequence
## position

# Running the simulation

# Analyzing data

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

