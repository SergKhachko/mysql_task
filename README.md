# Description
This file contains the information regarding `mysqldump` test task

# Commands
### Start DB with `./init/*.sql` scripts to insert some data
```bash
docker-compose up -d
```

### [Optional] Use Docker to run `mysqldump`
```
docker run -it --rm -v $PWD:/workdir -w /workdir mysql:5.7 bash
```

### Install `mbuffer` to control disk I/O
```
yum -y install mbuffer
```

### Create dump files in parallel
```
mysqldump -uroot -pexample -h host.docker.internal --single-transaction --quick playground |\
    mbuffer |\
    tee \
        >(gzip > full_dump.sql.gz) \
        >(grep -v "^INSERT INTO \`log_" | gzip > minimized_dump.sql.gz) \
        >/dev/null
```
Command breakdown:
- `mysqldump`
  - with `--single-transaction` option to avoid locking tables
  - with `--quick` option, to read 1 row at a time (slows down reads)
- `mbuffer` uses memory buffer to avoid excessive I/O (also configurable with parameters)
- `tee` to split stdout (we want a single pass of `mysqldump`)
- `gzip` is for compression on the fly
- ``grep -v "^INSERT INTO \`log_"`` is to ignore inserts into `log_*` tables
- `>/dev/null` is to ignore stdout to terminal from `tee`