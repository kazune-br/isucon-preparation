# Dockerの起動 
```
$ docker build . -t isucon_ubuntu18_mysql8
## $ docker run -it -d --name isucon_ubuntu18_mysql8 isucon_ubuntu18_mysql8 /bin/bash --login
$ docker-compose up -d
$ docker exec -it isucon_ubuntu18_mysql8 bash
```



# Setup for ISUCON8 inside Docker
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



# REFS
- [MySQL in Docker frozen at root password config](https://stackoverflow.com/questions/38356219/mysql-in-docker-frozen-at-root-password-config)
- [[MySQL][ERROR] Fatal error: Please read “Security” section of the manual to find out how to run mysqld as root!と言われた時の行動](https://qiita.com/yosida001/items/f7acb893843c550e0074)
- [mysql PPA - invalid signature](https://askubuntu.com/questions/1120363/mysql-ppa-invalid-signature)
- [docker-compose MySQL8.0 のDBコンテナを作成する](https://qiita.com/ucan-lab/items/b094dbfc12ac1cbee8cb)
- [Ubuntu Linux 18.04 LTSに、MySQL 8.0をインストールする](https://kazuhira-r.hatenablog.com/entry/20180612/1528812326)
- [MySQL 8 最新版を Ubuntu に apt インストールする](https://xn--o9j8h1c9hb5756dt0ua226amc1a.com/?p=499)
- [DockerのubuntuイメージにMySQL-Client8.0.11をインストール。](https://kuzunoha-ne.hateblo.jp/entry/2018/06/29/204039)
