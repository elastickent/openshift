FROM java:8-jre

# https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html
# https://packages.elasticsearch.org/GPG-KEY-elasticsearch
RUN apt-key adv --keyserver ha.pool.sks-keyservers.net --recv-keys 46095ACC8548582C1A2699A9D27D666CD88E42B4

RUN echo "deb http://packages.elastic.co/elasticsearch/5.x/debian stable main" > /etc/apt/sources.list.d/elasticsearch-5.x.list

RUN set -x \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends elasticsearch htop

LABEL io.openshift.min-memory 1Gi
LABEL io.openshift.tags elasticsearch
LABEL io.openshift.non-scalable true

ENV PATH /usr/share/elasticsearch/bin:$PATH

WORKDIR /usr/share/elasticsearch

RUN set -ex \
	&& for path in \
		./data \
		./logs \
		./config \
		./config/scripts \
	; do \
		mkdir -p "$path"; \
		chown -R elasticsearch:elasticsearch "$path"; \
	done

RUN /usr/share/elasticsearch/bin/elasticsearch-plugin install x-pack --batch
COPY config ./config
COPY config/ca-bundle.crt /etc/ca-bundle.crt
COPY docker-entrypoint.sh /

EXPOSE 9200 9300

RUN chmod -R og+w /usr/share/elasticsearch
#RUN mkdir -p /etc/pki/java/cacerts
#RUN chmod -R og+w /etc/pki/java/cacerts
#RUN  keytool -import -trustcacerts -keystore /etc/pki/java/cacerts -storepass changeit -noprompt -alias openshiftca -file /etc/ca-bundle.crt
RUN  keytool -import -trustcacerts -keystore /etc/ssl/certs/java/cacerts -storepass changeit -noprompt -alias openshiftca -file /etc/ca-bundle.crt
RUN  update-ca-certificates -f 
RUN /var/lib/dpkg/info/ca-certificates-java.postinst configure
USER 1000
ENV ESHOSTS=127.0.0.1
ENV ESMASTERS=3
ENV ESDATA=/usr/share/elasticsearch/data
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["elasticsearch"]
