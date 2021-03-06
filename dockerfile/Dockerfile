FROM ubuntu:18.04

ENV LANG=C.UTF-8
ENV DEBIAN_FRONTEND noninteractive

ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64
ENV ZK_HOME=/opt/zookeeper
ENV MAVEN_HOME=/opt/maven
ENV NODE_HOME=/opt/node
ENV PATH $JAVA_HOME/bin:$PATH:$ZK_HOME/bin:$MAVEN_HOME/bin:$NODE_HOME/bin

#1,安装jdk,sudo,python,vim,ping和ssh
# echo "deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse\ndeb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse\ndeb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse\ndeb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse" > /etc/apt/sources.list && \
# echo "deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse\ndeb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse\ndeb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse\ndeb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse" > /etc/apt/sources.list.d/aliyun.list && \
RUN apt-get update && \
    apt-get -y install tzdata wget openjdk-8-jdk sudo python vim iputils-ping net-tools openssh-server python-pip netcat-openbsd && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean && \
    echo "Asia/Shanghai" > /etc/timezone && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata && \
    pip install kazoo

#2,安装ZK
RUN cd /opt && \
    wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz && \
    tar -zxvf zookeeper-3.4.14.tar.gz && \
    mv zookeeper-3.4.14 zookeeper && \
    rm -rf ./zookeeper-*tar.gz && \
    mkdir -p /tmp/zookeeper && \
    rm -rf /opt/zookeeper/conf/zoo_sample.cfg

#3,安装maven
RUN cd /opt && \
    wget https://archive.apache.org/dist/maven/maven-3/3.6.2/binaries/apache-maven-3.6.2-bin.tar.gz && \
    tar -zxvf apache-maven-3.6.2-bin.tar.gz && \
    mv apache-maven-3.6.2 maven && \
    rm -rf ./apache-maven-*tar.gz

#4,安装node
RUN cd /opt && \
    wget https://nodejs.org/dist/v8.9.4/node-v8.9.4-linux-x64.tar.gz && \
    tar -zxvf node-v8.9.4-linux-x64.tar.gz && \
    mv node-v8.9.4-linux-x64 node && \
    rm -rf ./node-v8.9.4-*tar.gz

#5,安装mysql
RUN echo "mysql-server mysql-server/root_password password root" | debconf-set-selections && \
    echo "mysql-server mysql-server/root_password_again password root" | debconf-set-selections && \
    apt-get update && \
    apt-get -y install mysql-server-5.7 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    mkdir -p /var/lib/mysql && \
    mkdir -p /var/run/mysqld && \
    mkdir -p /var/log/mysql && \
    chown -R mysql:mysql /var/lib/mysql && \
    chown -R mysql:mysql /var/run/mysqld && \
    chown -R mysql:mysql /var/log/mysql && \
    sed -i -e "$ a [client]\n\n[mysql]\n\n[mysqld]"  /etc/mysql/my.cnf && \
    sed -i -e "s/\(\[client\]\)/\1\ndefault-character-set = utf8/g" /etc/mysql/my.cnf && \
    sed -i -e "s/\(\[mysql\]\)/\1\ndefault-character-set = utf8/g" /etc/mysql/my.cnf && \
    sed -i -e "s/\(\[mysqld\]\)/\1\ninit_connect='SET NAMES utf8'\ncharacter-set-server = utf8\ncollation-server=utf8_general_ci\nbind-address = 0.0.0.0/g" /etc/mysql/my.cnf

#6,安装nginx
RUN apt-get update && \
  apt-get install -y nginx && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* && \
  echo "\ndaemon off;" >> /etc/nginx/nginx.conf && \
  chown -R www-data:www-data /var/lib/nginx

#7,下载escheduler并编译
ENV version=test-20190929-2037
ENV ds_version=1.1.0-SNAPSHOT

RUN cd /opt && \
    wget https://github.com/gary0416/incubator-dolphinscheduler/archive/${version}.tar.gz && \
    tar -zxvf ${version}.tar.gz && \
    mv incubator-dolphinscheduler-${version} easyscheduler_source && \
    rm -rf ./${version}.tar.gz && \
    cd /opt/easyscheduler_source && \
    mvn -U clean package assembly:assembly -Dmaven.test.skip=true && \
    rm -rf /root/.m2 && \
    chmod -R 777 /opt/easyscheduler_source/escheduler-ui && \
    cd /opt/easyscheduler_source/escheduler-ui && \
    rm -rf /opt/easyscheduler_source/escheduler-ui/node_modules && \
    npm install node-sass --unsafe-perm && \
    npm install && \
    npm run build && \
    npm cache clean -f && \
    mv /opt/easyscheduler_source/escheduler-ui/dist /opt/escheduler-ui && \
    mkdir -p /opt/escheduler && \
    tar -zxvf /opt/easyscheduler_source/target/escheduler-${ds_version}.tar.gz -C /opt/escheduler && \
    cp /opt/easyscheduler_source/dockerfile/conf/zookeeper/zoo.cfg /opt/zookeeper/conf && \
    rm -rf /opt/escheduler/conf && \
    mv /opt/easyscheduler_source/dockerfile/conf/escheduler/conf /opt/escheduler/conf && \
    cp /opt/easyscheduler_source/dockerfile/conf/nginx/default.conf /etc/nginx/conf.d && \
    cp /opt/easyscheduler_source/dockerfile/startup.sh /root/startup.sh && \
    rm -rf /opt/easyscheduler_source

# 修改权限和设置软连
RUN chmod +x /root/startup.sh && \
  chmod +x /opt/escheduler/script/create-escheduler.sh && \
  chmod +x /opt/zookeeper/bin/zkServer.sh && \
  chmod +x /opt/escheduler/bin/escheduler-daemon.sh && \
  rm -rf /bin/sh && \
  ln -s /bin/bash /bin/sh && \
  mkdir -p /tmp/xls

# 开放端口
EXPOSE 2181 2888 3888 3306 80 12345 8888

ENTRYPOINT ["/root/startup.sh"]
