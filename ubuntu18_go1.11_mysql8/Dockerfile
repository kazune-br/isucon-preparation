FROM ubuntu:18.04

# 必要なパッケージのインストール
RUN apt-get update && apt-get -y upgrade
RUN apt install -y wget vim git curl lsb-release gnupg

# goのインストールと解答
RUN curl -o /usr/local/src/golang.tar.gz https://dl.google.com/go/go1.11.linux-amd64.tar.gz && \
    tar xf /usr/local/src/golang.tar.gz -C /usr/local/

# 環境変数の設定(docker runしたときに読み込まれる)
RUN echo 'export GOROOT=/usr/local/go' >> ~/.bashrc && \
    echo 'export GOPATH=$HOME/go' >> ~/.bashrc && \
    echo 'export PATH=$GOPATH/bin:$GOROOT/bin:$PATH' >> ~/.bashrc

# gui用の設定をコマンドで回避するため
COPY ./myslq_installation_command.txt /tmp/myslq_installation_command.txt

# MySQLのダウンロード
RUN wget https://dev.mysql.com/get/mysql-apt-config_0.8.13-1_all.deb && \
    dpkg -i mysql-apt-config_0.8.13-1_all.deb < /tmp/myslq_installation_command.txt

# 設定ファイルが実行され、aptの内容が更新されたため、改めてupdateの実施
RUN apt -y update

# mysql-clientのインストールの実行
ENV DEBIAN_FRONTEND noninteractive
RUN apt install -y mysql-client mysql-server

WORKDIR /isucon

CMD ["/bin/bash"]