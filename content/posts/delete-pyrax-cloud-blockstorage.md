---
title: "Delete Pyrax Cloud Block Storage Volumes Snapshots Older than N days"
date: 2015-12-17T23:28:38+01:00
authors: ["David StClair"]
draft: false
---
If you find yourself with a lot of rackspace cloud volumes you need to remove, you can do this manually or you can run a simple python script.

This very simple example shows you how to delete snapshots taken of rackspace cloud volumes older than N days. In this example N days will be 7 days.

This example makes the following assumptions:
- you have a Pyrax installed
- you have your Rackspace API credentials key.
- you know your Rackspace volume uuid – which looks like this: 56568786-4659-67575-9ab3-6767968

```python {.myclass linenos=table,hl_lines=[8,"15-17"],linenostart=199}
import pyrax
from datetime import timedelta, datetime 
 
pyrax.set_setting("identity_type", "rackspace")
pyrax.set_credentials('daveid', some-cred-string )
 
cbs = pyrax.cloud_blockstorage
 
num_days=7 #Set this to number of days
snapshots = cbs.list_snapshots()
 
for snapshot in snapshots:
    create_date = snapshot.created_at
    create_date = create_date.split(".")[0]
    create_date = datetime.strptime(create_date, '%Y-%m-%dT%H:%M:%S')
    if datetime.now() - create_date &gt; timedelta(days = num_days):
        snapshot.delete()
```
