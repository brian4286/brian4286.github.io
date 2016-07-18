---
title: How To - Restore MariaDB slave with XtraBackup
---

Lets look at a faster way to restore a [MySQL](http://www.mysql.com){:target="_blank"} (who still uses MySQL?) or the better [MariaDB](https://mariadb.org){:target="_blank"}, [Percona](https://www.percona.com){:target="_blank"} master/slave replication after a failure. Lets look at the old hard way...

{% highlight text %}
mysql> RESET MASTER;
mysql> FLUSH TABLES WITH READ LOCK;
mysql> SHOW MASTER STATUS;

mysqldump -uroot -p --opt  --master-data --add-drop-table --single-transaction --comments --hex-blob --dump-date --no-autocommit --all-databases > /tmp/all-databases.sql

mysql> UNLOCK TABLES;

scp /tmp/all-databases.sql root@slave:/tmp

mysql> STOP SLAVE;

mysql -uroot -p < /tmp/all-databases.sql

mysql> RESET SLAVE;
mysql> CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000022', MASTER_LOG_POS=20595391;
mysql> START SLAVE;
mysql> SHOW SLAVE STATUS;
{% endhighlight %}

mysqldump is just painful or me when a better alternative exists. So here comes [XtraBackup](https://www.percona.com/software/mysql-database/percona-xtrabackup){:target="_blank"} to the rescue. This is by far the fastest way to get replication running again, especially without any downtime.

First lets use XtraBackup's innobackupex wrapper to do a full dump of the master database. I am going to assume that you have your credentials saved in ~/.my.cnf. Depending on your open file limit and the number of tables you have you may need to set your ulimit. You can check it with ulimit -n. If necessary increase it ulimit -n {new value}. For most people this won't be a issue, but it creeps for me once and a while.

From the Master server, we will use innobackupex to dump the data. Then apply the logs needed for the restore and copy it over with rsync. As always with XtraBackup you are looking for the completed OK! message. I am going to assume since you have a Master/Slave setup you have ssh keys setup between the machines, you are using [Ed25519](/how-to-setup-ssh-ed25519-keys.html){:target="_blank"} keys right?

{% highlight text %}
innobackupex /tmp/mysql
...
innobackupex: completed OK!

innobackupex --apply-log /tmp/mysql
rsync -avz -e ssh /tmp/mysql root@{slave}:/tmp
{% endhighlight %}

Now we will be working from the slave. We will need to stop the MySQL server, as always verify it has properly stopped.

{% highlight text %}
mysqld stop
ps aux | grep mysql
{% endhighlight %}

Now I will make a backup of the old mysql data directory. Obviously if you have moved it, you will need to adjust the path.

{% highlight text %}
mv /var/lib/mysql /var/lib/mysql.bak
{% endhighlight %}

Now lets move the new MySQL data directory in-place and start the database.

{% highlight text %}
mv /tmp/mysql /var/lib/
chown mysql:mysql /var/lib/mysql
mysqld start
{% endhighlight %}

Now that we have the slave running it is still far behind, so we will need to get replication running again.

{% highlight text %}
cat /var/lib/mysql/xtrabackup_binlog_info
mysql-bin.000022     20595391
{% endhighlight %}

Now the key is we have the bin-log number and the exact position to start the replication again on the slave.

{% highlight text %}
mysql> RESET SLAVE;
mysql> CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000022', MASTER_LOG_POS=20595391;
mysql> START SLAVE;
mysql> SHOW SLAVE STATUS \G
{% endhighlight %}

You should now see that Slave_IO_Running: Yes, also pay attention to the seconds behind master.

If this is your first time working with XtraBackup you should read my how to.
