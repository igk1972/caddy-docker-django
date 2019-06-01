FROM python:3.7-alpine

ENV PYTHONUNBUFFERED 1

RUN apk update \
  # psycopg2 dependencies
  && apk add gcc python3-dev musl-dev \
  # Postgresql dependencies
  && apk add postgresql-dev \
  # Pillow dependencies
  && apk add jpeg-dev zlib-dev freetype-dev lcms2-dev openjpeg-dev tiff-dev tk-dev tcl-dev \
  # CFFI dependencies
  && apk add libffi-dev py-cffi

RUN addgroup -S django \
    && adduser -S -G django django

ENV CADDYPATH=/var/caddy CADDY_ETC=proxy CADDY_PLUGINS=http.cache,http.cors,http.prometheus,http.ratelimit,http.realip,http.supervisor

RUN apk add ca-certificates wget tar \
  && wget --quiet -O /etc/mime.types https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types \
  && mkdir -p ${CADDY_ETC} ${CADDYPATH} \
  && wget --quiet -O - \
    --header "Accept: application/tar+gzip, application/x-gzip, application/octet-stream" \
    "https://caddyserver.com/download/linux/amd64?plugins=${CADDY_PLUGINS}&license=personal&telemetry=off" \
  | tar --no-same-owner -C /usr/bin/ -xz caddy \
  && chmod 0755 /usr/bin/caddy \
  && /usr/bin/caddy -version

CMD ["caddy", "-log", "stdout", "-agree=true"]

# Requirements are installed here to ensure they will be cached.
COPY ./requirements/production.txt /requirements_production.txt
RUN pip install --no-cache-dir -r /requirements_production.txt

COPY ["./app", "./Caddyfile", "/app/"]

RUN python /app/manage.py collectstatic --noinput \
  && chown -R django /app

WORKDIR /app

USER django

