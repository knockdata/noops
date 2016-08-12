FROM java:8 

RUN apt-get update \
  && apt-get install -y curl \
  && curl -sL http://d3kbcqa49mib13.cloudfront.net/spark-2.0.0-bin-hadoop2.7.tgz \
   | gunzip \
   | tar x -C /usr/ \
  && mv /usr/spark* /usr/spark \
  && apt-get purge -y --force-yes curl \
  && apt-get clean autoclean \
  && apt-get autoremove -y --force-yes \
  && rm -rf /tmp/* \
  && rm -rf ~/.m2 ~/.npm ~/.cache \
  && rm -rf /var/lib/{apt,dpkg,cache,log}/

WORKDIR /usr/spark
# CMD ["sbin/start-master.sh"]
CMD ["bin/spark-class", "org.apache.spark.deploy.master.Master"]
