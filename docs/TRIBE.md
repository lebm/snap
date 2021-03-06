<!--
http://www.apache.org/licenses/LICENSE-2.0.txt


Copyright 2015 Intel Corporation

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

<img src="https://cloud.githubusercontent.com/assets/6523391/19813910/24b75b66-9d3c-11e6-98ed-e6897faafd1c.png" width="20%" align="right">
</br>
</br>
</br>
</br>

# Tribe

Tribe is the name of the clustering feature in Snap.  When it is enabled, snapteld instances can join to one another through an `agreement`. When an action is taken by one snapteld instance that is a member of an agreement, that action will be carried out by all other members of the agreement. When a new snapteld joins an existing agreement it will retrieve plugins and tasks from the members of the agreement.

## Usage
This walkthrough assumes you have downloaded a Snap release as described in [Getting Started](../README.md#getting-started).

### Starting the first snapteld in tribe mode

Start the first node:
```
$ snapteld --tribe -t 0
```

Only `--tribe` and some trust level (`-t 0`) is required to start tribe. This will result in defaults for all other parameters:
* Default `tribe-node-name` will be the same as your hostname
* Default `tribe-seed` is port 6000
* Default `tribe-addr` and `tribe-port` are the same as `snapteld` and `tribe-seed` (ex. 127.0.0.1:6000)

See `snapteld -h` for all the possible flags.

### Creating an initial agreement

Members of a tribe only share configuration once they join an agreement. To create your first agreement:
```
$ snaptel agreement create all-nodes
Name 	    Number of Members 	 plugins 	 tasks
all-nodes 	0 			         0 		     0
```

Join our running snapteld into this agreement:
```
$ snaptel agreement join all-nodes `hostname`
Name 	    Number of Members 	 plugins 	 tasks
all-nodes 	1 			         0   		 0
```

### Joining other snapteld into an existing tribe
Since tribe is implemented on top of a gossip based protocol there is no "master." All other nodes who join a tribe by communicating with any existing member.

Start another instance of snapteld to join to our existing tribe. The local IP address is 192.168.136.176 in our example. Note that we need a few more parameters to avoid conflicting ports on a single system:
```
$ snapteld --tribe -t 0 --tribe-port 6001 --api-port 8182 --tribe-node-name secondnodename --tribe-seed 192.168.136.176:6000 --control-listen-port 8083
```

Both snapteld instances will see each other in their member list:
```
$ snaptel member list
Name
secondnodename
firstnode
```

This member needs to join the agreement:
```
$ snaptel agreement join all-nodes secondnodename
Name 		 Number of Members 	 plugins 	 tasks
all-nodes 	 2       			 0 		     0
```

From this point forward, any plugins or tasks you load will load into both members of this agreement.

*Note: Once the cluster is started subsequent new nodes can choose to establish membership through **any** node as there is no "master".*