---
layout: docs
project: depcon
menu: docs
title: Application Groups
first-section: Creating an Application Group
---

{:.no-margin-top}
## Overview

Applications can be nested into a n-ary tree, with groups as branches and applications as leaves. Application Groups are used to partition multiple applications into manageable sets.

![Marathon Application Groups](/assets/img/docs/groups.png)

***Image provided by Mesosphere Marathon*

### Creating an Application Group

In this example we'll use the traditional wordpress blog example.  This is a great example for using application groups since wordpress depdends on MySQL.  We've built a group below which creates wordpress along with it's depdency of MySQL.  We also have added 2 Depcon parameters called ```Password``` and ```Blog``` which allow us to set the MySQL root password and blog database name via deploy.

{:.descriptor}
```
id: /site
groups:
- id: db
  apps:
  - id: mysql
    container:
      type: DOCKER
      docker:
        image: mysql/mysql-server:5.6
        network: BRIDGE
        forcePullImage: false
        portMappings:
        - containerPort: 3306
          servicePort: 3306
          protocol: tcp
    env:
      MYSQL_ROOT_PASSWORD: ${Password}
      MYSQL_DATABASE: ${Blog}
    healthChecks:
    - portIndex: 0
      protocol: TCP
      gracePeriodSeconds: 300
      intervalSeconds: 30
      timeoutSeconds: 20
      maxConsecutiveFailures: 3
- id: wordpress
  apps:
  - id: blog
    dependencies:
    - /site/db/mysql
    container:
      type: DOCKER
      docker:
        image: wordpress:4.4.2
        network: BRIDGE
        forcePullImage: false
        portMappings:
        - containerPort: 80
          hostPort: 0
          servicePort: 80
          protocol: tcp
    env:
      WORDPRESS_DB_PASSWORD: ${Password}
      WORDPRESS_DB_HOST: ${HOST}:3306
      WORDPRESS_DB_NAME: ${Blog}
    healthChecks:
    - path: /
      portIndex: 0
      protocol: HTTP
      gracePeriodSeconds: 300
      intervalSeconds: 30
      timeoutSeconds: 20
      maxConsecutiveFailures: 3
      ignoreHttp1xx: false
```

Given the descriptor above we can deploy using Depcon similar to a normal application.

{:.console}
```
$ depcon group create wordpress.yml -p Password=mypassword -p Blog=containx --wait
```

### Querying Application Groups

**List all running Application Groups**

Executing ```depcon group list``` will list all application groups

{:.console}
```
ID              VERSION                   GROUPS  APPS
/depcon         2016-04-04T05:38:27.077Z	1       0
/site           2016-04-04T05:38:27.077Z	2       0
/site/db        2016-04-04T05:38:27.077Z	0       1
/site/wordpress 2016-04-04T05:38:27.077Z	0       1
```
**Listing groups starting based on a root identifier path**

If you have a lot of groups sometimes you may want to only list groups starting from a particular id path.  

{:.console}
```
depcon group get /site
```

The output from the command above would be:

{:.console}
```
ID              VERSION                   GROUPS  APPS
/site           2016-04-04T05:38:27.077Z	2       0
/site/db        2016-04-04T05:38:27.077Z	0       1
/site/wordpress 2016-04-04T05:38:27.077Z	0       1
```

### Destroy/Teardown an Application Group

In order to destroy/teardown a group and all of it's application instances you must know it's ID. This can be obtained from the descriptor template or by executing ```depcon group list``` to see all the application groups.

The command would then be: ```depcon group destroy /my/group```

### Converting group Formats

If you currently have descriptors in ```JSON``` format and would like to take advantage of Depcon's ```YAML``` support, a conversion option is available.  The conversion options allow you to change between the two formats at anytime and preserves Depcon parameters in the process.

To convert an existing ```group-descriptor.json``` to ```group.descriptor.yaml``` execute:

{:.console .command}
```
$ depcon app convert group-descriptor.json group-descriptor.yaml
```

{% include common-links.html %}
