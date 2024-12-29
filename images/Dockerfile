FROM ubuntu:latest
ARG TARGETPLATFORM
RUN apt-get -y update
RUN apt-get -y install ca-certificates curl uidmap iproute2 iptables slirp4netns git

ENV DOCKER_VERSION 27.4.1
RUN set -eux; \
	\
	case "$TARGETPLATFORM" in \
		'linux/amd64') \
			url='https://download.docker.com/linux/static/stable/x86_64/docker-27.4.1.tgz'; \
			;; \
		'linux/arm64') \
			url='https://download.docker.com/linux/static/stable/aarch64/docker-27.4.1.tgz'; \
			;; \
		*) echo >&2 "error: unsupported 'docker.tgz' architecture ($TARGETPLATFORM)"; exit 1 ;; \
	esac; \
	\
	curl "$url" -o 'docker.tgz'; \
	\
	tar --extract \
		--file docker.tgz \
		--strip-components 1 \
		--directory /usr/local/bin/ \
		--no-same-owner \
	; \
	rm docker.tgz; \
	\
	dockerd --version; \
	containerd --version; \
	ctr --version; \
	runc --version

RUN set -eux; \
	\
	case "$TARGETPLATFORM" in \
		'linux/amd64') \
			url='https://download.docker.com/linux/static/stable/x86_64/docker-rootless-extras-27.4.1.tgz'; \
			;; \
		'linux/arm64') \
			url='https://download.docker.com/linux/static/stable/aarch64/docker-rootless-extras-27.4.1.tgz'; \
			;; \
		*) echo >&2 "error: unsupported 'rootless.tgz' architecture ($TARGETPLATFORM)"; exit 1 ;; \
	esac; \
	\
	curl  "$url" -o 'rootless.tgz'; \
	\
	tar --extract \
		--file rootless.tgz \
		--strip-components 1 \
		--directory /usr/local/bin/ \
		'docker-rootless-extras/rootlesskit' \
		'docker-rootless-extras/rootlesskit-docker-proxy' \
	; \
	rm rootless.tgz; \
	\
	rootlesskit --version

ENV DOCKER_BUILDX_VERSION 0.19.3
RUN set -eux; \
	\
	case "$TARGETPLATFORM" in \
		'linux/amd64') \
			url='https://github.com/docker/buildx/releases/download/v0.19.3/buildx-v0.19.3.linux-amd64'; \
			sha256='32ed111e941e385c2fb8261eba06a4056915718fd606f8278834ac1931d261a2'; \
			;; \
		'linux/arm64') \
			url='https://github.com/docker/buildx/releases/download/v0.19.3/buildx-v0.19.3.linux-arm64'; \
			sha256='138b587399b27bb61945a36d67866177b85dea1155101a2be63c7ab715f18a2e'; \
			;; \
		*) echo >&2 "warning: unsupported 'docker-buildx' architecture ($TARGETPLATFORM); skipping"; exit 0 ;; \
	esac; \
	\
	curl "$url" -L -o 'docker-buildx' ; \
	echo "$sha256 *docker-buildx" | sha256sum -c; \
	\
	plugin='/usr/local/libexec/docker/cli-plugins/docker-buildx'; \
	mkdir -p "$(dirname "$plugin")"; \
	mv -vT 'docker-buildx' "$plugin"; \
	chmod +x "$plugin"; \
	\
	docker buildx version

ENV DOCKER_COMPOSE_VERSION 2.32.1
RUN set -eux; \
	\
	case "$TARGETPLATFORM" in \
		'linux/amd64') \
			url='https://github.com/docker/compose/releases/download/v2.32.1/docker-compose-linux-x86_64'; \
			sha256='ec81c40f138db0ca3aee71c2fffb0075636bea5a02109c75177f66e1b8f568b9'; \
			;; \
		'linux/arm64') \
			url='https://github.com/docker/compose/releases/download/v2.32.1/docker-compose-linux-aarch64'; \
			sha256='7aa6406406be13c075e92cfbf322470318a9ad7a58c93a9fb3a215dc14aed8bd'; \
			;; \
		*) echo >&2 "warning: unsupported 'docker-compose' architecture ($TARGETPLATFORM); skipping"; exit 0 ;; \
	esac; \
	\
	curl "$url" -L -o 'docker-compose'; \
	echo "$sha256 *"'docker-compose' | sha256sum -c -; \
	\
	plugin='/usr/local/libexec/docker/cli-plugins/docker-compose'; \
	mkdir -p "$(dirname "$plugin")"; \
	mv -vT 'docker-compose' "$plugin"; \
	chmod +x "$plugin"; \
	\
	docker compose version

COPY modprobe.sh /usr/local/bin/modprobe
COPY dockerd-entrypoint.sh /usr/local/bin/

RUN set -eux; \
	groupmod -n rootless ubuntu; \
	usermod -d /home/rootless -m -l rootless ubuntu; \
	groupadd docker; \
	usermod -aG docker rootless
RUN set -eux; \
	echo 'rootless:100000:65536' >> /etc/subuid; \
	echo 'rootless:100000:65536' >> /etc/subgid

RUN set -eux; \
	mkdir -p /run/user; \ 
	chmod 1777 /run/user
RUN set -eux; \
	mkdir -p /certs/client; \ 
	chmod 1777 /certs /certs/client

USER rootless
RUN set -eux; \
	mkdir -p /home/rootless/.local/share/docker
VOLUME /home/rootless/.local/share/docker
RUN set -eux; \
	mkdir -p /home/rootless/.config/docker
COPY daemon.json /home/rootless/.config/docker/

ENV XDG_RUNTIME_DIR=/run/user/1000
ENV DOCKER_HOST="unix://$XDG_RUNTIME_DIR/docker.sock"
ENV DOCKER_TLS_CERTDIR=/certs
ENTRYPOINT ["dockerd-entrypoint.sh"]
CMD []