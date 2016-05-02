---
layout: docs
project: depcon
menu: docs
title: Getting Started
first-section: Overview
---

{:.no-margin-top}
## Overview

Depcon makes managing clusters that run docker containers a breeze.  It offers the ability to define environments such as test, pre-prod, production against Marathon (initial release), Kubernetes and Amazon ECS.  

**Some key features of Depcon are:**


- Blue/Green deployments
- Variable substitution in deployment descriptors
- Output results in Column, YAML and JSON formats for easy integration with automation
- Accepts deployment descriptors in either JSON or YAML format
- **Full Mesos/Marathon support**
  - Application, Group & Task management
  - Partial application updates: CPU, Memory & Scaling
  - Quick application rollback
  - Leader election, Server info and elapsed response
- **Docker compose support**
  - Supports all major operating systems
  - Works with Docker Machine aka Boot2Docker
  - Adds param support -- `${PARAMS}` can be placed in compose files
- Future releases will offer a proposed Open Deployment Descriptor format which will allow Depcon to deploy a common descriptor against Marathon, Kubernetes, ECS and Docker Swarm.
- Ability to wait until a new or updated container deployment is healthy

## Installation

### Binary Installation

Binaries are available through GitHub releases.  You can download the appropriate binary, package and version from the [Releases](https://github.com/ContainX/depcon/releases) page

### Building Source

**Pre-Requisites**

- GOLANG 1.5+
- Vendor support enabled.  You **MUST** add env variable `GO15VENDOREXPERIMENT=1` to enable

Add Depcon and its package dependencies to your go `src` directory

{:.console .command}
```
$ go get -v github.com/ContainX/depcon
```

Once the `get` has completed, you should find your new `depcon` (or `depcon.exe`) executable sitting inside the `$GOPATH/bin/`

To update Depcon's dependencies, use `go get` with the `-u` option.

{:.console .command}
```
go get -u -v github.com/ContainX/depcon
```

### Depcon in Docker

With each release we publish a very small docker image containing depcon.

**Quick Example**

First, run Depcon in the background capture cid and add an alias

{:.console .command}
```
$ cid=$(docker run -itd -v $PWD:/data containx/depcon)
$ alias depcon="docker exec -it ${cid} depcon"
```

Next, use Depcon just like any the native install

{:.console .command}
```
$ depcon app list
```

## Configuration

### Environments

Depcon uses a configuration file it stores in the users home directory.  If your familiar with Docker then this concept is quite similar when logging into Docker hub.  Environments are keyed by a user defined name (ex: DEV, Test, Stage, Prod) and capture the remote details about a cluster.

{:.callout .callout-info}
The first time Depcon is executed it will prompt you there is no configuration.  It will then proceed to create your initial environment.  This is the same as **Adding an Environment** below.  

**Add a new Environment**

To add a new Marathon environment type the following command: ```depcon config env add```.  This command prompts Depcon to ask you a series of questions about your environment.

For example:

```
$ depcon config env add
Environment Name (eg. test, stage, prod) : dev
Marathon URL (eg. http://hostname:8080)  : http://1.1.1.1:8080
Authentication Required [y/N]? Y
Username: john
Password: password

New environment "dev" added successfully!
```

**Listing Environments**

To list configured environments type ```depcon config env list```.

Example output:

```
NAME    TYPE      URI                               AUTH  DEFAULT
dev     marathon  http://1.1.1.1:8080               false false
test    marathon  http://qa.mycompany.io/marathon   true  false
stage   marathon  http://stage.mycompany.io:8080    true  true
```

**Renaming an Environment**

To change a name of an existing environment the command is:

```depcon config env rename [oldname] [newname]```

**Deleting an Environment**

To delete an Environment: ```depcon config env delete [name]```

**Setting the default Environment**

If your using an environment more than others its often best to set a default.  By setting a default it eliminates the need to specify ```depcon -e env some-command```.

The syntax is: ```depcon config env default [name]```

### Output Format

Depcon is flexible.  It allows output to be in the form of **Column**, **JSON** or **YAML**.  Regardless of the default setting you can execute and have the output returned in the desired format.  The command below sets the default if no output flag is specified during execution.

```depcon config output [column | json | yaml]```

### Jailing Commands

In the future Depcon will support many cluster types.  Currently we only support Apache Mesos/Marathon.  Jailing allows us to cut out a portion of the command tree during execution.  Since we know for a particular environment name it's technology type we can ```chroot``` the command sequence.

For example, if the environment is Marathon, **without command chroot**

```depcon mar app list```

**With chroot / jailing**

```depcon app list```

{% include common-links.html %}
