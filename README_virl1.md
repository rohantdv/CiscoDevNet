# virlutils

[![Build Status](https://travis-ci.org/CiscoDevNet/virlutils.svg?branch=master)](https://travis-ci.org/CiscoDevNet/virlutils)
[![Coverage Status](https://coveralls.io/repos/github/CiscoDevNet/virlutils/badge.svg?branch=master)](https://coveralls.io/github/CiscoDevNet/virlutils?branch=master)
[![PyPI version](https://badge.fury.io/py/virlutils.svg)](https://badge.fury.io/py/virlutils)

A collection of utilities for interacting with [Cisco VIRL](https://learningnetworkstore.cisco.com/virlfaq/aboutVirl) 1.x or Cisco Modeling Labs (CML) 2+.

This document describes the new version of virlutils (aka cmlutils) that works with Cisco Modeling Labs 2+.  Documentation for working with VIRL/CML 1.x is available [here](README_virl1.md).

### virl up

`virl` (or `cml`) is a devops style cli which supports the most common VIRL/CML operations.  Adding new ones is easy...

```sh
Usage: cml [OPTIONS] COMMAND [ARGS]...

Options:
  --debug / --no-debug  Print any debugging output.
  --help                Show this message and exit.

Commands:
  clear        clear the current lab ID
  cockpit      opens the Cockpit UI
  console      console for node
  definitions  manage image and node definitions
  down         stop a lab
  extract      extract configurations from all nodes in a lab
  generate     generate inv file for various tools
  id           get the current lab title and ID
  license      work with product licensing
  ls           lists running labs and optionally those in the cache
  nodes        get node list for the current lab
  pull         pull topology.yaml from repo
  rm           remove a lab
  save         save lab to a local yaml file
  search       list topologies available via github
  ssh          ssh to a node
  start        start a node
  stop         stop a node
  telnet       telnet to a node
  ui           opens the Workbench for the current lab
  up           start a lab
  use          use lab launched elsewhere
  version      version information
  wipe         wipe a lab or nodes within a lab
```



<!-- TOC depthFrom:2 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage / Workflows](#usage--workflows)
- [Development](#local-development)

<!-- /TOC -->

## Prerequisites

- Python 3.6.8+ (tested with Python 3.7)

## Installation

1. Clone this repo

```
git clone https://github.com/CiscoDevNet/virlutils
cd virlutils
```

2. Either (2a) use pip, or (2b) use setup.py

   2a. Use pip

   ```
   pip install virlutils
   ```

   Or

   ```sh
   pip install cmlutils
   ```

   2b. Use setup.py

   ```sh
   python3 -m venv venv
   source venv/bin/activate
   python setup.py install
   ```

## Configuration

There really isn't much to configure, just set your VIRL/CML credentials.  
There are a few different ways to accomplish this, pick whichever one works best for you. The options listed below are in the `preferred` order.


#### .virlrc in working directory

Add a .virlrc to the working directory, this will always be checked first and
is useful when you want to override one or more parameters for a particular project
directory.

The contents would look something like this.

```
VIRL_HOST=specialvirlserver.foo.com
```


#### environment variables
You can also add them as environment variables. This is useful if you want to override
the global VIRL settings.

```sh
export VIRL_HOST=192.0.2.100
export VIRL_USERNAME=guest
export VIRL_PASSWORD=guest
```

#### .virlrc in your home directory

Configure VIRL credentials globally by putting them in ~/.virlrc the formatting

```
VIRL_USERNAME=netadmins
VIRL_PASSWORD=cancodetoo!
```

### Other configuration options

In addition to basic credentials, the following configuration options are supported
using any of the methods mentioned previously

* `VIRL_TELNET_COMMAND` - allows the user to customize the telnet command that is called.
  This command will be passed the host/ip information from the running simulation

   Example:
   ```sh
   export VIRL_TELNET_COMMAND="mytelnet {host}"
   ```

* `VIRL_CONSOLE_COMMAND` - (*VIRL or CML 1.x only*) allows the user to customize the telnet command that is called.

This command will be passed the host/ip and port information information from the running simulation

  Example:
  ```sh
  export VIRL_TELNET_COMMAND="mytelnet {host} {port}"
  ```

* `VIRL_SSH_USERNAME` - the username by which SSH connections to the nodes running in
  the simulation will be initiated with

  Example:
  ```sh
  export VIRL_SSH_USERNAME=netadmin
  ```


* `VIRL_SSH_COMMAND` - allows the user to customize the ssh command that is called.
  This command will be passed the host/ip as well as the username from the running simulation

  Example:
  ```sh
  export VIRL_SSH_COMMAND="myssh {username}@{host}"
  ```

- `CML_VERIFY_CERT` - (*CML 2+ only*) The path to a PEM-encoded certificate file to use to verify the CML controller VM's SSL certificate.  If you do not wish to verify the certificate, set this to "False"

  Example:

  ```sh
  export CML_VERIFY_CERT=/etc/certs/ca_bundle.pem
  ```

- `CML_CONSOLE_COMMAND` - (*CML 2+ only*) allows the user to customize the SSH command that is called.

  This command will be passed the CML controller VM IP, the console path of the node, and the CML controller username (**note:** you may have to force a TTY allocation in your SSH command)

  Example:

  ```sh
  export CML_SSH_COMMAND="myssh {user}@{host} {console}"
  ```

- `CML2_PLUS` - If set in the config or in the environment then virlutils will assume the server is a CML 2+ server and not try and automatically guess its version.  If omitted, then virutils will attempt to automatically determine the CML/VIRL server version

  Example:

  ```sh
  export CML2_PLUS="yes"
  ```



### Why so many choices??!?!

Understanding the precedence allows you to do some pretty cool things.

Assume the following directory structure...

```
.
├── dev
│   ├── .virlrc
│   └── topology.virl
├── prod
│   ├── .virlrc
│   └── topology.virl
└── test
    ├── .virlrc
    └── topology.virl

```

This allows three major benefits.  

1. you can easily use different credentials/servers for various environments
2. you can specify environment specific details into your .virl files if you need to. we find
this most useful in the context of out-of-band management networks/gateways and such.
3. you have a badass workflow..

```sh
(netdevops-demo) ➜  dev git:(test) ✗ virl ls  
Running Simulations
╒══════════════╤══════════╤════════════╤═══════════╕
│ Simulation   │ Status   │ Launched   │ Expires   │
╞══════════════╪══════════╪════════════╪═══════════╡
╘══════════════╧══════════╧════════════╧═══════════╛
(netdevops-demo) ➜  dev git:(test) ✗ cd ../test
(netdevops-demo) ➜  test git:(test) ✗ virl ls
Running Simulations
╒═════════════════════╤══════════╤════════════════════════════╤═══════════╕
│ Simulation          │ Status   │ Launched                   │ Expires   │
╞═════════════════════╪══════════╪════════════════════════════╪═══════════╡
│ test_default_hfMQHh │ ACTIVE   │ 2018-03-18T06:23:05.607199 │           │
╘═════════════════════╧══════════╧════════════════════════════╧═══════════╛
(netdevops-demo) ➜  test git:(test) ✗ cd ../prod
(netdevops-demo) ➜  prod git:(test) ✗ virl ls
Running Simulations
╒═════════════════════╤══════════╤════════════════════════════╤═══════════╕
│ Simulation          │ Status   │ Launched                   │ Expires   │
╞═════════════════════╪══════════╪════════════════════════════╪═══════════╡
│ prod_default_jbdKOW │ ACTIVE   │ 2018-03-18T06:18:04.635601 │           │
╘═════════════════════╧══════════╧════════════════════════════╧═══════════╛
```



## Usage / Workflows

### Find and import VIRL files

A collection of VIRL/CML 1.x topologies is being maintained at https://github.com/virlfiles

These repos can be searched from the command line.

```
$ virl search ios
Displaying 1 Results For ios
╒════════════════════════╤═════════╤═══════════════╕
│ Name                   │   Stars │ Description   │
╞════════════════════════╪═════════╪═══════════════╡
│ virlfiles/2-ios-router │       0 │               │
╘════════════════════════╧═════════╧═══════════════╛
```

Once you find an intersting topology, you can either `pull` the topology into your
current environment or launch it directly

pull topology to local directory (as topology.virl)
```
virl pull virlfiles/2-ios-router
```

launch the topology directly using `virl up`

```
virl up virlfiles/2-ios-router
```

***NOTE:*** for CML 2+, you can import VIRL/CML 1.x files, but some nodes may not be present in CML 2+.

### Basic Workflow

in the absence of better documentation, here's a sample workflow


```
(venv) KECORBIN-M-90Y9:virl_cli kecorbin$ virl ls

    Here is a list of all the running nodes

╒═════════════════╤══════════╤════════════════════════════╤═══════════╕
│ Simulation      │ Status   │ Launched                   │ Expires   │
╞═════════════════╪══════════╪════════════════════════════╪═══════════╡
│ topology-CoC73j │ ACTIVE   │ 2017-12-02T14:44:29.209647 │           │
╘═════════════════╧══════════╧════════════════════════════╧═══════════╛
(venv) KECORBIN-M-90Y9:virl_cli kecorbin$ virl up
Launching Simulation from topology.virl
virl_cli-GnMIWY


(venv) KECORBIN-M-90Y9:virl_cli kecorbin$ virl ls

    Here is a list of all the running nodes

╒═════════════════╤══════════╤════════════════════════════╤═══════════╕
│ Simulation      │ Status   │ Launched                   │ Expires   │
╞═════════════════╪══════════╪════════════════════════════╪═══════════╡
│ topology-CoC73j │ ACTIVE   │ 2017-12-02T14:44:29.209647 │           │
├─────────────────┼──────────┼────────────────────────────┼───────────┤
│ virl_cli-GnMIWY │ ACTIVE   │ 2017-12-08T07:35:46.444588 │           │
╘═════════════════╧══════════╧════════════════════════════╧═══════════╛


(venv) KECORBIN-M-90Y9:virl_cli kecorbin$ virl nodes virl_cli-GnMIWY

    Here is a list of all the running nodes

╒═══════════╤══════════╤══════════╤═════════════╤═══════════════════════╕
│ Node      │ Type     │ State    │ Reachable   │ management-protocol   │
╞═══════════╪══════════╪══════════╪═════════════╪═══════════════════════╡
│ iosv-2    │ IOSv     │ BUILDING │ False       │ telnet                │
├───────────┼──────────┼──────────┼─────────────┼───────────────────────┤
│ ~mgmt-lxc │ mgmt-lxc │ ACTIVE   │ True        │ ssh                   │
├───────────┼──────────┼──────────┼─────────────┼───────────────────────┤
│ iosv-1    │ IOSv     │ ACTIVE   │ False       │ telnet                │
╘═══════════╧══════════╧══════════╧═════════════╧═══════════════════════╛


(venv) KECORBIN-M-90Y9:virl_cli kecorbin$ virl console virl_cli-GnMIWY iosv-1
iosv-1
Attempting to connect to console of iosv-1
Trying 10.94.140.41...
Connected to mm-c1-6620.cisco.com.
Escape character is '^]'.

[OK] (elapsed time was 9 seconds)

Building configuration...

telnet> quit
Connection closed.


(venv) KECORBIN-M-90Y9:virl_cli kecorbin$ virl down virl_cli-GnMIWY
Shutting Down Simulation virl_cli-GnMIWY.....SUCCESS
(venv) KECORBIN-M-90Y9:virl_cli kecorbin$ virl ls

    Here is a list of all the running nodes

╒═════════════════╤══════════╤════════════════════════════╤═══════════╕
│ Simulation      │ Status   │ Launched                   │ Expires   │
╞═════════════════╪══════════╪════════════════════════════╪═══════════╡
│ topology-CoC73j │ ACTIVE   │ 2017-12-02T14:44:29.209647 │           │
├─────────────────┼──────────┼────────────────────────────┼───────────┤
│ virl_cli-GnMIWY │ STOP     │ 2017-12-08T07:35:46.444588 │           │
╘═════════════════╧══════════╧════════════════════════════╧═══════════╛

(venv) KECORBIN-M-90Y9:virl_cli kecorbin$ virl ls

    Here is a list of all the running nodes

╒═════════════════╤══════════╤════════════════════════════╤═══════════╕
│ Simulation      │ Status   │ Launched                   │ Expires   │
╞═════════════════╪══════════╪════════════════════════════╪═══════════╡
│ topology-CoC73j │ ACTIVE   │ 2017-12-02T14:44:29.209647 │           │
╘═════════════════╧══════════╧════════════════════════════╧═══════════╛

```

### Localization

virlutils provides a handy way of maintaining portability across multiple VIRL
backend servers.  Any configuration that is stored in your `topology.virl` file
can make use of some special tags which will be substituted at launch (`virl up`) for parameters
unique to the virl host.  

Currently the following tags are supported:

* {{ gateway }} - will be replaced with the default gateway of the `flat` network
* {{ flat1_gateway }} - will be replaced with the gateway IP address of the  `flat1` network
* {{ dns_server }} - replaced with the dns_server configured on the VIRL host

**NOTE:** these tags must be copied exactly (including surrounding braces+spaces)

### Inventory Generation

virlutils will generate inventories for various management systems

#### pyATS Testbed Generation

quickly turn your simulations into a testbed file that can be used for pyATS/Genie

```
virl generate pyats
```

#### Ansible Inventory Generation

quickly turn your simulations into an inventory file that can be used to run your playbooks
against.  Both INI and YAML(default) formats are supported by the tool.

```
Usage: virl generate ansible [OPTIONS] [ENV]

  generate ansible inventory

Options:
  -o, --output TEXT   output File name
  --style [ini|yaml]  output format (default is yaml)
  --help              Show this message and exit.
```

The ansible group membership can be controlled by adding additional extensions to your
VIRL files.  


```
<node name="router1" type="SIMPLE" subtype="CSR1000v" location="361,129" ipv4="172.16.252.6" ipv6="2001:db8:b:0:1::2">
  <extensions>
    <entry key="ansible_group" type="String">mygroup</entry>
  </extensions>
</node>
```

would result in the following inventory entry

```
all:
  children:
    mygroup:
      hosts:
        router1:
          ansible_host: 172.16.252.6

```

**NOTE:** if the ansible_group key is not specified for a node, that node will not be included during
inventory generation.  

#### Cisco Network Services Orchestrator

You can add/update Network Services Orchestrator with your VIRL simulation.


Usage

```
virl generate nso [OPTIONS] [ENV]

  generate nso inventory

Options:
  -o, --output TEXT           just dump the payload to file without sending
  --syncfrom / --no-syncfrom  Perform sync-from after updating devices
  --syncto / --no-syncto      Perform sync-to afgter updating devices
  --help                      Show this message and exit.
```

output

```
Updating NSO....
Enter NSO IP/Hostname: localhost
Enter NSO username: admin
Enter NSO password:
Successfully added VIRL devices to NSO

```

**NOTE**:  NSO environment is also attempted to be determined using the following environment
variables

* NSO_HOST
* NSO_USERNAME
* NSO_PASSWORD

NSO Configuration Example

```
export NSO_HOST=localhost
export NSO_USERNAME=admin
export NSO_PASSWORD=admin
```



#### Tab Completions


```
➜  test git:(test) virl l<tab>
logs  ls  

```

You can activate VIRL autocompletions by executing the following command

```
eval "$(_VIRL_COMPLETE=source virl)"
```

zsh users may need to run the following prior

```
autoload bashcompinit
bashcompinit
```

## Local Development

If you have an idea for a feature you would like to see, we gladly accept pull requests.  To get started developing, simply run the following..

```
git clone https://github.com/CiscoDevNet/virlutils
cd virlutils
python setup.py develop
```

### Linting

We use flake 8 to lint our code. Please keep the repository clean by running:

```
flake8
```

### Testing

We have some testing implemented, but would love to have better coverage. If you
add a feature, or just feel like writing tests please update the appropriate files
in the `tests` folder.

To run the tests in the `tests` folder, you can simply run `make test` from
the project root.
