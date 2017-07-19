---
layout: post
title: Data science tech support episode 4 PostgreSQL psql via ssh port forwarding
tags: PostgreSQL psql ssh ssh-port-forwarding
---

Want to use PostgreSQL remotely?  Want encrypt all communication to PostgreSQL?  Want to avoid touching your AWS security group to add open up the TCP 5432? Want to avoid having to update PostgreSQL with your IP address when it changes from work to home?


Modify your ssh client configuration such that the local host port 5432 are forwarded over ssh and then connect to that hosts 5432.  You'll need to edit your [~/.ssh/config file](https://man.openbsd.org/ssh_config) using your favorite editor.  Of course use different values for 'crh-aws-metis1' that can be whatever single word nickname you want this host to have. Update the 'ec2-user' to be the name of the host user on the AWS host. (I used the Amazon Linux Image so it is ec2-user.) Then update the '/home/crh/.ssh/crh-aws-metis1.pem' with the SSH private key file you generated for accessing AWS hosts. Update the 'ec2-34-227-52-231.compute-1.amazonaws.com' with the public host name of your AWS host after you turn it on. N.B. that'll change when you turn off/on.

```
~/.ssh/config
Host crh-aws-metis1
    User ec2-user
    IdentityFile /home/crh/.ssh/crh-aws-metis1.pem
    HostName ec2-34-227-52-231.compute-1.amazonaws.com
    LocalForward 5432 localhost:5432
```

Not strictly necessary. If your PostgreSQL is already listening on all IPs you can close that down.  I chose to make sure it was _only_ listening to the localhost IP and not listen to any others. Note your postgresql.conf file is likely in a different location.
```
[ec2-user@ip-172-31-3-251 ~]$ sudo diff /var/lib/pgsql95/data/postgresql.conf{.bak,}
58a59,60
> listen_addresses = 'localhost'		# what IP address(es) to listen on;
> #listen_addresses = '*'
```

Update the other configuration file too. I chose to do this because I was using a different user and I don't particularly trust the whole ident thing, preferring to use md5.

```
[ec2-user@ip-172-31-3-251 ~]$ sudo diff /var/lib/pgsql95/data/pg_hba.conf.bak /var/lib/pgsql95/data/pg_hba.conf
82c82,83
< host    all             all             127.0.0.1/32            ident
---
> #host    all             all             127.0.0.1/32            ident
> host    all             all             127.0.0.1/32            md5
84c85
< host    all             all             ::1/128                 ident
---
> # host    all             all             ::1/128                 ident

```

Restart the server (or possibly reload).
```
sudo service postgresql95 restart
```

Then just use localhost when you connect.
```
psql -U ec2-user -h localhost -d baseballdata
```

