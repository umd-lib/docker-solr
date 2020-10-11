
FROM solr:5

LABEL maintainer="UMD Libraries"
LABEL repository="https://github.com/umd-lib/docker-solr"

ARG SOLR_VERSION="4.10.4"

USER root

ENV SOLR_USER="solr" \
    SOLR_GID="8983" \
    SOLR_ARCHIVE_URL="https://archive.apache.org/dist/lucene/solr/$SOLR_VERSION/solr-$SOLR_VERSION.tgz" \
    PATH="/opt/solr/bin:/opt/docker-solr/scripts:$PATH"

RUN set -ex; \
  MAX_REDIRECTS=1; \
  echo "downloading $SOLR_ARCHIVE_URL"; \
  if wget -t 10 --max-redirect $MAX_REDIRECTS --retry-connrefused -nv "$SOLR_ARCHIVE_URL" -O "/opt/solr-$SOLR_VERSION.tgz"; then break; else rm -f "/opt/solr-$SOLR_VERSION.tgz"; fi; \
  if [ ! -f "/opt/solr-$SOLR_VERSION.tgz" ]; then echo "failed all download attempts for solr-$SOLR_VERSION.tgz"; exit 1; fi; \
  tar -C /opt --extract --file "/opt/solr-$SOLR_VERSION.tgz"; \
  rm -rf /opt/solr; \
  mv "/opt/solr-$SOLR_VERSION" /opt/solr; \
  rm "/opt/solr-$SOLR_VERSION.tgz"*; \
  rm -Rf /opt/solr/docs/ /opt/solr/dist/{solr-core-$SOLR_VERSION.jar,solr-solrj-$SOLR_VERSION.jar,solrj-lib,solr-test-framework-$SOLR_VERSION.jar,test-framework}; \
  mkdir -p /opt/solr/server/solr/lib /docker-entrypoint-initdb.d /opt/docker-solr; \
  mkdir -p /opt/solr/server/solr/mycores /opt/solr/server/logs /opt/mysolrhome; \
  sed -i -e "s/\"\$(whoami)\" == \"root\"/\$(id -u) == 0/" /opt/solr/bin/solr; \
  sed -i -e 's/lsof -PniTCP:/lsof -t -PniTCP:/' /opt/solr/bin/solr; \
  if [ -f "/opt/solr/contrib/prometheus-exporter/bin/solr-exporter" ]; then chmod 0755 "/opt/solr/contrib/prometheus-exporter/bin/solr-exporter"; fi; \
  chown -R "$SOLR_USER:$SOLR_GROUP" /opt/solr /docker-entrypoint-initdb.d /opt/docker-solr; \
  chown -R "$SOLR_USER:$SOLR_GROUP" /opt/mysolrhome; \
  cp /opt/solr/example/solr/solr.xml /opt/solr/server/solr/

RUN chmod -R 777 /tmp/

COPY --chown=solr:solr scripts /opt/docker-solr/scripts

EXPOSE 8983
WORKDIR /opt/solr/server/
USER $SOLR_USER

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["solr-foreground"]
