FROM python:3.9.0-alpine
LABEL maintainer="Victor R. Santos <victor-rds@users.noreply.github.com>"

ARG PUID=373
ARG PGID=373
ARG ETE_VERSION=master
ARG ETE_URL="https://github.com/etesync/server/archive/${ETE_VERSION}.tar.gz"

ENV BASE_DIR /etebase

ENV DATA_DIR /data
ENV MEDIA_ROOT ${DATA_DIR}/media
ENV ETEBASE_EASY_CONFIG_PATH ${DATA_DIR}/etebase-server.ini
ENV SECRET_FILE ${DATA_DIR}/secret.txt

ENV STATIC_ROOT /srv/etebase/static

ENV SERVER="http"
ENV PORT 3735

WORKDIR ${BASE_DIR}

# OS packages and deps installation
# hadolint ignore=DL3008,DL3018
RUN set -eux; \
    apk add --no-cache openssl libressl mailcap libpq libffi bash; \
    apk add --no-cache --virtual .build-deps coreutils gcc make libffi-dev linux-headers libc-dev libressl-dev postgresql-dev wget; \
    wget -nv --no-hsts -O /tmp/etebase.tar.gz "${ETE_URL}"; \
    tar --strip-components=1 -vzxf /tmp/etebase.tar.gz; \
    pip install --no-cache-dir --progress-bar off -r ${BASE_DIR}/requirements.txt uwsgi daphne 'Twisted[tls,http2]'; \
    apk del .build-deps; \
    rm -r /tmp/etebase.tar.gz ${BASE_DIR}/example-configs ${BASE_DIR}/etebase-server.ini.example; \
    rm -r /root/.cache; \
    chown -R ${PUID}:${PGID} "${BASE_DIR}"

# Etebase directories and user setup 
RUN set -eux; \
    addgroup --system --gid ${PGID} etebase; \
    adduser --system --no-create-home --disabled-password \
    --gecos '' --home ${BASE_DIR} --ingroup etebase --uid ${PUID} etebase; \
    mkdir -p ${MEDIA_ROOT} ${STATIC_ROOT}; \
    chown -R ${PUID}:${PGID} ${DATA_DIR} ${STATIC_ROOT}; \
    chmod -R 750 ${DATA_DIR}; \
    export DJANGO_STATIC_ROOT=${STATIC_ROOT}; \
    ${BASE_DIR}/manage.py collectstatic --no-input; \
    rm ${BASE_DIR}/secret.txt

COPY --chown=${PUID}:${PGID} context /

USER etebase:etebase

EXPOSE ${PORT}

VOLUME /data

ENTRYPOINT ["/entrypoint.sh"]