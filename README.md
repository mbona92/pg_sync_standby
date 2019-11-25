# pg_sync_standby

pg_sync_standby is a tool used to synchronize a postgres standby server with a master server using streaming replication.

It can be used to inizialize a new standby or to re-sync a standby that, for any reason, is no more in sync with master (for example if standby cannot catch required archives with restore command because they are already removed).


## Requirements

- Postgresql >= 9.0
- psql
- rsync
- lsof

## Usage

pg_sync_standby has the following options:

```bash
[mbona92@arch pg_sync_standby]$ ./pg_sync_standby --help
Usage:
   pg_sync_standby [OPTION]

Options:
  -c                     configuration file (default: pg_sync_standby.conf)
  -v                     set verbose output in rsync
  -L                     log file
  -l                     set backup label
  -e                     specify file containing dir or files to exclude from rsync
  -f                     fast checkpoint
  -o                     rsync options. Must be specified within double quotes
  -r                     specify recovery.conf file to copy within data directory
  -j                     use this many parallel jobs to rsync
  --bwlimit              specify bandwith limit according to the same option of rsync
  --start-slave          start database when sync finish
  --stop-slave           stop slave database
  --start-cmd            commad or bash script used to start slave. Must be specified within double quotes if contains spaces
  --stop-cmd             commad or bash script used to stop slave. Must be specified within double quotes if contains spaces
  --conf-dir             directory used to save configuration file. Must not be inside pgdata or tablespace
  --empty-slave-dirs     remove slave pgdata and datafile before start sync
  --new-config-file      generate new configuration file with specified name (default: pg_sync_standby.conf)

  --help                 display this help

Master pg connection options:
  -d                     connect to master database name
  -h                     master database server host or socket directory
  -p                     master database server port number
  -u                     connect as specified master database user

Slave pg connection options:
  -D                     connect to slave database name
  -H                     slave database server host or socket directory
  -P                     slave database server port number
  -U                     connect as specified slave database user

Master ssh connection options:
  --host                 hostname or ip address for ssh connection to master database host
  --user                 username for ssh connection to primary master host
```

## Configuration File

```bash
# Master connection options
m_pghost=
m_pgport=
m_pguser=
m_pgdatabase=

# Slave connection options
s_pghost=
s_pgport=
s_pguser=
s_pgdatabase=

# Sync options
m_host=
m_user=
exclude_file=
bwlimit=
rsync_opt=

# Other options
backup_label=                                       # Label passed to pg_start_backup() function
stop_slave=                                         # Stop slave before start sync
start_slave=                                        # Automatically start slave after sync is completed
start_cmd=                                          # Command or bash script used to start slave
stop_cmd=                                           # Command or bash script used to stop slave
fast_checkpoint=                                    # Force checkpoint as soon as possible in pg_start_backup()
bck_config_dir=                                     # Backup slave configuration file if it is started before start process
remove_slave_datafile=                              # Remove slave datafile before begin rsync
recovery_file=                                      # Specify a recovery.conf file to be used
max_jobs=                                           # Use this many parallel jobs to rsync
```
## Usage Example

```bash
[postgres@pgslave ~]$ /opt/postgres/bin/pg_sync_standby/pg_sync_standby -c pg_sync_standby.conf -j 4 --conf-dir /var/lib/psql/pgconf
INFO: Backing up configuration files...
INFO: Stopping slave database...
INFO: Executing pg_start_backup on master database...
INFO: /var/lib/pgsql/12/data successfully synced!
INFO: /var/lib/pgsql/tbs/data successfully synced!
INFO: /var/lib/pgsql/tbs/indexes successfully synced!
INFO: /var/lib/pgsql/tbs/data2 successfully synced!
INFO: /var/lib/pgsql/tbs/data3 successfully synced!
INFO: /var/lib/pgsql/tbs/data4 successfully synced!
INFO: /var/lib/pgsql/tbs/data5 successfully synced!
INFO: Executing pg_stop_backup on master database...
INFO: Restoring configuration files from /var/lib/psql/pgconf/...
INFO: Starting slave database...
```

