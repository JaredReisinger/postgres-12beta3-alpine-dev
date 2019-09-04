# postgres-12beta3-alpine-dev

A variant of the [postgres:12-beta3-alpine](https://github.com/docker-library/postgres/blob/d6e8fe3240b3d2c5d1a03f005360710812714163/12/alpine/Dockerfile) image, with dev tools in place.

## Usage

The expectation is that you would use this image as the base from which you would build a PostgreSQL extension:

```Dockerfile
FROM jaredreisinger/postgres-12beta3-alpine-dev:latest as builder

ENV HASHIDS_VERSION 1.2.1

RUN set -ex \
	\
	&& wget -O pg_hashids.tar.gz https://github.com/iCyberon/pg_hashids/archive/v$HASHIDS_VERSION.tar.gz \
	&& mkdir -p /usr/src/postgresql/contrib/pg_hashids \
	&& tar \
		--extract \
		--file pg_hashids.tar.gz \
		--directory /usr/src/postgresql/contrib/pg_hashids \
		--strip-components 1 \
	&& rm pg_hashids.tar.gz \
	&& cd /usr/src/postgresql/contrib/pg_hashids \
	&& make install

# copy the built output into a clean postgres image
FROM postgres:12-beta3-alpine

COPY --from=builder /usr/local/lib/postgresql/pg_hashids* /usr/local/lib/postgresql/.
COPY --from=builder /usr/local/share/postgresql/extension/pg_hashids* /usr/local/share/postgresql/extension/.
```

See [JaredReisinger/postgres-12beta3-alpine-hashids](https://github.com/JaredReisinger/postgres-12beta3-alpine-hashids) for a practical example.

## Why does this exist?

I needed to get pg_hashids into a Docker-run postgres; the most obvious way to do so is to build from a container/image with the postgres development tools, and then copy into the clean base postgres image (the one *after* they've removed the tools and build dependencies).  Rather than combine all of that into one Dockerfile, I'm making this "development" variant public so that other folks can do the same thing, if they need to.
