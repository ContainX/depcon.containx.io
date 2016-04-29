---
layout: docs
project: depcon
menu: docs
title: Parameter Substitution
first-section: Overview
---

{:.no-margin-top}
## Overview

Depcon exposes a powerful feature which reduces the amount of application descriptors, compose files, etc.  It treats these files like templates allowing you to resolve user defined values at deployment time.  We refer to this as parameter substitution.

### Example Usage

Let's create a simple application descriptor which deploys a Redis Cache instance.  Depending on the environment (CI, QA, PROD) we want the ability to change certain constraints during our continuous delivery phase.

**redis.yml**

```
---
id: /storage/redis-{$NAME}
cpus: 0.5
mem: ${MEMORY}
instances: 1
container:
  type: DOCKER
  docker:
    image: redis
    network: BRIDGE
    forcePullImage: false
    portMappings:
    - containerPort: 6379
      servicePort: ${SPORT}
      protocol: tcp
healthChecks:
- portIndex: 0
  protocol: TCP
  gracePeriodSeconds: 50
  intervalSeconds: 30
  timeoutSeconds: 20
  maxConsecutiveFailures: 10
```

In the descriptor above we have exposed 3 parameters: ```NAME```, ```MEMORY``` and ```SPORT```.  Let's see what happens when we try to deploy this descriptor as is:

{:.console}
```nohighlight
$ depcon app create redis.yml

WARNING [depcon]: Cannot find a value for varible ${NAME} which was defined in redis.yml
WARNING [depcon]: Cannot find a value for varible ${MEMORY} which was defined in redis.yml
WARNING [depcon]: Cannot find a value for varible ${SPORT} which was defined in redis.yml
ERROR   [depcon]: One or more ${PARAMS} that were defined in the app configuration could not be resolved.
```

{:.callout .callout-info}
The behavior above is expected and Depcon will abort the deployment.  In some scenarios depending on your templates it may be ok to not resolve parameters.  If this is the case you can execute the above again with an additional flag of either ```-i``` or ```--ignore```.

### Providing Parameters needed for Resolution

There are two ways we can provide Depcon the resolution of our 3 parameters we defined above.  

Using parameter flags during deployment:

{:.console}
```
$ depcon app create redis.yml -p NAME=dev -p MEMORY=256.0 -p SPORT=16379
```

Environment variables:

{:.console}
```
$ export NAME=dev
$ export MEMORY=256.0
$ export SPORT=16379

$ depcon app create redis.yml
```

Both methods above will yield the same results.  The final template would look something like this during deployment:

```
---
id: /storage/redis-dev
cpus: 0.5
mem: 256.0
instances: 1
container:
  type: DOCKER
  docker:
    image: redis
    network: BRIDGE
    forcePullImage: false
    portMappings:
    - containerPort: 6379
      servicePort: 16379
      protocol: tcp
healthChecks:
- portIndex: 0
  protocol: TCP
  gracePeriodSeconds: 50
  intervalSeconds: 30
  timeoutSeconds: 20
  maxConsecutiveFailures: 10
```
