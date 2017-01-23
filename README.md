# Description

- This script is a wrapper for Percona's [**pt-table-sync**](https://www.percona.com/doc/percona-toolkit/2.2/pt-table-sync.html) to sync data between databases/tables.
  - **pt-table-sync** is part of [Percona Toolkit](https://www.percona.com/doc/percona-toolkit/).
- I wrote this to assist with live migrating a client whose various tables contained over 2 billion rows to a new database cluster.
- The problem I was encountering was I kept getting **deadlocks** and the replication threads kept killing my **pt-table-sync** command when the replication thread applied a commit that modified the table currently syncing.
- The script will automatically break up the table being synced (if possible) and then sync it over in increments to drastically reduce the chance of **deadlocks**.
- The script detects the primary key on the table given, determines the start/end points, and then syncs it in sections.
  - By default it'll do 200000 entries at a time (can be changed via **-C value**).

# Initial Usage (more coming soon)

- Here is the current help menu for the script:

```bash
Here are the available options:
 -h                     (-h) Displays the help output.
 -d SRCDB               (-d) Controls the source database to sync from.
 -D DESTDB              (-D) Controls the destination database to sync to.
                                If the src/dest databases are the same name,
                                  specify '-d' or '-D' (one or the other).
                                The script will set both to be the same when
                                  only one option is specified.
                                Setting both won't cause any issues.
 -t SRCTBL              (-t) Controls the source table to sync from.
 -T DESTTBL             (-T) Controls the destination table to sync to.
                                If the src/dest tables are the same name,
                                  specify '-t' or '-T' (one or the other).
                                The script will set both to be the same when
                                  only one option is specified.
                                Setting both won't cause any issues.
 -t ALL or -T ALL       (-t/-T) 'ALL' has special meaning within this script.
                                If you specify 'ALL' for the table, then
                                  the script will loop through and sync
                                  every table within the specified database.
 -p                     (-p) Prompts for the source MySQL password (for the user specified).
 -P                     (-P) Prompts for the destination MySQL password (for the user specified).
                                If source and destination passwords are the same,
                                  just call '-p' or '-P' (one or the other).
                                  It'll set both passwords to be the same by
                                  default unless you call both.
 -f /full/path/to/file  (-f) Sets the source MySQL password from the file provided.
 -F /full/path/to/file  (-F) Sets destination MySQL password from the file provided.
                                Files should be in '.my.cnf' format, ie 'password=PASS'.
                                If source/destination passwords are the same,
                                  just call '-f' or '-F' once.
                                  It'll set both passwords to be the same by
                                  default unless you call both.
                                Options '-p' and '-f' are mutually exclusive, as are
                                  '-P' and '-F'. However, you can mix them, ie
                                  '-p' with '-F' or vice versa.
 -u SRCUSER             (-u) Sets the source MySQL user (default: root).
 -U DESTUSER            (-U) Sets the destination MySQL user (default: root).
                                If both source/dest users are the same, call just
                                  '-u' or '-U' as it'll set both to be the same.
 -i SRCIP               (-i) Controls source IP/Hostname to sync from (default: localhost).
 -I DESTIP              (-I) Controls destination IP/Hostname to sync to.
 -o SRCPORT             (-o) Changes source MySQL port (default: 3306).
 -O DESTPORT            (-O) Changes destination MySQL port (default: 3306).
 -s /path/to/socket     (-s) Source MySQL Socket to connect through (default: /var/lib/mysql/mysql.sock).
 -S /path/to/socket     (-S) Destination MySQL Socket to connect through (only works if destination MySQL instance is local).
 -c CHUNKSIZE           (-c) Controls value of '--chunk-size' (default: 5000).
 -C INCREMENT           (-C) Controls how many rows per sync group (default: 200000).
 -l LOCK                (-l) Controls value of '--lock' (default: 0).
 -b /mysql/binary/path  (-b) Path to a specific MySQL binary (default: /usr/bin/mysql).
 -B /pt/table/sync      (-B) Path to a specific 'pt-table-sync' binary (default: /usr/bin/pt-table-sync).
 -e START               (-e) Controls the start value of the sync (in regards to the primary key column).
 -E END                 (-E) Controls the end point of the sync (in regards to the primary key column).
 -z                     (-z) Executes the 'pt-table-sync' with '--dry-run' for testing first.
 -Z                     (-Z) Executes the 'debug' function which prints out variable data.
```

### Test Run
- Run the script with **-z** if you want to execute the **pt-table-sync** command with **`--dry-run`** to test first.

### Debug
- Run **-Z** to print out debugging variable information and make sure you have all the necessary variables.
- Example Output:

```bash
./tablesync -d fakedb -t faketbl -pP -i 127.0.0.1 -S /var/lib/mysql/mysql.sock -o 3310 -e 0 -E 10000 -Z

Please provide the source MySQL password now:
>
Please provide the destination MySQL password now:
>
Source Database:         fakedb
Source Table:            faketbl
Source Password:         fakesourcepass
Source User:             root
Source IP:               127.0.0.1
Source Port:             3310

Destination Database:    fakedb
Destination Table:       faketbl
Destination Password:    fakedestpass
Destination User:        root
Destination Socket:      /var/lib/mysql/mysql.sock

MySQL Binary:            /usr/bin/mysql
'pt-table-sync' Binary:  /usr/bin/pt-table-sync

Chunk Size:              5000
'--lock' value:          0
Increment value:         200000
Start value:             0
End value:               10000

Debug complete. Exiting.
```

# Dbsake Compatibility

- The script is compatible with a [**dbsake** sandbox instance](https://github.com/abg/dbsake) for the source.
- You would specify the MySQL binary as the full path to the **sandbox.sh** (via **-b** option)
  - The script will update the MySQL binary path to **`/full/path/sandbox.sh mysql`**.
- Specify the path to the socket within the **data** directory to connect via the socket (which works by default).
- If you want to connect over IP+Port, there are a few configuration changes that must be made.
  - Within the **my.sandbox.cnf** you would uncomment **`port =`** and set it to an available port.
  - Also need to comment out **`skip-networking`**, then restart the sandbox instance.
  - From there, you should be able to connect over **127.0.0.1** (not **localhost**, assuming it's a local instance) and the port configured.
