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

[^1]: We recommend ensuring that the downloaded file is correct. The SHA sum for the VM archive is ``6d5d6b377f7a95a13507e57209c064080f273829``.