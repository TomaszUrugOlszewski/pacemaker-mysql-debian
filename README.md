# pacemaker-mysql-debian
Pacemaker agent for MySQL


What is it?
===========

Resource Agent for managing MySQL servers on Debian, leaving replication
managing to MySQL, and system administrator.

It's very minimalistic agent - can start, stop, and check MySQL status. And even
for those basic functions, it relies on /etc/init.d/mysql script. With
this agent you can manage your MySQL with /etc/init.d/mysql - even when 
pacemaker is turned off.

No resources migration, no replication management.


What is it for?
===============

To pin IP address to specific MySQL server in master-master replication, and do
automatic failover. 


How it works
============

MySQL-Debian resource position across cluster is constant, and it depends on
server-id parameter configured in MySQL server. You can't migrate mysql-debian
resources between nodes. So when for example MySQL die on crm1, it can't be
moved to crm2. It makes no sense, because on crm2 there is second mysql-debian
resource. It should be master-master replication.

- Before working with pacemaker, you need to manually create master-master replication between nodes.
- Create user for monitoring, with USAGE grant
- Set different server-id on both MySQL servers in my.cnf
- server-id in my.cnf must match with server-id in pacemaker resource


Usage
=====

Create resources, we assume that you have 2 nodes in cluster with server ids 1 and 2 - crm1 and crm2

`$ pcs resource create mysql_crm1 ocf:heartbeat:mysql-debian server_id=1 monitoruser_login=monitor monitoruser_pass=monitor123`

`$ pcs resource create mysql_crm2 ocf:heartbeat:mysql-debian server_id=2 monitoruser_login=monitor monitoruser_pass=monitor123`

Set preferences for above resources:

`$ pcs constraint location mysql_crm1 prefers crm1`

`$ pcs constraint location mysql_crm2 prefers crm2`

Set pereference for first IP address:

`$ pcs constraint colocation add mysql_vip_master with mysql_crm1 300`

`$ pcs constraint colocation add mysql_vip_master with mysql_crm2 250`

 Set preference for second IP address:

`$ pcs constraint colocation add mysql_vip_slave with mysql_crm2 200`

`$ pcs constraint colocation add mysql_vip_slave with mysql_crm1 150`
