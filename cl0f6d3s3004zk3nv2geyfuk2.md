## Setting up PostgreSQL and configuring incoming connections

In this article, I'll give you a step-by-step guide on how to install the PostgreSQL database on Ubuntu 20.04.

First of all, you'll need to update your sources using


```
user@ubuntu:~$ sudo apt update
``` 

Once that's done go ahead and use the apt package manager to install postgrsql:


```
user@ubuntu:~$ sudo apt install postgresql postgresql-contrib
``` 

The above command will install the PostgreSQL database on your system. By default Postgres automatically creates a user with the name of `postgres`, this user has full superuser access.

To access the `psql` interface (psql is a command-line based front-end for Postgres 
you can use psql to interface with Postgres) use the command below:

```
user@ubuntu:~$ sudo -u postgres psql
```

or you can use the postgres user that was created by Postgres upon installation and use the psql command from there.

```
user@ubuntu:~$ su postgres
postgres@ubuntu:~$ psql
```

Notice how the first command `su postgres` changed the user to postgres. Once you run the `psql` command an interface similar to the one below will show up, this is where you can run SQL and Postgres specific queries.

```
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1))
Type "help" for help.

postgres=# 
```

## Creating a new database

You can use the command shown below to create a user first:

```
postgres=# CREATE USER '<username>' WITH ENCRYPTED PASSWORD '<new-password>';
```

The above command will create a new user with a username and a password, make sure to change those to whatever you prefer (don't use `<>`).

Once the user is created, we'll need to create a database.

```
postgres=# CREATE DATABASE '<db-name>'; 
``` 

The above command will create a new database (Make sure to change the `<db-name>`). Now we need to grant access to this database to the user we created in the previous step.

```
postgres=# GRANT ALL PRIVILEGES ON DATABASE '<db-name>' TO '<username>';
```

Now that our database is created and a user has access to that database the basic step is done. But if you want your database to be accessible everywhere or only to a specific IP, keep reading.

## Modifying connections to the database

Now that you have your database up and running you might want to modify who can have access to it. You can either allow everyone (which is not a good idea) or a specific set of IPs (using CIDR).

To achieve this we will need to edit the Postgres config which can be found in `/etc/postgresql/12/main/` path may vary based on Postgres version.

```
user@ubuntu:~$ cd /etc/postgresql/12/main/
user@ubuntu:~$ ls
conf.d  environment  pg_ctl.conf  pg_hba.conf  pg_ident.conf  postgresql.conf start.conf
```

As you can see there are many available files here, but the ones we are interested in are `postgresql.conf` and `pg_hba.conf`. Use the nano text editor to edit the postgresql.conf file.

```
user@ubuntu:~$ sudo nano postgresql.conf
```

You'll have to scroll down a bit until you see a section called `CONNECTIONS AND AUTHENTICATION` it will look something like this:

```
#------------------------------------------------------------------------------
# CONNECTIONS AND AUTHENTICATION
#------------------------------------------------------------------------------
# - Connection Settings -
listen_addresses = 'localhost,192.168.0.3'         # what IP address(es) to listen on;
```

You'll need to assign the list of IPs that you want to have access to Postgres to  `listen_addresses`. Alternatively, you can set listen_addresses equal to `'*'` which will mean that anyone on the internet can have access to our database (Bad idea).

Now save this file by pressing `ctrl+x` and tying `Y` and accepting the provided name.

Finally, we'll need to edit the `pg_hba.conf` file. This file controls the client authentication. We'll add our database and user and what IP can authenticate with them. Previously what we did was, we specified who can make a connection to the database.

Open the `pg_hba.conf` file using nano:

```
user@ubuntu:~$ sudo nano pg_hba.conf
```

At the bottom of the file specify your user and database and what IPs are allowed to authenticate.

```
host    username     db-name     192.168.0.0/24        md5
```

Make sure to change the username and db-name with the one you created and the IP address to the IP of your computer that you'll use to authenticate.

Finally, save the file using `ctrl+x` then `Y` and accept the provided file name.

You can use telnet command  to check if you can make a connection to the db:

```
user@ubuntu:~$ telnet 192.168.0.2 5241
```

Change the IP with the Postgres server IP. `5241` is the default port for postgres.


And that's it. Thanks for reading.