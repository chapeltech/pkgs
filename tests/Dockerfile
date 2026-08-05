FROM debian:trixie-slim

RUN apt-get update \
	&& DEBIAN_FRONTEND=noninteractive apt-get install \
	    --no-install-recommends -y \
	    apt-utils ca-certificates createrepo-c curl dpkg-dev git gnupg jq rpm \
	    shellcheck yamllint \
	&& rm -rf /var/lib/apt/lists/*
