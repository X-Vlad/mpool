### Features

- Supports clusters of pools each running individual currencies
- Ultra-low-latency, multi-threaded Stratum implementation using asynchronous I/O
- Adaptive share difficulty ("vardiff")
- PoW validation (hashing) using native code for maximum performance
- Session management for purging DDoS/flood initiated zombie workers
- Payment processing
- Banning System
- Live Stats [API](https://github.com/AlphaX-Projects/alphaxcore/wiki/API) on Port 4000
- WebSocket streaming of notable events like Blocks found, Blocks unlocked, Payments and more
- POW (proof-of-work) & POS (proof-of-stake) support
- Detailed per-pool logging to console & filesystem
- Runs on Linux and Windows

### Supported Coins

Refer to [this file](/src/Alphaxcore/coins.json) for a complete list.

### Runtime Requirements on Linux

- [.Net Core 3.1 SDK](https://www.microsoft.com/net/download/core)
- [PostgreSQL Database](https://www.postgresql.org/)
- Coin Daemon (per pool)
- Alphaxcore needs to be built from source on Linux and Windows. Refer to the section further down below for instructions.

### Runtime Requirements on Windows

- [.Net Core 3.1 Runtime](https://www.microsoft.com/net/download/core)
- [PostgreSQL Database](https://www.postgresql.org/)
- Coin Daemon (per pool)

### Basic PostgreSQL Database setup

Create the database:

```console
$ createuser alphaxcore
$ createdb alphaxcore
$ psql (enter the password for postgres)
```

Inside psql execute:

```sql
alter user alphaxcore with encrypted password 'some-secure-password';
grant all privileges on database alphaxcore to alphaxcore;
```

Import the database schema:

```console
$ wget https://raw.githubusercontent.com/AlphaX-Projects/alphaxcore/master/src/Alphaxcore/Persistence/Postgres/Scripts/createdb.sql
$ psql -d alphaxcore -U alphaxcore -f createdb.sql
```

### Advanced PostgreSQL Database setup

If you are planning to run a Multipool-Cluster, the simple setup might not perform well enough under high load. In this case you are strongly advised to use PostgreSQL 11 or higher. After performing the steps outlined in the basic setup above, perform these additional steps:

**WARNING**: The following step will delete all recorded shares. Do **NOT** do this on a production pool unless you backup your <code>shares</code> table using <code>pg_backup</code> first!

```console
$ wget https://raw.githubusercontent.com/AlphaX-Projects/alphaxcore/master/src/Alphaxcore/Persistence/Postgres/Scripts/createdb_postgresql_11_appendix.sql
$ psql -d alphaxcore -U alphaxcore -f createdb_postgresql_11_appendix.sql
```

After executing the command, your <code>shares</code> table is now a [list-partitioned table](https://www.postgresql.org/docs/11/ddl-partitioning.html) which dramatically improves query performance, since almost all database operations Alphaxcore performs are scoped to a certain pool. 

The following step needs to performed **once for every new pool** you add to your cluster. Be sure to **replace all occurences** of <code>mypool1</code> in the statement below with the id of your pool from your Alphaxcore configuration file:

```sql
CREATE TABLE shares_mypool1 PARTITION OF shares FOR VALUES IN ('mypool1');
```

Once you have done this for all of your existing pools you should now restore your shares from backup.

### [Configuration](https://github.com/AlphaX-Projects/alphaxcore/wiki/Configuration)

### [API](https://github.com/AlphaX-Projects/alphaxcore/wiki/API)

### Building from Source

#### Building on Ubuntu 20.04

```console
$ wget -q https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb
$ sudo dpkg -i packages-microsoft-prod.deb
$ sudo apt-get update -y
$ sudo apt-get install apt-transport-https -y
$ sudo apt-get -y install dotnet-sdk-3.1 git cmake build-essential libssl-dev pkg-config libboost-all-dev libsodium-dev libzmq5
$ git clone https://github.com/AlphaX-Projects/alphaxcore
$ cd alphaxcore/src/Alphaxcore
$ dotnet publish -c Release --framework netcoreapp3.1  -o ../../build
```

#### Building on Windows

Download and install the [.Net Core 3.1 SDK](https://www.microsoft.com/net/download/core)

```dosbatch
> git clone https://github.com/AlphaX-Projects/alphaxcore
> cd alphaxcore/src/Alphaxcore
> dotnet publish -c Release --framework netcoreapp3.1  -o ..\..\build
```

#### Building on Windows - Visual Studio

- Download and install the [.Net Core 3.1 SDK](https://www.microsoft.com/net/download/core)
- Install [Visual Studio 2019](https://www.visualstudio.com/vs/). Visual Studio Community Edition is fine.
- Open `Alphaxcore.sln` in VS 2019


#### After successful build

Create a configuration file <code>config.json</code> as described [here](https://github.com/AlphaX-Projects/alphaxcore/wiki/Configuration)

```
cd ../../build
dotnet Alphaxcore.dll -c config.json
```

## Running a production pool

A public production pool requires a web-frontend for your users to check their hashrate, earnings etc. Alphaxcore does not include such frontend but there are several community projects that can be used as starting point.
