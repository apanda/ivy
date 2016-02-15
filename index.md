---
layout: default
title: Home
---

<a name="overview"></a>Overview
-------------------------------

Thank you for your interest in Ivy, a tool for interactive verification of systems.

This webpage contains the artifact for the PLDI 2016 [paper about Ivy](http://www.cs.tau.ac.il/~odedp/pldi16-paper228.pdf), that describes the ideas and methodology behind Ivy.

Below, you will find instructions on how to download a VM that contains Ivy, and a walk-through of the running example from the paper --- verifying a simple leader election protocol.
The VM also contains all other examples mentioned in the paper (under the directory ``~/ivy/examples/pldi16``).

The paper describes a basic language called RML. The actual examples are written in Ivy's modeling language, 
which is essentially RML but with richer syntax.

<a name="downloading"></a>Downloading and Getting Started
---------------------------------------------------------
We provide Ivy as a [VirtualBox](https://www.virtualbox.org) [virtual machine](http://www.cs.tau.ac.il/~odedp/ivyvm.zip). Running our VM requires that at least 2 GB of memory be available.

Let us get started by downloading and starting IVY:

1. Download and install [VirtualBox](https://www.virtualbox.org) for your VM.
2. Download the [VM](http://www.cs.tau.ac.il/~odedp/ivyvm.zip) archive (1.5 GB, sha1sum: ``6918aaa78fe3a03bddc360d0ed1ee7a8a6aa0eac``).
3. Unzip the archive, on Linux and OS X this can be done by running `unzip ivyvm.zip` in the directory to which you downloaded the archive. On Windows, you can use [7Zip](http://www.7-zip.org/download.html) to do so.
3. Once unzipped, use VirtualBox to open the `ivyvm.vbox` file, and start the VM.
4. Once booted, log into the VM with the username `ivyuser` and password `ivy`.
5. Once logged in start Firefox (you can click on the icon on the left bar).
6. Next open a terminal and type `cd ~/ivy/examples/pldi16` and hit enter. This will take you to a directory containing the Ivy examples discussed in the paper.

<a name="bmc"></a>Using Bounded Model Checking to Check Models
---------------------------------------------------------------
We begin by looking at a [ring based leader election protocol](http://cs.ucsb.edu/~hatem/cs271/decentralized-extrema-finding.pdf) previously described by Chang and Roberts.
In this protocol, each node is assigned a unique ID. Each node sends a message containing its ID to its neighbor.
Nodes forwards a received message if and only if the ID contained in the message is greater than their own ID.
Any node which receives its own message is then declared to be the leader.
In this example we check two conjectures: (1) at most one node considers itself the leader, and (2) the leader has the highest ID in the ring.

We begin by looking at how Ivy can be used during the specification phase.
For this demonstration we have included a buggy version of the specification, where we forgot to mention that nodes have unique IDs.
This of course breaks the leader election protocol. To see how Ivy's bounded model checking can help with this run `python ../../ivy/ivy_bmc.py leader_election_ring_bug.ivy` in the previously opened terminal window. This should result in a [Jupyter](http://jupyter.org/) notebook opening in Firefox as shown below.

![Ivy started]({{ site.url }}/assets/bmc-highlighted.png)

Click on the run cell button (highlighted in the picture above) to start bounded model checking. This will shortly produce a violation as shown below

![BMC violation]({{ site.url }}/assets/bmc-start.png)

The Ivy Main window indicates that a counter example was found with 4 protocol actions transitions (the 0 to 1 transition is a dummy initialization step). Click on state 4 to see the transition just before the the violation. As shown below, the text box in the Ivy Main window shows the various relations leading up to this violation. We can also use the Transition View Widget to get a more visual representation. To do this click on the + icon in the Transition View Widget (highlighted below), this results in Ivy showing edges representing all binary relations in the model.

![BMC violation highlighted]({{ site.url }}/assets/bmc-selected-highlight.png)

You can examine the full trace by clicking on states 1 through 5 in the Ivy Main window. Whenever you click on a state, the Transition View Widget will display that state on the right hand side, and the previous state on the left hand side. Thus, clicking on a state reveals the transition that took place to reach that state, and the Transition View Widget always displays a concrete system transition, with the pre state on the left and the post state on the right.

As is readily apparent our model is wrong since both node0 and node1 in this example have the same ID. This can be fixed by adding an axiom to the model, which we have done in the file `leader_election_ring.ivy`. We use this in the next section.

<a name="inductive"></a>Finding an Inductive Invariant for Leader Election
-----------------------------------------------------------------------------
Now that we have a correct model, it is time to get inductive invariants for this system. To do this start by killing (using `ctrl-c`) the previous Jupyter session, and run `python ../../ivy/ivy2.py leader_election_ring.ivy`. This will again result in a Jupyter notebook opening. Once the notebook is open run the cell, this will again result in two windows: Ivy Main and Transition View Widget. We start by checking if our model is already inductive, to do this click the "Check Inductiveness" button in the Transition View Widget.
This instructs Ivy to check if our current set of conjectures is inductive.
This will result in Ivy finding a counter example showing that the invariant is not inductive, as shown below.

![Leader ordering is not inductive]({{ site.url }}/assets/noninductive1.png)

We then highlight the set of conditions that seem to not fit into the model in this transition: check the boxes to see the `idn` (ID assigned to a node), `le` (ordering relation between IDs), `pending` (pending messages) and `leader` relations as shown below.

![Transition for non-inductiveness]({{ site.url }}/assets/ninductive-relations.png)

The problem here is that `node1` has a pending message with its ID, even though its ID is not the highest possible ID. This should be impossible, and we need to strengthen our model with this.

To begin strengthening, click Gather Facts to collect all the relations that hold as shown below.

![Gather facts]({{ site.url }}/assets/gatherfacts1.png)

We can now use Bounded Model Checking to reduce these facts to a minimal form, using the Minimize Conjecture function in Ivy. Ivy should then produce a conjecture as shown below.

![New conjecture]({{ site.url }}/assets/conjecture1.png)

Ivy has found a stronger conjecture which holds after 3 protocol actions.
The display has switched to a special mode where only the facts (edges and labels) that participate in the minimized conjecture are depicted in the left hand side, and the nodes that participate in the conjecture are highlighted.
The dashed blue edge indicates an inequality that is part of the minimized conjecture (in normal display mode, inequality is implicit between any two distinct nodes).
Note that you might get a slightly different (but equivalent) conjecture, where the inequality edge (dashed blue) is between the IDs and not the nodes.
These two forms are equivalent since each node has a unique ID.
The minimized conjecture precisely states that a node cannot have a pending message with its own ID, if there is another node with a higher ID.
Therefore, Ivy correctly generalizes this counterexample to inductiveness.
We can now proceed by adding this conjecture to our list of conjectures by clicking on the Strengthen button in the Transition View.

We can then proceed by again trying to Check Inductiveness. This time Ivy indicates that the previously added conjecture is not inductive as shown below.

![Added conjecture is not inductive]({{ site.url }}/assets/ninductive2.png)

The problem is not immediately obvious this time, so we need to also show the `ring.le` relation, which corresponds with ring topology as shown below.

![Added conjecture is not inductive with more relations]({{ site.url }}/assets/ninductive2-more-rel.png)

Now the problem is apparent. Our previous conjecture does not require that messages follow the ring topology. In this case, node1 has received node0's ID, despite the fact that had this message followed the ring topology, it would not have been received. We can again use Gather Facts and Minimize Conjecture to generate a conjecture corresponding to our observation as shown below.

![Conjecture 2]({{ site.url }}/assets/conjecture2.png)

Observe that in the previous screenshot, `id0` is unselected and no green edges connect it to a node. This is to indicate that it was not included in the minimized conjecture. The dashed blue edge indicates that `id1` and `id2` are not equal, note that this is a part of the minimized conjecture. We proceed by strengthening with this conjecture (click the Strengthen button as before).

We can again check inductiveness, resulting in another case where the previous conjecture is non-inductive because it doesn't account for some of the ring topology. The non-inductive conjecture and the conjecture we add to make it inductive are shown below.

![Added conjecture is not inductive]({{ site.url }}/assets/ninductive3.png)
![Conjecture 3]({{ site.url }}/assets/conjecture3.png)

Finally we need to repeat this process one more time to arrive at an inductive invariant that can be used to prove correctness for this system. When an inductive invariant is found Ivy informs as shown below.

![Inductive invariants found]({{ site.url }}/assets/inductive1.png)

<a name="other_examples"></a>Other Examples
-----------------------------------------------------------------------------

The ``~/ivy/examples/pldi16`` directory contains all examples mentioned in the paper.
The examples contain the full inductive invariants, where the conjectures that were obtained interactively are at the end of the file.
To try to verify these examples, you can comment out some conjectures, and then go through the interactive verification process.
Some examples use ternary relations, which require projections to visualize. A ternary relation can be projected by right clicking on a node and clicking an "add ..." button.
A good place to try out projections is the ``leader_election_ring_btw.ivy`` example, which contains a version of the leader election with a ternary between relation,
which allows expressing the last 3 conjectures of the plain version with just one conjecture.

<hr />
