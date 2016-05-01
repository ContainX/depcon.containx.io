---
layout: docs
project: depcon
menu: docs
title: Task Management
first-section: Querying
---

Tasks are represented in the form of how Mesos see's the Marathon application. They provide a linked view of a Marathon application to Mesos tasks and allow you to obtain both identifiers.

{:.no-margin-top}
### Querying Tasks

**Listing all tasks**

The  ```depcon task list``` command will output listing of all tasks disregarding status

Example output:

{:.console}
```nohighlight
APP_ID         HOST         VERSION                   STARTED_AT          TASK_ID
/prod/app/a    14.39.14.85  2016-04-30T01:25:10.769Z  2016-04-30 01:25:26 app_a.61a061bd-0e72-11e6-8f1b-0242cceb4d6e
/prod/app/b    14.39.14.93  2016-04-23T18:12:15.586Z  2016-04-23 18:12:16 app_b.e8bdca15-097e-11e6-8f1b-0242cceb4d6e
/qa/app/a      14.39.14.93  2016-03-27T21:14:10.669Z  2016-03-27 21:14:11 qa_app_a.d975ecc0-f460-11e5-ad09-bc764e063d34
/qa/app/b      14.39.14.93  2016-04-23T18:12:15.586Z  2016-04-23 18:12:46 qa_app_b.faa2c876-097e-11e6-8f1b-0242cceb4d6e
/dev/db/mongo  14.39.14.85  2016-04-05T16:54:12.256Z  2016-04-05 16:54:48 dev_mongo.17b2cd06-fb4f-11e5-ad09-bc764e063d34
...
```

**Listing all tasks for an Application**

To view all tasks for all instances of an application use: ```depcon task get /my/app/id```

Example output:

{:.console}
```nohighlight
APP_ID         HOST         VERSION                   STARTED_AT          TASK_ID
/prod/app/a    14.39.14.85  2016-04-30T01:25:10.769Z  2016-04-30 01:25:26 app_a.61a061bd-0e72-11e6-8f1b-0242cceb4d6e
/prod/app/a    14.39.14.93  2016-04-23T18:12:15.586Z  2016-04-23 18:12:16 app_b.e8bdca15-097e-11e6-8f1b-0242cceb4d6e
```

### Killing Tasks

**Kill a specific task by its ID**

To kill a task by its identifier use ```depcon task kill taskId```.  Marathon will then kill the task and replace it with a new one.  If you would like to scale down the run the above command with the ```--scale``` flag.

**Kill all tasks by application ID**

To kill all task by for an application ```depcon task killall /my/app```.  Marathon will then kill all the tasks and replace them with new ones.  If you would like to scale down then run the above command with the ```--scale``` flag.

**Kill all tasks by application ID on a particular slave node**

This is almost the same as the above command but adding the ```--host``` flag.  

Example: ```depcon task killall /my/app --host="slave-hostname.com"```
