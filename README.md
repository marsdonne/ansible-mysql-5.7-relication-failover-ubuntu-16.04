# Ansible role for installing Mysql 5.7 master-slave with gtid and automatic failover

This Ansible role should
1) add [Oracle repo](https://dev.mysql.com/downloads/repo/apt/) into servers
2) protect hosts with [UWF firewall](https://en.wikipedia.org/wiki/Uncomplicated_Firewall)
3) install Mysql 5.7 packages
4) configure [mysql master-slave replication](https://dev.mysql.com/doc/refman/5.7/en/replication.html) (replica seed is possible)
5) install [Haproxy](https://en.wikipedia.org/wiki/HAProxy) as mysql frontend
6) install [Orchestrator](https://github.com/github/orchestrator) for automatic failover and simply mysql replication management  
Note: resolving hostnames important for orchestrator  
Note: support orchestrator raft, proxy by haproxy on port 3001
7) add mysqldump into root cron on all mysql servers that will take full backup from mysql slave daily
8) install [Keepalived](https://keepalived.org) for virtual high-available IP address (VIP) and single entry point for databases access.

Tested with ubuntu 16.04, centos 7 and ansible 2.4

**Important**  
For avoiding split brains I recommend minimum setup of master + 2 slaves  
But this role would work even with master + 1 slave, master initially marked as backup in haproxy exactly for this type of setup

**Another Important**  
By default all servers contain read-only=1 in my.cnf, master would be changed in runtime after setup.
So you'll get read only master after any restart except orchestrator auto failover

Minimum variables that you should know and set:

group_vars/all.yml
```
ansible_user: mars

frontend_mysql_master_port: 3310
frontend_mysql_slave_port: 3311
mysql_port: 3306

mysql_replication_user:
  name: replicator
  host: '%'
  password: '_strong_password_'
mysql_root_user:
  name: root
  host: '%'
  password: '_strong_password_'
```

How to test:
1) start vagrant vms
```
vagrant up
```
2) run ansible playbook
```
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook --ask-pass --ask-sudo-pass -i hosts mysql.yml
```

Manually recover Dead-Master to slave:

```
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook replicate.yml --ask-pass --ask-sudo-pass -e '{"master":"192.168.56.123","slave":"192.168.56.111"}'
```
