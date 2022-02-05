MySQL 8.0 SQL Database Server container image
=============================================

This container image includes MySQL 8.0 SQL database server for  general usage.
Users can choose between RHEL8 and Rokylinux8 based images.

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

The following environment variables influence the MySQL configuration file. They are all optional.

**`MYSQL_LOWER_CASE_TABLE_NAMES (default: 0)`**  
       Sets how the table names are stored and compared

**`MYSQL_MAX_CONNECTIONS (default: 151)`**  
       The maximum permitted number of simultaneous client connections

**`MYSQL_MAX_ALLOWED_PACKET (default: 200M)`**  
       The maximum size of one packet or any generated/intermediate string

**`MYSQL_FT_MIN_WORD_LEN (default: 4)`**  
       The minimum length of the word to be included in a FULLTEXT index

**`MYSQL_FT_MAX_WORD_LEN (default: 20)`**  
       The maximum length of the word to be included in a FULLTEXT index

**`MYSQL_AIO (default: 1)`**  
       Controls the `innodb_use_native_aio` setting value in case the native AIO is broken. See http://help.directadmin.com/item.php?id=529

**`MYSQL_TABLE_OPEN_CACHE (default: 400)`**  
       The number of open tables for all threads

**`MYSQL_KEY_BUFFER_SIZE (default: 32M or 10% of available memory)`**  
       The size of the buffer used for index blocks

**`MYSQL_SORT_BUFFER_SIZE (default: 256K)`**  
       The size of the buffer used for sorting

**`MYSQL_READ_BUFFER_SIZE (default: 8M or 5% of available memory)`**  
       The size of the buffer used for a sequential scan

**`MYSQL_INNODB_BUFFER_POOL_SIZE (default: 32M or 50% of available memory)`**  
       The size of the buffer pool where InnoDB caches table and index data

**`MYSQL_INNODB_LOG_FILE_SIZE (default: 8M or 15% of available memory)`**  
       The size of each log file in a log group

**`MYSQL_INNODB_LOG_BUFFER_SIZE (default: 8M or 15% of available memory)`**  
       The size of the buffer that InnoDB uses to write to the log files on disk

**`MYSQL_DEFAULTS_FILE (default: /etc/my.cnf)`**  
       Point to an alternative configuration file

**`MYSQL_BINLOG_FORMAT (default: statement)`**  
       Set sets the binlog format, supported values are `row` and `statement`

**`MYSQL_LOG_QUERIES_ENABLED (default: 0)`**  
       To enable query logging set this to `1`

**`MYSQL_DEFAULT_AUTHENTICATION_PLUGIN (default: caching_sha2_password)`**  
       Set default authentication plugin. Accepts values `mysql_native_password` or `caching_sha2_password`.

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