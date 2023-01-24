---
title: Understand ROS Nodes
tags: [linux, ros]
---

## Concepts

### Nodes

ROS nodes are executables in a ROS package able to publish messages or subscribe to a topic through ROS client communication library.

### Client Libraries 

ROS client libraries allows nodes written in different languages to communicate with each other. There are two ROS libraries which are `rospy` and `roscpp`.

### ROSCore

`roscore` serves as a core service for all the ROS operations.

## Basic Commands 

### `rosnode`

You can list all running nodes by execution of `rosnode list`.

And to view details about certain node: `rosnode info /node_name`

For example, `rosnode info /rosout` may emit message:
```
------------------------------------------------------------------------
Node [/rosout]
Publications:
 * /rosout_agg [rosgraph_msgs/Log]

Subscriptions:
 * /rosout [unknown type]

Services:
 * /rosout/get_loggers
 * /rosout/set_logger_level

contacting node http://machine_name:54614/ ...
Pid: 5092
```

### `rosrun`

`rosrun` enables you to run a node with a package through the name of that package. For example: `rosrun turtlesim turtlesim_node` creates a window with a turtle in the middle.

