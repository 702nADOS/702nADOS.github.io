---
title: dom0-HW
keywords: network, deploy, offline, online
last_updated: October 19, 2017
tags: [dom0-HW]
summary: ""
sidebar: main_sidebar
permalink: dom0-HW.html
folder: "argOS"
---

# dom0-HW

dom0-HW is the network component within operating-system.
The confirguration of dom0-HW takes place inside the dom0-HW.run file in operating-system/genode-dom0-HW/run .

```
<network dhcp="no" ip-address="192.168.217.21" subnet-mask="255.255.255.0" default-gateway="192.168.217.1" port="3001"/>
```

This line tells dom0-HW which adress and ports to listen to.

## How to

Once the configuration of dom0-HW is done, it is possible to send binaries to the oparting-system.
Those binaries get meta information sent with them. Those meta infos look like this.

```
<periodictask>
<id>1</id>
<executiontime>999999999999</executiontime>
<criticaltime>0</criticaltime>
<ucfirmrt/>
<uawmean>
<size>10</size>
</uawmean>
<priority>127</priority>
<period>0</period>
<offset>0</offset>
<quota>5M</quota>
<pkg>tumatmul</pkg>
<config>
<arg1>21000</arg1>
</config>
</periodictask>
```

To send the binary and the description, the python script dom0_program.py is used.
Just run "python3 dom0_program.py" inside operating-system/toolchain-host/host_dom0/ .
A very simple example of dom0_program.py could look like this.

```
#!/usr/bin/env python3

from dom0_client import *
from dom0_sql import *
import time

session = Dom0_session('192.168.217.21', 3001)

session.read_tasks(script_dir + 'tasks.xml')
session.send_descs()
session.send_bins()

session.profile(script_dir + 'log'+str(i)+'.xml')

session.live(script_dir + 'log.xml')

```
Right on top, the ip of the genode machine is configured.
Furtheron the task description is read, so the python script knows which descriptions and binaries to send in the next step.
It is also possible to get information about the system.
First there is live(...) which returns an xml file including information about all threads online and their information.
Furthermore there is profile(...) which returns an xml file inclunding information about the events that happend since the last call.

The functions called in this file are stored in dom0_client.py.
dom0_client provides the following functions.

```
import socket
import code
import struct
import magicnumbers
import os
import re
import subprocess

script_dir = os.path.dirname(os.path.realpath(__file__)) + '/'

class Dom0_session:
	"""Manager for a connection to the dom0 server."""
	def __init__(self, host='192.168.217.20', port=3001):
		"""Initialize connection."""
	def connect(self, host='192.168.217.20', port=3001):
		"""Connect to the Genode dom0 server."""
	def read_tasks(self, tasks_file=script_dir+'tasks.xml'):
		"""Read XML file and enumerate binaries."""
	def send_descs(self):
		"""Send task descriptions to the dom0 server."""
	def send_bins(self):
		"""Send binary files to the dom0 server."""
	def start(self):
		"""Send message to start the tasks on the server."""
	def start_ex(self):
		"""Send task descriptions and binaries, and start the execution."""
	def stop(self):
		"""Send message to kill all tasks."""
	def clear(self):
		"""Send command to stop all tasks and clear the current task set."""
	def profile(self, log_file=script_dir+'log.xml'):
		"""Get profiling information about all running tasks."""
	def live(self, log_file=script_dir+'log.xml'):
		"""Get profiling information about all running tasks."""
	def close(self):
		"""Close connection."""
```



