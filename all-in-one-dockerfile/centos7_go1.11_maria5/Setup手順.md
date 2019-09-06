# Basic Setup
## (0) dockerfileから起動
```
$ docker build . -t isucon
$ docker run --privileged -d --name mycent mycent /sbin/init
$ docker exec -it isucon bash
```

## (1) docker-composeで起動
```
$ docker-compose up -d
$ docker exec -it isucon bash
```

## (2) mariaDBの設定と起動

```bash
$ vim /etc/my.cnf.d/server.cnf
[server]
character-set-server = utf8

[mysqld]
character-set-server = utf8mb4
plugin-load = handlersocket.so

[client]
default-character-set = utf8mb4

$ systemctl enable mariadb
$ systemctl start mariadb
```

# Setup for ISUCON8
## (1) DBの初期化
```bash
$ ./db/init-user.sh
$ ./db/init.sh
```
## (2) make, gb, vendorのインストールとechoの起動
```bash
$ cd webapp/go
$ yum install make
$ go get github.com/constabulary/gb/...
$ go get github.com/constabulary/gb/cmd/gb-vendor
$ make deps
$ ./run_local.sh
```

# etc
## (1) run_local.sh
```bash
#!/bin/bash
set -ue

export DB_DATABASE=torb
export DB_HOST=localhost
export DB_PORT=3306
export DB_USER=isucon
export DB_PASS=isucon

GOPATH=`pwd`:`pwd`/vendor go run torb
```

## (2) init-user.sh
```bash
#!/bin/bash

cat <<'EOF' | mysql -uroot
CREATE USER 'isucon'@'%' IDENTIFIED BY 'isucon';
GRANT ALL ON torb.* TO 'isucon'@'%';
CREATE USER 'isucon'@'localhost' IDENTIFIED BY 'isucon';
GRANT ALL ON torb.* TO 'isucon'@'localhost';
EOF
```

## (3) init.sh
```bash
#!/bin/bash

ROOT_DIR=$(cd $(dirname $0)/..; pwd)
DB_DIR="$ROOT_DIR/db"
BENCH_DIR="$ROOT_DIR/bench"

export MYSQL_PWD=isucon

mysql -uisucon -e "DROP DATABASE IF EXISTS torb; CREATE DATABASE torb;"
mysql -uisucon torb < "$DB_DIR/schema.sql"

if [ ! -f "$DB_DIR/isucon8q-initial-dataset.sql.gz" ]; then
  echo "Run the following command beforehand." 1>&2
  echo "$ ( cd \"$BENCH_DIR\" && bin/gen-initial-dataset )" 1>&2
  exit 1
fi

mysql -uisucon torb -e 'ALTER TABLE reservations DROP KEY event_id_and_sheet_id_idx'
gzip -dc "$DB_DIR/isucon8q-initial-dataset.sql.gz" | mysql -uisucon torb
mysql -uisucon torb -e 'ALTER TABLE reservations ADD KEY event_id_and_sheet_id_idx (event_id, sheet_id)'
```
