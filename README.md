tipper
======

Simple CLI for dumping schema from one mysql database and (optionally) configuring another with the dumped data.

### Requirements

- bash v4+
- source and target mysql server 5.4+ instances

### Example

The following command dumps data from the database `tipper_lorry` on `tipper.host.com` to the sql file `tipper.sql`.
It then creates that database on the local mysql host instance.

``` bash
$ ./tipper -p path/to/config -dcu

Setting configuration...
Dumping tipper_lorry from source on tipper.host.com to /path/to/working/directory/tipper.sql...
Creating database tipper_lorry on localhost from /path/to/working/directory/tipper.sql...
Done!
```

`tipper.host.com` is not a real server instance, so this won't work, but you can try it on your own mysql instances.

### Usage

`$ /path/to/tipper OPTIONS`


| Option    | Description |
|:----------|:------------|
| `-c`      | set this option to create database and schema from dump; if set with `-d`, dump will be performed first; will fail if dumpfile does not exist and  `-d` not set |
| `-d`      | set this option to dump schema from source |
| `-h`      | show help text and exit |
| `-p PATH` | path to config file used in this script (default: `/path/to/cwd/config`) |
| `-u`      | set this options to create user in target database if the user doesn't already exist |

### Config file spec

`tipper` reads its setting from a config file, which may be generated using `config.template` in this directory. Settings may also be set via environmnet variables, which always take precedence over config file settings. By default, tipper will look into `/path/to/cwd/config` for the config file, but you may pass in the config file location with the `-p` option when executing `tipper`.

| Config Setting               | Environment Variable            | Required                        | Description |
|:-----------------------------|:--------------------------------|:--------------------------------|:------------|
| `source_database`            | `TP_SOURCE_NAME`                | No                              | Name of database to dump; if omitted, all databases and all tables will be dumped |
| `source_host`                | `TP_SOURCE_HOST`                | Yes (if running with `-d`)      | Name of the database host to dump from |
| `source_user`                | `TP_SOURCE_USER`                | Yes (if running with `-d`)      | Name of the user to connect with when dumping from the source host |
| `source_password`            | `TP_SOURCE_PASSWORD`            | No                              | Password for the user you connect with if one is set |
| `source_tables`              | `TP_SOURCE_TABLES`              | No                              | Space-delimited list of tables to dump; ignored if `source_database` not set
| `source_dump_data`           | `TP_SOURCE_DUMP_DATA`           | No                              | Set to `true` to dump data for all tables
| `source_add_drop_database`   | `TP_SOURCE_ADD_DROP_DATABASE`   | No                              | Set to `true` to add `DROP DATABASE` statements before `CREATE DATABASE` statements |
| `source_add_drop_table`      | `TP_SOURCE_ADD_DROP_TABLE`      | No                              | Set to `true` to add `DROP TABLE` statements before `CREATE TABLE` statements |
| `target_file`                | `TP_TARGET_FILE`                | Yes                             | Path to file in which to write the dump and/or create a new database from |
| `target_host`                | `TP_TARGET_HOST`                | Yes (if running with `-c`/`-u`) | Name of the database host on which to create new database |
| `target_user`                | `TP_TARGET_USER`                | Yes                             | Name of user on the host with which to create new database |
| `target_password`            | `TP_TARGET_PASSWORD`            | No                              | Password for the user on the host with which to create new database if one is set |
| `target_root_user`           | `TP_TARGET_ROOT_USER`           | Yes (if running with `-u`)      | Name of admin user with which to create a new user on the target; only used with `-u` |
| `target_root_password`       | `TP_TARGET_ROOT_PASSWORD`       | No                              | Password of admin user with which to create a new user on the target; only used with `-u` |

The above settings map almost directly to `mysqldump` options. For now they represent a limited subset, but more will be added over time.

### Tips

- Much like `mysqldump`, you may specify either dumping all specified tables either with data or dumping all specified tables with no data. If you need a mixture, you should create two config files -- one that includes the tables you want with data and which tables you want without data, and invoke `tipper` once with one config file and once with the other.
- For now, `source_database` only allows one database. Without it, you get all databases. Future support may allow specifying multiple databases, but for now you should use multiple config files with multiple `tipper` executions for granular support of multiple databases.
