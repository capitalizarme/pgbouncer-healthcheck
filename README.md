# PGBouncer healthcheck

## Introduction

This implements a http `/health` check for PGBouncer hosts.
This healthcheck provides a simple yes/no health response based
on attempting to connect to PGBouncer and execute a request, as
well as checking the Datadog agent.

There are also several endpoints used for diagnostics.

## Building and running

This can be built on a local system with docker using the provided `Makefile`.

    $ make build

You can then run the server binary

    $ ./pgbouncer-healthcheck


## Main Endpoints

Note: See https://pgbouncer.github.io/usage.html for details of the PGBouncer
`SHOW` command outputs.

### /

This always returns a `200 OK`

### /health
The healthcheck returns a 200 OK response or a `500 Internal Server Error` response.
It checks the PGBouncer and also the Datadog Agent and requires both to be healthy.

## PGBouncer Information Endpoints

### /users
Returns the results of PGBouncer's `SHOW USERS` command formatted as JSON.

Example:

```json
[
  {
    "Name": "dd-agent",
    "PoolMode": null
  },
  {
    "Name": "jasoirwjofjdsslkmcknmwofn",
    "PoolMode": null
  },
  {
    "Name": "pgbouncer",
    "PoolMode": null
  },
  {
    "Name": "sidekiq_realtime",
    "PoolMode": null
  }
]
```

### /configs

Returns the results of PGBouncer's `SHOW CONFIG` command formatted as JSON.

Example:

```json
[
  {
    "Key": "job_name",
    "Value": "pgbouncer",
    "Changeable": false
  },
  {
    "Key": "conffile",
    "Value": "/etc/pgbouncer/pgbouncer.ini",
    "Changeable": true
  },
  {
    "Key": "logfile",
    "Value": "",
    "Changeable": true
  },
  {
    "Key": "pidfile",
    "Value": "/var/run/postgresql/pgbouncer.pid",
    "Changeable": false
  },
  {
    "Key": "listen_addr",
    "Value": "0.0.0.0",
    "Changeable": false
  },
  {
    "Key": "listen_port",
    "Value": "6543",
    "Changeable": false
  }
]
```
*(Truncated for brevity)*

### /databases

Returns the results of PGBouncer's `SHOW DATABASES` command formatted as JSON.
This is essentially the content of the `[databases]` section in the config file.

Example:

```json
[
  {
    "Name": "delivery_prod",
    "Host": "roo-prod-96.example.eu-west-1.rds.amazonaws.com",
    "Port": "12345",
    "Database": "prod",
    "ForceUser": null,
    "PoolSize": 100,
    "ReservePool": 1,
    "PoolMode": null,
    "MaxConnections": 0,
    "CurrentConnections": 0
  },
  {
    "Name": "delivery_prod_read1",
    "Host": "roo-prod-96-read1.example.eu-west-1.rds.amazonaws.com",
    "Port": "12345",
    "Database": "prod",
    "ForceUser": null,
    "PoolSize": 100,
    "ReservePool": 1,
    "PoolMode": null,
    "MaxConnections": 0,
    "CurrentConnections": 0
  }
]
```

### /pools

Returns the results of PGBouncer's `SHOW POOLS` command formatted as JSON.
This is similar to `SHOW DATABASES`, except it's a product of that set by
the users, since the pooling is per user per logical DB.

Example:

```json
[
  {
    "Database": "prod_read1_web",
    "User": "kldfwokoekrokerkreo",
    "ClActive": 459,
    "ClWaiting": 0,
    "SvActive": 2,
    "SvIdle": 5,
    "SvUsed": 0,
    "SvTested": 0,
    "SvLogin": 0,
    "MaxWait": 0,
    "PoolMode": "transaction"
  },
  {
    "Database": "prod_read1_worker",
    "User": "kldfwokoekrokerkreo",
    "ClActive": 127,
    "ClWaiting": 0,
    "SvActive": 0,
    "SvIdle": 4,
    "SvUsed": 0,
    "SvTested": 0,
    "SvLogin": 0,
    "MaxWait": 0,
    "PoolMode": "transaction"
  }
]
```

