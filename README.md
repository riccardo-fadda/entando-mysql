MySQL 8.0 SQL Database Server container image
=============================================

This container image includes MySQL 8.0 SQL database server for  general usage.
Users can choose between RHEL8 and RockyLinux8 based images.

Description
-----------

This container image provides a containerized packaging of the MySQL mysqld daemon
and client application. The mysqld server daemon accepts connections from clients
and provides access to content from MySQL databases on behalf of the clients.

Usage
-----

If you want to set only the mandatory environment variables and not store
the database in a host directory, execute the following command:

```
$ podman run -d --name mysql_database -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -p 3306:3306 (Image:tag)
```

This will create a container named `mysql_database` running MySQL with database
`db` and user with credentials `user:pass`. Port 3306 will be exposed and mapped
to the host. If you want your database to be persistent across container executions,
also add a `-v /host/db/path:/var/lib/mysql/data` argument. This will be the MySQL
data directory.

If the database directory is not initialized, the entrypoint script will first
run [`mysql_install_db`](https://dev.mysql.com/doc/refman/en/mysql-install-db.html)
and setup necessary database users and passwords. After the database is initialized,
or if it was already present, `mysqld` is executed and will run as PID 1. You can
 stop the detached container by running `podman stop mysql_database`.


Environment variables and volumes
---------------------------------

The image recognizes the following environment variables that you can set during
initialization by passing `-e VAR=VALUE` to the Docker run command.

**`MYSQL_USER`**  
       User name for MySQL account to be created

**`MYSQL_PASSWORD`**  
       Password for the user account

**`MYSQL_DATABASE`**  
       Database name

**`MYSQL_ROOT_PASSWORD`**  
       Password for the root user (optional)

You can also set the following mount points by passing the `-v /host:/container` flag to Docker.

**`/var/lib/mysql/data`**  
       MySQL data directory


**Notice: When mouting a directory from the host into the container, ensure that the mounted
directory has the appropriate permissions and that the owner and group of the directory
matches the user UID or name which is running inside the container.**


MySQL auto-tuning
-----------------

When the MySQL image is run with the `--memory` parameter set and you didn't
specify value for some parameters, their values will be automatically
calculated based on the available memory.

**`MYSQL_KEY_BUFFER_SIZE (default: 10%)`**  
       `key_buffer_size`

**`MYSQL_READ_BUFFER_SIZE (default: 5%)`**  
       `read_buffer_size`

**`MYSQL_INNODB_BUFFER_POOL_SIZE (default: 50%)`**  
       `innodb_buffer_pool_size`

**`MYSQL_INNODB_LOG_FILE_SIZE (default: 15%)`**  
       `innodb_log_file_size`

**`MYSQL_INNODB_LOG_BUFFER_SIZE (default: 15%)`**  
       `innodb_log_buffer_size`



MySQL root user
---------------
The root user has no password set by default, only allowing local connections.
You can set it by setting the `MYSQL_ROOT_PASSWORD` environment variable. This
will allow you to login to the root account remotely. Local connections will
still not require a password.

To disable remote root access, simply unset `MYSQL_ROOT_PASSWORD` and restart
the container.


Changing passwords
------------------

Since passwords are part of the image configuration, the only supported method
to change passwords for the database user (`MYSQL_USER`) and root user is by
changing the environment variables `MYSQL_PASSWORD` and `MYSQL_ROOT_PASSWORD`,
respectively.

Changing database passwords through SQL statements or any way other than through
the environment variables aforementioned will cause a mismatch between the
values stored in the variables and the actual passwords. Whenever a database
container starts it will reset the passwords to the values stored in the
environment variables.