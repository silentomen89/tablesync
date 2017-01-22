# Quick Description

- This script utilizes Percona's **pt-table-sync** to sync data between databases/tables.
- The script will break up a table and sync it in increments to help avoid **deadlocks**.
- The script detects the primary key on the table given, determines the start/end points, and then syncs it in sections.
  - By default it'll do 200000 entries at a time (can be changed via **-C value**).
- I wrote this while live migrating a client with over 2 billion rows to a new database cluster.
- I kept getting deadlocks because the replication threads were updating the tables and killing the **pt-table-sync**.
- I worked around this by breaking up the tables into sections and syncing it that way, which is what this script does.

# Much better docs to come

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
- Script is compatible with **dbsake** sandbox instance for the source.
  - Just specify the full path to the **sandbox.sh** and the script will update it to **`/full/path/sandbox.sh mysql`**.
  - It's not compatible with socket connections yet, so you need to get the Sandbox instance listening on a port.
    - Uncomment **`port =`** and **`skip-networking`** should suffice.
  - You also have to specify **127.0.0.1** instead of **localhost** if it's a local **dbsake** sandbox instance.
- Run the script with **-z** if you want to execute the **pt-table-sync** command with **`--dry-run`** to test first.
- Run **-Z** to print out debugging variable information and make sure you have all the necessary variables.
