---
layout: post
title: "Using Cluster Suite's GUI to configure High Availability Schedulers "
date: 2012-10-18 13:20
comments: true
categories: [condor, grid, grid computing, computing, Red Hat, MRG Grid, cluster_suite, cluster]
---
In an [earlier post](http://rrati.github.com/blog/2012/09/26/using-cluster-suite-to-manage-a-high-availability-scheduler/) I talked about using Cluster Suite
to manage high availability schedulers and referenced the command line tools
available perform the configuration.  I'd like to focus on using the GUI that
is part of Cluster Suite to configure an HA schedd.  It's a pretty simple
process but does require you run a wallaby shell command to complete the
configuration.

The first thing you need to do is create or import your cluster in the GUI.
If you already have a cluster in the GUI then make sure the nodes you want to
be part of a HA schedd configuration are part of the cluster.

The next step is to create a restricted Failover Domain.  Nodes in this domain
will run the schedd service you create, and making it restricted ensures that
no nodes outside the Failover Domain will run the service.  If a node in the
Failover Domain isn't available then the service won't run.

The third step is to create a service that will comprise your schedd. Make
sure that the relocation policy on the service is Relocate and that it is
configured to use whatever Failover Domain you have already setup.  The
service will contain 2 resources in a parent-child configuration.  The parent
service is the NFS Mount and the child service is a condor instance resource.
This is what sets up the dependency between the NFS Mount being required for
the condor instance to run.  When the resources are configured like this it
means the parent must be functioning for the child to operate.

Finally, you need to sync the cluster configuration with wallaby.  This is
easily accomplished by logging into a machine in the cluster and running:
	wallaby cluster-sync-to-store
That wallaby shell command will inspect the cluster configuration and
configure wallaby to match it.  It can handle any number of schedd
configurations so you don't need to run it once per setup.  However, until
the cluster-sync-to-store command is executed, the schedd service you created
can't and won't run.

Start your service or wait for Cluster Suite to do it for you and you'll find
an HA schedd in your pool.

You can get a video of the process as [ogv](http://rrati.fedorapeople.org/videos/cs_gui_schedd.ogv) or [mp4](http://rrati.fedorapeople.org/videos/cs_gui_schedd.mp4) if the inline video doesn't work.

{% video http://rrati.fedorapeople.org/videos/cs_gui_schedd.mp4 640 480 %}

