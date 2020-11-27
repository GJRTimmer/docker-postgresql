[![build status](https://gitlab.timmertech.nl/docker/postgresql/badges/master/pipeline.svg)](https://gitlab.timmertech.nl/docker/postgresql/commits/master)
[![](https://images.microbadger.com/badges/image/datacore/postgresql.svg)](https://microbadger.com/images/datacore/postgresql)
[![](https://images.microbadger.com/badges/license/datacore/postgresql.svg)](https://microbadger.com/images/datacore/postgresql)

Docker image for running a PostgreSQL server

- [Introduction](#introduction)
- [Installation](#installation)
  - [Volume Locations](#volume-locations)
  - [Configuration Options](#configuration-options)
    - [General Options](#general-options)
    - [Timezone](#timezone)
    - [UID/GID mapping](#uidgid-mapping)
    - [Database Options](#database-options)
  - [Creating databases](#creating-databases)
  - [Database Initialization](#database-initialization)
    - [Example](#example)
- [Enabling extensions](#enabling-extensions)
  - [Creating Snapshot](#creating-snapshot)
  - [Creating Backup](#creating-backup)

# Introduction

`Dockerfile` to create a [Docker](https://www.docker.com/) container image for [PostgreSQL](http://postgresql.org/).

PostgreSQL is an object-relational database management system (ORDBMS) with an emphasis on extensibility and standards-compliance [[source](https://en.wikipedia.org/wiki/PostgreSQL)].

# Installation

<details>
<summary>Install from DockerHub</summary>
<p>

Download:

```bash
docker pull datacore/postgresql:latest
```

Build:

```bash
git clone https://github.com/gjrtimmer/docker-postgresql.git
cd docker-postgresql
docker build --build-arg=PGV=$(cat VERSION) -t datacore/postgresql .
```

</p>
</details>

<br/>

<details>
<summary>Install from TimmerTech</summary>
<p>

Download:

```bash
docker pull registry.timmertech.nl/docker/postgresql:latest
```

Build:

```bash
git clone https://gitlab.timmertech.nl/docker/postgresql.git
cd postgresql
docker build --build-arg=PGV=$(cat VERSION) -t datacore/postgresql .
```

</p>
</details>

## Volume Locations

Default Locations:

| ENV | Default Location | Description |
|-----|------------------|-------------|
| ```PG_HOME``` | /var/lib/postgresql | Data directory |
| ```PG_LOGDIR``` | /var/log/postgresql | Log directory |
| ```PG_RUNDIR``` | /var/run/postgresql | Run directory |
| ```PG_CERTDIR``` | /etc/postgresql/certs | Certificate directory | 
| ```PG_INIT_DB``` | /etc/postgresql/init.d | Database initialization scripts |

## Configuration Options

<details>
<summary>General</summary>

### General Options

<p>

| Option | Default | Description |
|--------|---------|-------------|
| [TZ](#timezone) | UTC | Set timezone, example ```Europe/Amsterdam``` |
| [PG_UID](#uidgid-mapping) `UID` | postgres | Map ownership to UID |
| [PG_GID](#uidgid-mapping) `GID` | postgres | Map ownership to GID |
</p>
</details>

<br/>

<details>
<summary>Timezone</summary>

### Timezone

<p>

Set the timezone for the container, defaults to ```UTC```.
To set the timezone set the desired timezone with the variable ```TZ```.

Example:

````bash
docker run --name postgresql -itd --restart always \
  --env 'TZ=Europe/Amsterdam' \
  registry.timmertech.nl/docker/postgresql:latest
````

</p>
</details>

<br />

<details>
<summary>UID / GID Mapping</summary>

### UID/GID mapping

<p>

The files and processes created by the container are owned by the `postgres` user that is internal to the container. In the absense of user namespace in docker the UID and GID of the containers `postgres` user may have different meaning on the host.

For example, a user on the host with the same UID and/or GID as the `postgres` user of the container will be able to access the data in the persistent volumes mounted from the host as well as be able to KILL the `postgres` server process started by the container.

To circumvent this issue you can specify the UID and GID for the `postgres` user of the container using the `PG_UID` and `PG_GID` variables respectively.

For example, if you want to assign the `postgres` user of the container the UID and GID `999`:

```bash
docker run --name postgresql -itd --restart always \
  --env 'PG_UID=999' --env 'PG_GID=999' \
  registry.timmertech.nl/docker/postgresql:12.5
```

</p>
</details>

<br/>

<details>
<summary>Database Options</summary>

### Database Options

<p>

| Option | Description |
|--------|-------------|
| PG_PASSWORD | Password for `postgres` user |
| DB_USER | Username for database(s) provided with `DB_NAME` |
| DB_PASS | Password for database(s) provided with `DB_NAME` |
| [DB_NAME](#creating-databases) `NAME` | Database(s) to create, multiple can be provided separated with a comma `,` |
| [DB_TEMPLATE](http://www.postgresql.org/docs/12.5/static/manage-ag-templatedbs.html) `TEMPLATE` | Template to use for newly created database(s) [Template Databases](http://www.postgresql.org/docs/12.5/static/manage-ag-templatedbs.html) |
| [DB_EXTENSION](#enabling-extensions) `EXTENSION` | Extension to enable for database(s) within `DB_NAME`, multiple can be provided separated with a comma `,` |
| PL_PERL | PL/Perl language extension, ```true``` to enable, default: ```false``` |
| PL_PYTHON | PL/Python3 language extension, ```true``` to enable, default: ```false``` |
| PL_TCL | PL/Tcl language extension, ```true``` to enable, default: ```false``` | 
| PG_CRON | Cron scheduler for postgresql, ```true``` to enable, default: ```false``` |
| PG_CRON_DB | Cron scheduler database `PG_CRON_DB`, default: ```postgres``` |

</p>
</details>

## Creating databases

A new PostgreSQL database can be created by specifying the `DB_NAME` variable while starting the container.

```bash
docker run --name postgresql -itd --restart always \
  --env 'DB_NAME=dbname' \
  registry.timmertech.nl/docker/postgresql:12.5
```

By default databases are created by copying the standard system database named `template1`. You can specify a different template for your database using the `DB_TEMPLATE` parameter. Refer to [Template Databases](http://www.postgresql.org/docs/12.5/static/manage-ag-templatedbs.html) for further information.

Additionally, more than one database can be created by specifying a comma separated list of database names in `DB_NAME`. For example, the following command creates two new databases named `dbname1` and `dbname2`.

```bash
docker run --name postgresql -itd --restart always \
  --env 'DB_NAME=dbname1,dbname2' \
  registry.timmertech.nl/docker/postgresql:12.5
```

## Database Initialization

Because this image is able to initalize multiple database; in several cases you want to be able to initialize a database with tables / indexes etc...

For this scenario this image provides a volume location for database initialization ```PG_INIT_DB```.
In this volume location sql files are expected with the following format ```<ID>-<DB_NAME>-<additional name>.sql```.

The ID is an id to order the files for loading in a specific order.

<details>
<summary>Example</summary>

### Example

<p>

```bash
10-postgres-create-tables.sql
```

</p>
</details>


# Enabling extensions

The image also packages the [postgres contrib module](http://www.postgresql.org/docs/12.5/static/contrib.html). A comma separated list of modules can be specified using the `DB_EXTENSION` parameter.

```bash
docker run --name postgresql -itd \
  --env 'DB_NAME=db1,db2' --env 'DB_EXTENSION=unaccent,pg_trgm' \
  registry.timmertech.nl/docker/postgresql:12.5
```

The above command enables the `unaccent` and `pg_trgm` modules on the databases listed in `DB_NAME`, namely `db1` and `db2`.

## Creating Snapshot

> **Untested**
> Reason: S6 Implementation

Similar to a creating replication slave node, you can create a snapshot of the master by specifying `REPLICATION_MODE=snapshot`.

Once the master node is created as specified in [Setting up a replication cluster](#setting-up-a-replication-cluster), you can create a snapshot using:

```bash
docker run --name postgresql-snapshot -itd --restart always \
  --link postgresql-master:master \
  --env 'REPLICATION_MODE=snapshot' --env 'REPLICATION_SSLMODE=prefer' \
  --env 'REPLICATION_HOST=master' --env 'REPLICATION_PORT=5432'  \
  --env 'REPLICATION_USER=repluser' --env 'REPLICATION_PASS=repluserpass' \
  registry.timmertech.nl/docker/postgresql:12.5
```

The difference between a slave and a snapshot is that a slave is read-only and updated whenever the master data is updated (streaming replication), while a snapshot is read-write and is not updated after the initial snapshot of the data from the master.

This is useful for developers to quickly snapshot the current state of a live database and use it for development/debugging purposes without altering the database on the live instance.

## Creating Backup

> **Untested**
> Reason: S6 Implementation

Just as the case of setting up a slave node or generating a snapshot, you can also create a backup of the data on the master by specifying `REPLICATION_MODE=backup`.

> The backups are generated with [pg_basebackup](http://www.postgresql.org/docs/12.5/static/app-pgbasebackup.html) using the replication protocol.

Once the master node is created as specified in [Setting up a replication cluster](#setting-up-a-replication-cluster), you can create a point-in-time backup using:

```bash
docker run --name postgresql-backup -it --rm \
  --link postgresql-master:master \
  --env 'REPLICATION_MODE=backup' --env 'REPLICATION_SSLMODE=prefer' \
  --env 'REPLICATION_HOST=master' --env 'REPLICATION_PORT=5432'  \
  --env 'REPLICATION_USER=repluser' --env 'REPLICATION_PASS=repluserpass' \
  --volume /srv/docker/backups/postgresql.$(date +%Y%m%d%H%M%S):/var/lib/postgresql \
  registry.timmertech.nl/docker/postgresql:12.5
```

Once the backup is generated, the container will exit and the backup of the master data will be available at `/srv/docker/backups/postgresql.XXXXXXXXXXXX/`. Restoring the backup involves starting a container with the data in `/srv/docker/backups/postgresql.XXXXXXXXXXXX`.
