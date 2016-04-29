---
layout: docs
project: depcon
menu: docs
title: Blue-Green Deployments
first-section: Deploying with zero downtime
---

{:.no-margin-top}
## Deploying with Zero Downtime

An important technique for reducing the risk of deployments is known as Blue-Green deployments.  If we call the current live production environment "blue", the technique consists of bring up a parallel "green" environment with the new version of software once everything and ready to go live, you simply flip the switch to the "green" environment, leaving the "blue" environment idle.

In the Mesos, Marathon, Docker world we can accomplish this within the cluster using load balancers.  Depcon plays a key part in this strategy by monitoring the "green" and when it becomes healthy it watches for drained connections to "blue".  It then removes a "blue" instance and adds "green" to the live pool.  It keeps doing this back and forth pattern until all of "blue" has been replaced.

![Blue-Green Deployment](/assets/img/docs/blue-green-deployment-sets.png)

### Requirements

Depcon takes advantage of **[Marathon-LB](https://github.com/mesosphere/marathon-lb)** which bundles HAProxy, LUA and a event stream listener to Marathon.  Marathon-LB ships as a Docker container so you can quickly roll it out across all your slave nodes.

Here is an example of manually running Marathon-LB on each slave node.  This can easily be put into an application descriptor and rolled out to all nodes via Depcon/Marathon.

{:.console}
```nohighlight
$ docker run -p PORTS=9090 --net=host -d --restart=always --name lb --privileged=true \
  mesosphere/marathon-lb sse --health-check --group external -m http://marathon-host:8080
```

### Depcon Usage

Deploying via blue-green is very similar to using the ```depcon app create``` syntax, in fact if this is the first time your deploying an application you can still use blue-green.  

Here's a summary of some of the available options and flags for blue-green:

{:.console}
```nohighlight
Usage:
  depcon app bluegreen [file(.json | .yaml)] [flags]

Flags:
      --dry[=false]: Dry run (no deployment or scaling)
  -c, --env-file="": Adds a file with a param(s) that can be used for substitution.
       These take precidence over env vars
  -i, --ignore[=false]: Ignore missing ${PARAMS} that are declared in app config that could not be resolved
                        CAUTION: This can be dangerous if some params define versions or other required information.
      --instances=1: Initial intances of the app to create
      --lb="http://localhost:9090": HAProxy URL and Stats Port
      --lb-timeout=300: HAProxy timeout - default 300 seconds
  -p, --param=[]: Adds a param(s) that can be used for substitution.
                  eg. -p MYVAR=value would replace ${MYVAR} with "value" in the application file.
                  These take precidence over env vars
      --resume[=true]: Resume from a previous deployment
      --stepdel=6: Delay (in seconds) to wait between successive deployment steps.
```

#### Deploying an Application

Let's re-use the redis example we have used in other guides.

**redis.yml**

```
id: /db/redis
cpus: 0.1
mem: 30.0
instances: 3
container:
  type: DOCKER
  docker:
    image: redis:latest
    network: BRIDGE
    portMappings:
    - containerPort: 5000
      hostPort: 0
      servicePort: 5000
      protocol: tcp
labels:
  HAPROXY_DEPLOYMENT_ALT_PORT: "5001"
  HAPROXY_DEPLOYMENT_GROUP: bluegreen
  HAPROXY_GROUP: external
healthChecks:
- portIndex: 0
  protocol: TCP
  gracePeriodSeconds: 50
  intervalSeconds: 30
  timeoutSeconds: 20
  maxConsecutiveFailures: 10
```

If you notice above there are a few extra options which have been added in comparison to our other examples:

```
labels:
  HAPROXY_DEPLOYMENT_ALT_PORT: "6380"
  HAPROXY_DEPLOYMENT_GROUP: bluegreen
```

The above labels tell Marathon-LB which the secondary "green" application port should have when adding canary versions.

And to deploy this application via blue-green:

{:.console}
```nohighlight
$ depcon app bluegreen redis.yml --lb="http://haproxy-hostname:9090"
```

As Depcon is deploying you will see it adding and draining "blue" and "green" in the log output:

{:.console}
```nohighlight
2016-04-29 14:46:43 INFO [depcon.deploy.wait]: Waiting for application deployment to complete for /db/redis-green
2016-04-29 14:46:43 INFO [depcon.deploy.wait]: Waiting for application deployment to complete for /db/redis-green
2016-04-29 14:46:45 INFO [depcon.deploy.wait]: Waiting for application deployment to complete for /db/redis-green
2016-04-29 14:46:47 INFO [depcon.deploy.wait]: Waiting for application deployment to complete for /db/redis-green
2016-04-29 14:46:49 INFO [depcon.deploy.wait]: Application deployment has completed for /db/redis-green, elapsed time 6.25 sec(s)
2016-04-29 14:46:50 INFO [depcon.deploy.wait]: 1 of 1 expected instances are healthy.  Elapsed health check time of 0.05 sec(s)
2016-04-29 14:46:56 INFO [depcon.marathon.bg]: Existing app running 3 instance, new app running 1 instances
2016-04-29 14:46:56 INFO [depcon.marathon.bg]: Found 8 backends across 2 HAProxy instances
2016-04-29 14:46:56 INFO [depcon.marathon.bg]: There are 1 drained backends, about to kill & scale for these tasks:
db_redis-blue.9ab0efdd-0e53-11e6-8f1b-0242cceb4d6e
2016-04-29 14:46:56 INFO [depcon.marathon.bg]: Scaling new app up to 2 instances
2016-04-29 14:46:56 INFO [depcon.marathon.bg]: Scaling old app down to 1 instances
2016-04-29 14:47:03 INFO [depcon.marathon.bg]: Existing app running 2 instance, new app running 2 instances
2016-04-29 14:47:03 INFO [depcon.marathon.bg]: Found 8 backends across 2 HAProxy instances
2016-04-29 14:47:03 INFO [depcon.marathon.bg]: There are 1 drained backends, about to kill & scale for these tasks:
db_redis-blue.9ab13dfe-0e53-11e6-8f1b-0242cceb4d6e
2016-04-29 14:47:03 INFO [depcon.marathon.bg]: Scaling new app up to 3 instances
2016-04-29 14:47:03 INFO [depcon.marathon.bg]: Scaling old app down to 1 instances
2016-04-29 14:47:09 INFO [depcon.marathon.bg]: Existing app running 1 instance, new app running 3 instances
2016-04-29 14:47:09 INFO [depcon.marathon.bg]: Found 8 backends across 2 HAProxy instances
2016-04-29 14:47:09 INFO [depcon.marathon.bg]: There are 1 drained backends, about to kill & scale for these tasks:
db_redis-blue.9ab0c8cc-0e53-11e6-8f1b-0242cceb4d6e
2016-04-29 14:47:09 INFO [depcon.marathon.bg]: About to delete old app /db/redis-blue

ID:        /db/redis-green
CPUs:      0.1
Memory:    30.00
Ports:     5000
Instances: 3
Version:   2016-04-29T21:45:47.145Z
Tasks:     Staged            : 0
           Running           : 3
           Healthy           : 3
           UnHealthy         : 0

Container: Type              : Docker
           Image             : redis:latest
           Network           : BRIDGE
```
