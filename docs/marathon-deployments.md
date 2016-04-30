---
layout: docs
project: depcon
menu: docs
title: Marathon Deployments
first-section: Listing queued deployments
---

{:.no-margin-top}
## Overview

Deployments a generally quick but at times you need to detect if one is stalled.  A deployment is the process in Marathon where a request to provision an application or group has been received and its in transition.  This can be due to waiting for the appropriate offer, health checks to pass or startup time.

### Querying

To view the current list of deployments and queued deployments run:

{:.console}
```nohighlight
$ depcon deploy list
```

If you have anything currently active or queued the result will be similar to:

{:.console}
```nohighlight
DEPLOYMENT_ID                         VERSION                   PROGRESS  APPS
22035f20-3b29-42d2-a40d-0b7ac46f45ee  2016-04-30T05:12:49.789   2/2       myapp
12235f20-1529-66d2-bc0d-5bdac46f45bf  2016-04-30T05:13:03.100   0/2       myapp2
```

### Deleting/Removing a deployment from the queue

To remove a currently active or pending deployment execute:

{:.console}
```nohighlight
$ depcon deploy delete <deploymentID>
```
