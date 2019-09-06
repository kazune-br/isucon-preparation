FROM ubuntu:18.04
RUN apt-get update && apt-get -y upgrade
RUN apt install -y wget vim git curl  && \
    curl -o /usr/local/src/golang.tar.gz https://dl.google.com/go/go1.11.linux-amd64.tar.gz && \
    tar xf /usr/local/src/golang.tar.gz -C /usr/local/
RUN echo 'export GOROOT=/usr/local/go' >> ~/.bashrc && \
    echo 'export GOPATH=$HOME/go' >> ~/.bashrc && \
    echo 'export PATH=$GOPATH/bin:$GOROOT/bin:$PATH' >> ~/.bashrc
RUN apt install -y mariadb-server-10.1 mariadb-client
WORKDIR /isucon8-practice
CMD ["/bin/bash"]