FROM postgres:11.6-alpine
ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd


COPY 01-create.sql /docker-entrypoint-initdb.d/
COPY 02-init.sql /docker-entrypoint-initdb.d/