### /clients

Returns the results of PGBouncer's `SHOW CLIENTS` command formatted as JSON.

Example:

```json
  {
    "Type": "C",
    "User": "kldfwokoekrokerkreo",
    "Database": "prod_read1_web",
    "State": "active",
    "Addr": "10.0.129.60",
    "Port": 50962,
    "LocalAddr": "10.0.78.178",
    "LocalPort": 6543,
    "ConnectTime": "2018-11-05 08:18:34",
    "RequestTime": "2018-11-05 10:51:22",
    "Ptr": "0x55fb15292a18",
    "Link": "",
    "RemotePid": 0,
    "TLS": ""
  },
  {
    "Type": "C",
    "User": "kldfwokoekrokerkreo",
    "Database": "prod_read1_web",
    "State": "active",
    "Addr": "10.0.69.98",
    "Port": 39750,
    "LocalAddr": "10.0.78.178",
    "LocalPort": 6543,
    "ConnectTime": "2018-11-05 08:53:58",
    "RequestTime": "2018-11-05 10:18:28",
    "Ptr": "0x7f7159b2a420",
    "Link": "",
    "RemotePid": 0,
    "TLS": ""
  }
]
```

### /servers

Returns the results of PGBouncer's `SHOW SERVERS` command formatted as JSON.

Example:

```json
[
  {
    "Type": "S",
    "User": "kldfwokoekrokerkreo",
    "Database": "prod_read1_web",
    "State": "idle",
    "Addr": "34.254.85.1",
    "Port": 31872,
    "LocalAddr": "10.0.78.178",
    "LocalPort": 45492,
    "ConnectTime": "2018-11-05 12:03:12",
    "RequestTime": "2018-11-05 12:10:12",
    "Ptr": "0x55fb15589028",
    "Link": "",
    "RemotePid": 64451,
    "TLS": "TLSv1.2/ECDHE-RSA-AES256-GCM-SHA384/ECDH=prime256v1"
  },
  {
    "Type": "S",
    "User": "kldfwokoekrokerkreo",
    "Database": "prod_read1_web",
    "State": "idle",
    "Addr": "34.254.85.1",
    "Port": 31872,
    "LocalAddr": "10.0.78.178",
    "LocalPort": 44966,
    "ConnectTime": "2018-11-05 11:43:18",
    "RequestTime": "2018-11-05 12:10:12",
    "Ptr": "0x55fb159663e0",
    "Link": "",
    "RemotePid": 57603,
    "TLS": "TLSv1.2/ECDHE-RSA-AES256-GCM-SHA384/ECDH=prime256v1"
  }
]
```

### /mems

Returns the results of PGBouncer's `SHOW MEM` command formatted as JSON.

Example:

```json
[
  {
    "Name": "user_cache",
    "Size": 200,
    "Used": 5,
    "Free": 76,
    "MemTotal": 16200
  },
  {
    "Name": "db_cache",
    "Size": 208,
    "Used": 21,
    "Free": 57,
    "MemTotal": 16224
  },
  {
    "Name": "pool_cache",
    "Size": 408,
    "Used": 11,
    "Free": 39,
    "MemTotal": 20400
  },
  {
    "Name": "server_cache",
    "Size": 392,
    "Used": 78,
    "Free": 322,
    "MemTotal": 156800
  },
  {
    "Name": "client_cache",
    "Size": 392,
    "Used": 2708,
    "Free": 10092,
    "MemTotal": 5017600
  },
  {
    "Name": "iobuf_cache",
    "Size": 4112,
    "Used": 3,
    "Free": 797,
    "MemTotal": 3289600
  }
]
```

## Server Debug Endpoints

These were added to help debug unreachable servers.
                 
### /debug/dmesg

Returns the raw output of `dmesg`

### /debug/processes

Returns the raw output of `ps -eo user,pid,ppid,c,stime,tty,%cpu,%mem,vsz,rsz,cmd`

### /debug/meminfo

Returns the contents of `/proc/meminfo`.

### /debug/logs

Returns the output of `journalctl --reverse -b --no-pager -n 50`

