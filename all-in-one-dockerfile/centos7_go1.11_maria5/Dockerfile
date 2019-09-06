FROM centos:7.5.1804
RUN yum -y update
RUN curl -o /usr/local/src/golang.tar.gz https://dl.google.com/go/go1.11.linux-amd64.tar.gz && \
    tar xf /usr/local/src/golang.tar.gz -C /usr/local/
RUN echo 'export GOROOT=/usr/local/go' >> ~/.bashrc && \
    echo 'export GOPATH=$HOME/go' >> ~/.bashrc && \
    echo 'export PATH=$GOPATH/bin:$GOROOT/bin:$PATH' >> ~/.bashrc
RUN yum install -y git mariadb-client mariadb-server
WORKDIR /isucon8-practice
CMD ["/bin/bash"]