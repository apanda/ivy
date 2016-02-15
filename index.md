---
layout: default
title: Home
---
Thank you for your interest in Ivy. We provide Ivy as a [VirtualBox](https://www.virtualbox.org) [virtual machine](http://www.cs.tau.ac.il/~odedp/ivyvm.tar.bz2). Running our VM requires that at least 2 GB of memory be available. 

Let us get started by downloading and starting IVY:

1. Download and install [VirtualBox](https://www.virtualbox.org) for your VM.
2. Download[^1] the [VM](http://www.cs.tau.ac.il/~odedp/ivyvm.tar.bz2) archive.
3. Untar the archive, on Linux and OS X this can be done by running `tar -jxvf ivyvm.tar.bz2` in the directory to which you downloaded the archive. On Windows, you can use [7Zip](http://www.7-zip.org/download.html) to do so.
3. Once untarred, use VirtualBox to open the `ivyvm.vbox` file, and start the VM.
4. Once booted, log into the VM with the username `ivyuser` and password `ivy`.
5. Once logged in start Firefox (you can click on the icon on the left bar).
6. Next open a terminal and type `cd ~/ivy/examples/pldi16` and hit enter. This will take you to a directory containing several Ivy examples. 
7. Finally type `python ../../ivy/ivy2.py leader_election_ring.py` and hit enter. This should open a [Jupyter](http://jupyter.org/) notebook in Firefox, as shown in the figure below.

![Ivy started]({{ site.url }}/assets/jupyter.png)

Using Ivy to Find Inductive Invariants
--------------------------------------
Now that you have launched the Jupyter notebook used by Ivy, it is time to actually begin running Ivy. Begin by running the sheet (the button to do so is highlighted in red above). 

Now you are ready to begin finding an inductive invariant for this system. For this example we are going to be considering a [ring based leader election protocol](http://cs.ucsb.edu/~hatem/cs271/decentralized-extrema-finding.pdf) previous described by Chang and Roberts. In this protocol, each node is assigned a unique ID. Each node sends a message containing its ID to its neighbor. Nodes forwards a received message if and only if the ID contained in the message is greater than their own ID. Any node which receives its own message is then declared to be the leader.  In this example we check two conjectures: (1) exactly one node considers itself the leader, and (2) the leader has the highest ID in the ring.

We begin by clicking Check Inductiveness in the TransitionViewWidget to check if both these axioms are inductive. This should result in a screen shot similar to what is show below, indicating that conjecture 2 is not inductive.

![Leader ordering is not inductive]({{ site.url }}/assets/noninductive1.png)

We then highlight the set of conditions that seem to not fit into the model in this transition: check the boxes to see the `idn` (ID assigned to a node), `le` (ordering relation between IDs), `pending` (pending messages) and `leader` relations as shown below.

![Transition for non-inductiveness]({{ site.url }}/assets/ninductive-relations.png)

The problem here is that `node1` has a pending message with its ID, even though its ID is not the highest possible ID. This should be impossible, and we need to strengthen our model with this.

To begin strengthening, select all the arrows and nodes in the left transition window and click Gather Facts to collect all the relations that hold as shown below.

![Gather facts]({{ site.url }}/assets/gatherfacts1.png)

We can now use Bounded Model Checking to reduce these facts to a minimal form, using the Minimize Conjecture function in Ivy. Ivy should then produce a conjecture as shown below.

![New conjecture]({{ site.url }}/assets/conjecture1.png)

This conjecture in fact disallows the problem we identified before, and is hence what we wanted. We can now proceed by adding this to our list of conjectures by clicking on the Strengthen button in the Transition View.

We can then proceed by again trying to Check Inductiveness. This time Ivy indicates that the previously added conjecture is not inductive as shown below.

![Added conjecture is not inductive]({{ site.url }}/assets/ninductive2.png)

The problem is not immediately obvious this time, so we need to also show the `ring.le` relation, which corresponds with ring topology as shown below.

![Added conjecture is not inductive with more relations]({{ site.url }}/assets/ninductive2-more-rel.png)

Now the problem is apparent. Our previous conjecture does not require that messages follow the ring topology. In this case, node1 has received node0's ID, despite the fact that had this message followed the ring topology, it would have not have been received. We can again use Gather Facts and Minimize Conjecture to generate a conjecture corresponding to our observation as shown below.

![Conjecture 2]({{ site.url }}/assets/conjecture2.png)

We can again check inductiveness, resulting in another case where the previous conjecture is non-inductive because it doesn't account for some of the ring topology. The non-inductive conjecture and the conjecture we add to make it inductive are shown below.

![Added conjecture is not inductive]({{ site.url }}/assets/ninductive3.png)
![Conjecture 3]({{ site.url }}/assets/conjecture3.png)

Finally we need to repeat this process one more time to arrive at an inductive invariant that can be used to prove correctness for this system. When inductive invariants are found Ivy informs as shown below.

![Inductive invariants found]({{ site.url }}/assets/inductive1.png)

<hr />

[^1]: We recommend ensuring that the downloaded file is correct. The SHA sum for the VM archive is ``6d5d6b377f7a95a13507e57209c064080f273829``.
