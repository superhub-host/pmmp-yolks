FROM debian:bullseye AS runner
LABEL maintainer="ruscalworld <me@ruscal.world>"

ARG PMMP_MAJOR_VERSION=5
ARG PHP_VERSION='php-8.1-latest'

RUN apt update && apt install -y curl tar iproute2 && \
	mkdir /tmp/php && \
	curl --location --progress-bar https://github.com/pmmp/PHP-Binaries/releases/download/${PHP_VERSION}/PHP-Linux-x86_64-PM${PMMP_MAJOR_VERSION}.tar.gz | tar -xvz -C /tmp/php && \
	mkdir /usr/php && \
	mv /tmp/php/bin/php7/* /usr/php && \
	grep -q '^extension_dir' /usr/php/bin/php.ini && \
	sed -ibak "s{^extension_dir=.*{extension_dir=\"$(find /usr/php -name *debug-zts*)\"{" /usr/php/bin/php.ini || echo "extension_dir=\"$(find /usr/php -name *debug-zts*)\"" >> /usr/php/bin/php.ini && \
	ln -s /usr/php/bin/php /usr/bin/php

USER container
ENV USER=container HOME=/home/container TERM=xterm
ENV DEBIAN_FRONTEND noninteractive

WORKDIR /home/container

COPY ./entrypoint.sh /entrypoint.sh
CMD [ "/bin/bash", "/entrypoint.sh" ]
