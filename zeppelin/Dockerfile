FROM debian:jessie

ENV BRANCH  branch-0.6

ENV ZEPPELIN_PORT 8080
ENV ZEPPELIN_HOME /usr/zeppelin
ENV ZEPPELIN_CONF_DIR $ZEPPELIN_HOME/conf
ENV ZEPPELIN_HIGHCHART_VERSION 0.6.0-SNAPSHOT

# Install basic packages
RUN apt-get update \
  && apt-get install -y git curl sed xmlstarlet

# JAVA
ARG JAVA_MAJOR_VERSION=8
ARG JAVA_UPDATE_VERSION=102
ARG JAVA_BUILD_NUMBER=14
ENV JAVA_HOME /usr/jdk1.${JAVA_MAJOR_VERSION}.0_${JAVA_UPDATE_VERSION}

ENV PATH $PATH:$JAVA_HOME/bin
RUN curl -sL --retry 3 --insecure \
  --header "Cookie: oraclelicense=accept-securebackup-cookie;" \
  "http://download.oracle.com/otn-pub/java/jdk/${JAVA_MAJOR_VERSION}u${JAVA_UPDATE_VERSION}-b${JAVA_BUILD_NUMBER}/server-jre-${JAVA_MAJOR_VERSION}u${JAVA_UPDATE_VERSION}-linux-x64.tar.gz" \
  | gunzip \
  | tar x -C /usr/ \
  && ln -s $JAVA_HOME /usr/java \
  && rm -rf $JAVA_HOME/man

# Install Maven 3.3.0
RUN curl -sL http://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz \
   | gunzip \
   | tar x -C /tmp/

RUN git clone https://github.com/knockdata/zeppelin-highcharts.git /tmp/zeppelin-highcharts \
  && cd /tmp/zeppelin-highcharts \
  && /tmp/apache-maven-3.3.9/bin/mvn package install -DskipTests -Dgpg.skip

# Clone zeppelin
RUN git clone https://github.com/apache/zeppelin.git /tmp/zeppelin \
  && cd /tmp/zeppelin \
  && git checkout $BRANCH

# ADD Highchart to zeppelin-web
RUN sed -i 's/"angular":/"highcharts": "^4.2.6","angular":/' /tmp/zeppelin/zeppelin-web/bower.json

# ADD zeppelin-highcharts to spark-dependencies
RUN xmlstarlet ed -s /_:project/_:dependencies -t elem -n dependency -v zeppelin-highcharts /tmp/zeppelin/spark-dependencies/pom.xml > pom2.xml \
  && sed -i "s:zeppelin-highcharts:<groupId>com.knockdata</groupId><artifactId>zeppelin-highcharts</artifactId><version>$ZEPPELIN_HIGHCHART_VERSION</version>:" pom2.xml \
  && mv -f pom2.xml /tmp/zeppelin/spark-dependencies/pom.xml

# Compile Zeppelin
RUN cd /tmp/zeppelin \
  && /tmp/apache-maven-3.3.9/bin/mvn package -Pbuild-distr -DskipTests

# Distribute zeppelin
RUN tar xvf /tmp/zeppelin/zeppelin-distribution/target/zeppelin*.tar.gz -C /usr/ \
  && mv /usr/zeppelin* $ZEPPELIN_HOME \
  && mkdir -p $ZEPPELIN_HOME/logs \
  && mkdir -p $ZEPPELIN_HOME/run

# Clean Env
RUN apt-get purge -y --auto-remove git curl xmlstarlet \
  && rm -rf /tmp/* \
  && rm -rf ~/.m2

WORKDIR $ZEPPELIN_HOME
CMD ["bin/zeppelin.sh"]
