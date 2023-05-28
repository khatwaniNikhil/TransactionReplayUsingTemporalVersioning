# Mariab - Temporal Versioning 
https://mariadb.com/kb/en/system-versioned-tables/

# Prerequisite
1. Sanity Mismatch finder Stored proc to check current snapshot of table of interest and other tables and report/flag any mismatch.
2. Tx Id will be passed as input

# Steps
1. Spinup a new mariadb server and attach it as slave to existing prod. db cluster.
2. ONE TIME - Once SLAVE HAS SYNCED TO MASTER
    1. STOP SLAVE;
    2. Alter db table to be Transaction-Precise versioned:
        alter table table_to_be_versioned 
        add column start_trxid BIGINT UNSIGNED GENERATED ALWAYS AS ROW START,
        add column end_trxid BIGINT UNSIGNED GENERATED ALWAYS AS ROW END,
        add column PERIOD FOR SYSTEM_TIME(start_trxid, end_trxid),
        add SYSTEM VERSIONING;
    3. Create table to maintain counter of transaction last processed
        1.  create table transaction_entry (start_trxid int(10) unsigned DEFAULT NULL);
    2.  insert into transaction_entry select max(start_trxid) from table_to_be_versioned;
    3.  START SLAVE;
