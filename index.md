---
layout: default
title: Home
---
Thank you for your interest in Ivy. We provide Ivy as a [VirtualBox](https://www.virtualbox.org) [virtual machine](http://www.cs.tau.ac.il/~odedp/ivyvm.tar.bz2). Running our VM requires that at least 2 GB of memory be available. 

Let us get started by downloading and starting IVY:

1. Download and install [VirtualBox](https://www.virtualbox.org) for your VM.
2. Download[^1] the [VM](http://www.cs.tau.ac.il/~odedp/ivyvm.tar.bz2) archive.
3. Untar the archive, on Linux and OS X this can be done by running ```tar -jxvf ivyvm.tar.bz2``` in the directory to which you downloaded the archive. On Windows, you can use [7Zip](http://www.7-zip.org/download.html) to do so.
3. Once untarred, use VirtualBox to open the ```ivyvm.vbox``` file, and start the VM.
4. Once booted, log into the VM with the username ```ivyuser``` and password ```ivy```.
5. Once logged in start Firefox (you can click on the icon on the left bar).
6. Next open a terminal and type ```cd ~/ivy/examples/pldi16``` and hit enter. This will take you to a directory containing several Ivy examples. 
7. Finally type ```python ../../ivy/ivy2.py leader_election_ring.py``` and hit enter. This should open a [Jupyter](http://jupyter.org/) notebook in Firefox, as shown in the figure below.

![Ivy started]({{ site.url }}/assets/jupyter.png)

Using Ivy to Find Inductive Invariants
--------------------------------------
Now that you have launched the Jupyter notebook used by Ivy, it is time to actually begin running Ivy. Begin by running the sheet (the button to do so is highlighted in red above). 

Now you are ready to begin finding an inductive invariant for this system. For this example we are going to be considering a [ring based leader election protocol](http://cs.ucsb.edu/~hatem/cs271/decentralized-extrema-finding.pdf) previous described by Chang and Roberts. In this protocol, each node is assigned a unique ID. Each node sends a message containing its ID to its neighbor. Nodes forwards a received message if and only if the ID contained in the message is greater than their own ID. Any node which receives its own message is then declared to be the leader.  In this example we check two conjectures: (1) exactly one node considers itself the leader, and (2) the leader has the highest ID in the ring.

We begin by clicking Check Inductiveness in the TransitionViewWidget to check if both these axioms are inductive. This should result in a screen shot similar to what is show below, indicating that conjecture 2 is not inductive.

![Leader ordering is not inductive]({{ site.url }}/assets/noninductive1.png)

We then highlight the set of conditions that seem to not fit into the model in this transition: check the boxes to see the ```idn``` (ID assigned to a node), ```le``` (ordering relation between IDs), ```pending``` (pending messages) and ```leader``` relations as shown below.

![Transition for non-inductiveness]({{ site.url }}/assets/ninductive-relations.png)

This indicates that ```node1``` had a pending message when it transitioned to leader, causing this violation. Next select all the arrows and nodes in the left transition window and click Gather Facts to collect all the relations that hold as shown below.

![Gather facts]({{ site.url }}/assets/gatherfacts1.png)




[^1]: We recommend ensuring that the downloaded file is correct. The SHA sum for the VM archive is ``6d5d6b377f7a95a13507e57209c064080f273829``.