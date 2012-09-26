---
layout: post
title: "Using Cluster Suite to Manage a High Availability Scheduler"
date: 2012-09-26 15:53
comments: true
categories: [condor, grid, grid computing, computing, Red Hat, MRG Grid, cluster_suite, cluster]
---
Condor provides simple and easy to configure HA functionality for the schedd
that relies upon shared storage (usually NFS).  The shared store is used to
store the job queue log and coordinate which node is running the schedd.  This
means that each node that can run a particular schedd not only have condor
configured but the node needs to be configured to access the shared storage.

For most people condor's native HA management of the schedd is probably
enough.  However, using Cluster Suite to manage the schedd provides some
additional control and protects against job queue corruption that can occur
in rare instances due to issues with the shared storage mechanism.

Condor even provides all the tools necessary to hook into Cluster Suite,
including a new set of commands for the wallaby shell that make configuration
and management tasks as easy as a single command.  While a functioning wallaby
setup isn't required to work with Cluster Suite, I would highly recommended
using it.  The wallaby shell commands greatly simplify configuring both
Cluster Suite and condor nodes (through wallaby).

There are two tools that condor provides for integrating with Cluster
Suite.  One is the set of wallaby shell commands I already mentioned.  The
other is a Resource Agent for condor, which gives Cluster Suite control over
the schedd.

With the above pieces in place and a fully functional wallaby setup,
configuration of a schedd is as simple as:

    wallaby cluster-create name=<name> spool=<spool> server=<server> export=<export> node1 node2 ...

With that single command, the wallaby shell command will configure Cluster
Suite to run an HA schedd to run on the list of nodes provided.  It will also
configure those same nodes in wallaby to run an HA schedd.  Seems nice, but
what are the advantages?  Plenty.

You gain a lot of control over which node is running the schedd.  With
condor's native mechanism, it's pot luck which node will run the schedd.  All
nodes point to the same shared storage and whoever gets there first will run
the schedd.  Every time.  If a specific node is having problems that cause
the schedd to crash, it could continually win the race to run the schedd
leaving your highly available schedd not very available.

Cluster Suite doesn't rely upon the shared storage to determine which node
is going to run the schedd.  It has a set of tools, including a GUI, that
allow you to move a schedd from one node to another at any time.  In addition
to that, you can specify parameters that control when Cluster Suite will
decide to move the schedd to another node instead of restarting it on the
same machine.  For example, I can tell Cluster Suite to move the schedd to
another machine if it restarts 3 times in 60 seconds.

Cluster Suite also manages the shared storage.  I don't have to configure
each node to mount the shared storage at the same mount point and ensure it
will be mounted at boot.  Cluster Suite creates the mount point on the machine
and mounts the shared storage when it starts the schedd.  This means the
shared store is only mounted on the node running the schedd, which removes
the job queue corruption that can occur if 2 HA schedds run at the same time
on 2 different machines.

Having Cluster Suite manage the shared storage for an HA schedd provides
another benefit as well.  Access to the shared storage becomes required for
the schedd to run.  If there is an interruption in accessing the shared
storage on a node running the schedd Cluster Suite will shutdown the schedd
and start it on another node.  This means no more split brain.

Are there any downsides to using Cluster Suite to manage my schedds? Not many
actually.  Obviously you need to have Cluster Suite installed on each node
that will be part of an HA schedd configuration, so there's an additional
disk space/memory requirement.  The biggest issue I've found is that since
the condor_master will not be managing the schedds, none of the daemon
management commands will work (ie condor_on|off|restart, etc).  Instead you
would need to use Cluster Suite's tools for those tasks.

You will also have to setup fencing in Cluster Suite for everything to work
correctly, which might mean new hardware if you don't have a remotely
manageable power setup.  If Cluster Suite can't fence a node when it
determines it needs to it will shut down the service completely to avoid
corruption.  A way to handle this if you don't have the power setup is to
use virtual machines for your schedd nodes.  Cluster Suite has a means to do
fencing without needing an external power management setup for virtual machines.

