FROM debian:bookworm AS runner
LABEL maintainer="ruscalworld <me@ruscal.world>"

# Legacy arguments: with only these set, the image is built exactly as before,
# from pmmp/PHP-Binaries (archived upstream, but its releases are still served).
ARG PMMP_MAJOR_VERSION=5
ARG PHP_VERSION='8.1'

# Where to take the PHP runtime from. Override these to build an image for a
# PocketMine-MP fork that ships its own PHP binaries (e.g. AmberPM with ext-rtc).
# An empty value falls back to the pmmp/PHP-Binaries default.
ARG PHP_BINARIES_REPO='pmmp/PHP-Binaries'
ARG PHP_BINARIES_TAG=''
ARG PHP_BINARIES_FILE=''
# Optional checksum file published next to the archive (e.g. SHA256SUMS.txt).
# When set, the archive is verified against it and the build fails on mismatch.
ARG PHP_BINARIES_CHECKSUMS=''
# Space-separated PHP extensions that must be loaded, checked at build time
# so a broken runtime never gets pushed (e.g. "rtc pmmpthread").
ARG PHP_REQUIRED_EXTENSIONS=''

RUN set -eu; \
	apt update && apt install -y curl tar iproute2 ca-certificates libstdc++6 && rm -rf /var/lib/apt/lists/*; \
	TAG="${PHP_BINARIES_TAG:-pm${PMMP_MAJOR_VERSION}-php-${PHP_VERSION}-latest}"; \
	FILE="${PHP_BINARIES_FILE:-PHP-${PHP_VERSION}-Linux-x86_64-PM${PMMP_MAJOR_VERSION}.tar.gz}"; \
	BASE="https://github.com/${PHP_BINARIES_REPO}/releases/download/${TAG}"; \
	mkdir -p /tmp/php && cd /tmp/php; \
	echo "Downloading ${BASE}/${FILE}"; \
	curl -fsSL --retry 3 -o "${FILE}" "${BASE}/${FILE}"; \
	if [ -n "${PHP_BINARIES_CHECKSUMS}" ]; then \
		curl -fsSL --retry 3 -o checksums.txt "${BASE}/${PHP_BINARIES_CHECKSUMS}"; \
		tr -d '\r' < checksums.txt | grep -E "[[:space:]]\*?${FILE}\$" | sha256sum -c -; \
	fi; \
	tar -xzf "${FILE}"; \
	mkdir /usr/php && mv /tmp/php/bin/php7/* /usr/php; \
	cd / && rm -rf /tmp/php; \
	EXT_DIR="$(find /usr/php/lib/php/extensions -maxdepth 1 -type d -name '*debug-zts*' | head -n1)"; \
	if grep -q '^extension_dir' /usr/php/bin/php.ini; then \
		sed -i "s{^extension_dir=.*{extension_dir=\"${EXT_DIR}\"{" /usr/php/bin/php.ini; \
	else \
		echo "extension_dir=\"${EXT_DIR}\"" >> /usr/php/bin/php.ini; \
	fi; \
	ln -s /usr/php/bin/php /usr/bin/php; \
	php -v; \
	for ext in ${PHP_REQUIRED_EXTENSIONS}; do \
		php -r "exit(extension_loaded('${ext}') ? 0 : 1);" || { echo "Required PHP extension '${ext}' is not loaded" >&2; exit 1; }; \
	done

USER container
ENV USER=container HOME=/home/container TERM=xterm
ENV DEBIAN_FRONTEND noninteractive

WORKDIR /home/container

COPY ./entrypoint.sh /entrypoint.sh
CMD [ "/bin/bash", "/entrypoint.sh" ]
