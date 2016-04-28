---
title: How To - Install MaxScale for MariaDB Galera
---

Recent project we implemented MaxScale to load balancing a cluster of MariaDB Galera servers. This solution previously used haproxy to evenly distribute the load across all the servers. The issue with haproxy is the site would get issues deadlock's. For a variety of reasons this solution required a way to split reads across the entire cluster but only write to a single node with failover. MaxScale fits this bill perfectly as it has intelligent SQL analysis of sql statements going though it.    

You and read more on the [MaxScale GitHub account](https://github.com/mariadb-corporation/MaxScale/wiki){:target="_blank"}.

First thing for the distribution you are using you need to [download the source, deb, rpm](https://downloads.mariadb.com/files/MaxScale){:target="_blank"}. We will be installing this example on Ubuntu 14.04 and the current version as of this post MaxScale 1.4.3. Substitute for the distribution and version of your choice.

What we are doing here is the following:
* Prep to download to /tmp
* In Ubuntu libaio1 is a dependency so it must be installed before MaxScale
* Download the latest deb file
* Install MaxScale
* Enable MaxScale to start on boot

{% highlight text %}
box:/# cd /tmp

box:/tmp# aptitude update; aptitude -y install libaio1

box:/tmp# wget https://downloads.mariadb.com/files/MaxScale/1.4.3/ubuntu/dists/trusty/main/binary-amd64/maxscale-1.4.2-1.ubuntu.trusty.x86_64.deb

box:/tmp# dpkg -i maxscale-1.4.3-1.ubuntu.trusty.x86_64.deb

box:/tmp# update-rc.d maxscale defaults
{% endhighlight %}

Now that we have MaxScale installed the next step is the configuration. The first thing we need to do is create a new MaxScale user. This user will login and get the credentials from the privileges table (you authenticate against MaxScale technically) and monitor each instance.

{% highlight text %}
MariaDB [(none)]> CREATE USER 'maxscale'@'%' IDENTIFIED BY 'you-password-here';
MariaDB [(none)]> GRANT SELECT ON mysql.db TO 'maxscale'@'%';
MariaDB [(none)]> GRANT SELECT ON mysql.user TO 'maxscale'@'%';
MariaDB [(none)]> GRANT SHOW DATABASES ON *.* TO 'maxscale'@'%';
{% endhighlight %}

Finally here is the working /etc/maxscale.cnf. I have included notes in the important configuration section.

{% highlight text %}
[maxscale]
threads=1 #number of CPU's here
syslog=0 #log to syslog
maxlog=1 #log to /var/log/maxscale
log_to_shm=1 #log to memory
log_warning=1 #log warnings
log_notice=1 #log notices
log_info=0 #log info
log_debug=0 #log full debug for dev only

[server1] #common name
type=server
address=172.16.20.125 #address
port=3306
protocol=MySQLBackend

[server2]
type=server
address=172.16.20.141
port=3306
protocol=MySQLBackend

[server3]
type=server
address=172.16.20.97
port=3306
protocol=MySQLBackend

[Galera Monitor]
type=monitor
module=galeramon
servers=server1,server2,server3 #which servers should be in the pool
user= #Add user dedicated MaxScale user
passwd= #Add the MaxScale password
monitor_interval=2000 #How frequent to check
disable_master_failback=1 #Once master fails stay don't fall back to old master
available_when_donor=1 #In Galera while in donor mode, continue to be available

[qla]
type=filter
module=qlafilter
options=/tmp/QueryLog

[fetch]
type=filter
module=regexfilter
match=fetch
replace=select

[RW Split Router]
type=service
router=readwritesplit
servers=server1,server2,server3 #which servers should be in the pool
user= #Same dedicated user from Galera Monitor
passwd= #MaxScale user password
max_slave_connections=100%
max_slave_replication_lag=30 #The amount of time to allow the slave to be beind

[CLI]
type=service
router=cli

[RW Split Listener]
type=listener
service=RW Split Router
protocol=MySQLClient
port=3306

[CLI Listener]
type=listener
service=CLI
protocol=maxscaled
address=127.0.0.1
port=6603
{% endhighlight %}

Now we can run the maxadmin commands to view the status. As you can see the three nodes are in the load balancer. Current server2 (172.16.20.141) is the current Master. You can test this by stopping MariaDB on the node marked Master and watch MaxScale automatically switch. 

{% highlight text %}
box:/tmp# maxadmin -pmariadb list servers
Servers.
-------------------+-----------------+-------+-------------+--------------------
Server             | Address         | Port  | Connections | Status              
-------------------+-----------------+-------+-------------+--------------------
server1            | 172.16.20.125   |  3306 |           0 | Slave, Synced, Running
server2            | 172.16.20.141   |  3306 |           0 | Master, Synced, Running
server3            | 172.16.20.97    |  3306 |           0 | Slave, Synced, Running
-------------------+-----------------+-------+-------------+--------------------
{% endhighlight %}
