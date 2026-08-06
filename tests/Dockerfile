FROM alpine:3.24 AS alpine-fixtures

RUN apk add --no-cache abuild alpine-sdk apk-tools-static openssl
COPY APKBUILD.in build-alpine-fixtures /fixture-source/
RUN /fixture-source/build-alpine-fixtures /alpine-fixtures

FROM debian:trixie-slim

RUN apt-get update \
	&& DEBIAN_FRONTEND=noninteractive apt-get install \
	    --no-install-recommends -y \
	    apt-utils ca-certificates createrepo-c curl dpkg-dev git gnupg gpgv jq \
	    openssl rpm \
	    shellcheck yamllint \
	&& rm -rf /var/lib/apt/lists/*

COPY --from=alpine-fixtures /sbin/apk.static /usr/local/bin/apk
COPY --from=alpine-fixtures /alpine-fixtures /alpine-fixtures
