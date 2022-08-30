---
title: "Snapshot Rackspace CBS volume with Python Pyrax"
date: 2015-12-17T23:28:38+01:00
authors: ["David StClair"]
draft: false
---
This very simple example shows you how to take a snapshot of a Rackpace cloud volume.
This document makes the following assumptions:
-you have a Pyrax installed
-you have your Rackspace API credentials key.
-you know your Rackspace volume uid – which looks like this: 56568786-4659-67575-9ab3-6767968

1. Add the pyrax import
```bash
import pyrax
```
2. Prepare to initialize your pyrax by setting the identity type and add your rackspace api credentials

```bash
pyrax.set_setting("identity_type", "rackspace")
pyrax.set_credentials(os.getenv('PYRAX_USERNAME', os.getenv('PYRAX_API_KEY')) 
```
3. Create a pyrax connection to your rackspace cloud storage
```bash
cbs = pyrax.cloud_blockstorage
```
4. Create a snaphot of a selected volume, we supply the id of the volume we wish to snapshot and a name for the snapshot you are about to make. I am passing the id 56568786-4659-67575-9ab3-6767968, and the name test_snap:


snapshot = cbs.create_snapshot("56568786-4659-67575-9ab3-6767968", name="test_snap")
Note. If the volume you are attempting to snapshot is attached to an Cloud server or status “in-use” then you have to use the rather brutal sounding argument force=True

snapshot = cbs.create_snapshot("56568786-4659-67575-9ab3-6767968", name="test_snap", force=True)
5. Get feed back from the snapshot creation process

```bash
try:
    pyrax.utils.wait_until(snapshot, "status", "available", attempts=0, verbose=True)
    print "Snapshot Created"
except:
    pass
```
Here is is all together

```bash
import pyrax
 
pyrax.set_setting("identity_type", "rackspace")
pyrax.set_credentials(os.getenv('PYRAX_USERNAME', os.getenv('PYRAX_API_KEY'))  
 
cbs = pyrax.cloud_blockstorage
 
snapshot = cbs.create_snapshot("56568786-4659-67575-9ab3-6767968", name="test_snap")
try:
    pyrax.utils.wait_until(snapshot, "status", "available", attempts=0, verbose=True)
    print "Snapshot Created"
except:
    pass
```
Now you can run the code and create a snapshot – save the code into a file (I called mine create_snap.py) and then run, you should see some output like this:
```bash
python create_snap.py

Current value of status: creating (elapsed: 0.2 seconds)

…snipped…

Current value of status: creating (elapsed: 803.1 seconds)
Current value of status: creating (elapsed: 808.2 seconds)
Current value of status: creating (elapsed: 813.6 seconds)
Current value of status: creating (elapsed: 818.8 seconds)
Current value of status: creating (elapsed: 824.0 seconds)
Current value of status: creating (elapsed: 829.1 seconds)
Current value of status: creating (elapsed: 834.5 seconds)
Current value of status: creating (elapsed: 839.9 seconds)
Current value of status: available (elapsed: 845.0 seconds)
Snapshot Created
```