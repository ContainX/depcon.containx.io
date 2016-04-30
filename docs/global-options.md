---
layout: docs
project: depcon
menu: docs
title: Global Options
first-section: Output Options
---

{:.no-margin-top}
## Output Options

Depcon makes it easy to integrate with third party systems. Any command or query in Depcon has the options to list results in tabular, ```JSON``` or ```YAML``` formats.

For example: ```depcon app list -o json``` would return a list of running applications in JSON form. You can also use ```-o yaml``` for YAML or no option which by default results in table/tabular form.

### Switching Environments

Depcon allows the ability to [configure *N* environments](/docs/getting-started/#environments). To switch between environments you can use the ```-e``` global flag.  

Examples:

{:.console}
```nohighlight
# Dev Integration
$ depcon -e dev app list

# QA
$ depcon -e qa app list

# PROD
$depcon -e prod app list
```

{:.callout .callout-info}
The ```-e``` flag is unique among other flags.  It must be used as the first argument as shown in the examples above

### Help

The ```-h``` or ```--help``` flag can be used to print usage at any level within Depcon.  It will always provide usage based on the current subtree in command sequences
