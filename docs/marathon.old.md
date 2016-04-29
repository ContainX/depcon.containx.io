---
layout: docs
project: depcon
menu: docs
title: Marathon
first-section: Applications
---

{:.no-margin-top}
# Managing Mesos with Marathon

Depcon supports [Mesosphere Marathon](https://mesosphere.github.io/marathon/) as it's primary cluster type.  The following user manual covers all the Depcon commands and advanced usage related to Marathon.

{:.callout .callout-info}
All the commands below assume you've set a ```default``` environment and have ```jailing/chroot``` enabled in the Depcon configuration.  This is not required, but to shorten examples we are setting the context with this enabled.

## Applications

Applications are an integral concept in Marathon. Each application typically represents a long running service, of which there would be many instances running on multiple hosts.

### Creating

The notion of creating an application using Depcon applies the same for first time application deployments and updating existing.  For example if an application does not exist on the cluster then Depcon simply creates it.  This is also known as deploying.  If you have a new version for an existing application you would like to deploy then the same commands apply below along with the addition of the ```--force``` flag.

#### Basic Application

Let us start with the simple example from the [Mesosphere Marathon site](https://mesosphere.github.io/marathon/docs/application-basics.html). We will create an application that prints ```Hello Marathon``` and then sleeps for 5 seconds, in an endless loop. Let's use the following service descriptor below in this example.

{:.callout .callout-info}
The example below is presented in JSON format.  Depcon supports descriptors in the form of ```JSON``` or ```YAML```.

```
{
    "id": "basic-0",
    "cmd": "while [ true ] ; do echo 'Hello Marathon' ; sleep 5 ; done",
    "cpus": 0.1,
    "mem": 10.0,
    "instances": 1
}
```

To deploy/create this application in Marathon using Depcon we would run the following:

```
depcon app create hello-marathon.json
```

and the output would be similar to [querying an application](#querying).

#### Advanced Application (Docker)

Lets jump into a more advanced deployment by using a Docker application.  We'll also utilize the power of Depcon with parameter substitution with the descriptor template.

For this example we'll create a template below in ```YAML``` format which deploy a Redis cache instance. We will also add some parameters for the version of redis and the service port we want to expose on our load balancer.

```
---
id: /storage/redis
cpus: 0.1
mem: 30.0
instances: 1
container:
  type: DOCKER
  docker:
    image: redis:${VERSION}
    network: BRIDGE
    portMappings:
    - containerPort: 6379
      hostPort: 0
      servicePort: ${LBPORT}
      protocol: tcp
healthChecks:
- portIndex: 0
  protocol: TCP
  gracePeriodSeconds: 50
  intervalSeconds: 30
  timeoutSeconds: 20
  maxConsecutiveFailures: 10
```
To deploy the container above using Depcon we would run it like before but feeding it the values we want for ```VERSION``` and ```LBPORT``` during it's deployment.  We will also tell Depcon to block until all health checks have passed by using the ```--wait``` flag.  This is a key flag to use when leveraging Depcon in your continuous delivery pipelines.

```
depcon app create redis.yml -p VERSION=latest -p LBPORT=6379 --wait
```

The end result would have the template translated during deploy to:

```
...
docker:
  image: redis:latest
  network: BRIDGE
  portMappings:
  - containerPort: 6379
    hostPort: 0
    servicePort: 6379
    protocol: tcp
...
```
For more information about substitution within templates, see the [Parameter Substitution](#parameter-substitution) section in these docs.

### Updating Memory and CPU for running applications

There may be times where you want to increase ```CPU``` or ```Memory``` for running applications without deploying the complete application.

```
depcon app update cpu 1.2
depcon app update mem 300
```

### Querying

**List all running Applications**

Using the syntax ```depcon app list``` command will output listing of all running applications.  

```
ID            INSTANCES	CPU   MEM     PORTS  CONTAINER                  VERSION
/services/a   2         0.50  300.00  4000   containx/service-a:latest  2016-03-29T06:09:12.527Z
/db/a/mysql   1         0.50  600.00  10003  mysql/mysql-server:5.6     2016-04-23T18:12:15.586Z
/db/b/mysql   1         0.50  600.00  10000  mysql/mysql-server:5.6     2016-03-27T21:14:10.669Z
...
```

**Fetching details for a specific Application**

Syntax: ```depcon app get /db/a/mysql```

```
ID:         /db/a/mysql
CPUs:       0.50
Memory:     600.00
Ports:      10001
Instances:  1
Version:    2016-04-09T17:01:15.823Z
Tasks:      Staged                   : 0
            Running                  : 1
            Healthy                  : 1
            UnHealthy                : 0

Container:  Type                     : Docker
            Image                    : mysql/mysql-server:5.6
            Network                  : BRIDGE

Environment:    MYSQL_DATABASE       : mydb
...                
```

### Destroy/Teardown an Application

In order to destroy/teardown a running application and all of it's instances you must know it's ID. This can be obtained from the descriptor template or by executing ```depcon app list``` to see all the running applications.

The command would then be: ```depcon app destroy /my/app/identifier```

### Rollback

Marathon natively supports the ability for quick rollbacks.  A few examples of issuing a rollback with Depcon are:

**Rollback to the last version**

```depcon app rollback /my/app/id```

**Rollback to a specific version**

Due to the fact the Marathon supports more than just Docker it has it's own notion of a version.  In order to find a list of assigned versions for a particular application we can issue he following command: ```depcon app versions /my/app/id```

Output similar to below will be outputted:

```
VERSIONS
2016-03-27T21:14:10.669Z
2016-02-23T09:10:11.001Z
2016-02-21T01:44:02.232Z
```

To rollback to the first version from the list above we would issue:

```depcon app rollback /my/app/id 2016-02-21T01:44:02.232Z```

### Scaling

To scale a running application up or down simply using Depcon:

```
depcon app scale [desiredCount]
```

### Converting between Formats

If you currently have descriptors in ```JSON``` format and would like to take advantage of Depcon's ```YAML``` support, a conversion option is available.  The conversion options allow you to change between the two formats at anytime and preserves Depcon parameters in the process.

To convert an existing ```app-descriptor.json``` to ```app.descriptor.yaml execute:

```
depcon app convert app-descriptor.json app-descriptor.yaml
```


# Application Groups

Applications can be nested into a n-ary tree, with groups as branches and applications as leaves. Application Groups are used to partition multiple applications into manageable sets.

![Marathon Application Groups](/assets/img/docs/groups.png)

***Image provided by Mesosphere Marathon*

### Creating an Application Group

### Querying Application Groups

TODO

# Blue/Green Deployments

TODO

# Parameter Substitution

{% include common-links.html %}
