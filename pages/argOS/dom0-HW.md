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

dom0-HW is a network component within operating-system.
The confirguration of dom0-HW takes place inside the dom0-HW.run file in operating-system/genode-dom0-HW/run .

```
<network dhcp="no" ip-address="192.168.218.21" subnet-mask="255.255.255.0" default-gateway="192.168.218.1" port="3001"/>
```

This line tells dom0-HW which adress and ports to listen to.

## How to

Once the configuration of dom0-HW is done, it is possible to send binaries to the oparting-system.
Those binaries get meta information sent with them. Those binaries look like this.

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


