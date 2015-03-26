## -*- docker-image-name: "armbuild/ocs-distrib-ubuntu:precise" -*-
FROM armbuild/ubuntu-debootstrap:precise
MAINTAINER Online Labs <opensource@ocs.online.net> (@online_en)


# Environment
ENV DEBIAN_FRONTEND noninteractive
# Useful for image inheritance
ENV OCS_BASE_IMAGE armbuild/ocs-ubuntu:precise


# Patch rootfs for docker-based builds
RUN apt-get -y -q update \
 && apt-get -y -qq install bash curl lsb-release tar \
 && apt-get -y clean \
 && curl -Lq http://j.mp/ocs-scripts | FLAVORS=common,docker-based bash -e \
 && /usr/local/sbin/builder-enter


# Install packages
ADD ./patches/etc/apt/sources.list /etc/apt/sources.list
RUN apt-get -q update && \
    apt-get -y -qq upgrade && \
    apt-get -y -qq install \
	bash-completion \
	bc \
	ca-certificates \
	cron \
	curl \
	dbus \
	dstat \
	ethstatus \
	file \
	fio \
	htop \
	ioping \
	iotop \
	iperf \
	iptables \
	iputils-ping \
	isc-dhcp-client \
	less \
	locate \
	lsb-release \
	lsof \
	make \
	man-db \
	mg \
	mosh \
	nano \
	nbd-client \
	net-tools \
	netcat \
	ntp \
	ntpdate \
	python-apt \
	python-yaml \
	rsync \
	rsyslog \
	screen \
	shunit2 \
	socat \
	ssh \
	sudo \
	sysstat \
	tcpdump \
	tmux \
	wget \
	whiptail \
	xnbd-client \
    && apt-get clean


# Patch rootfs
# - Tweaks rootfs so it matches Online Labs infrastructure
RUN rm -f /etc/update-motd.d/10-help-text /etc/update-motd.d/00-header
RUN curl -Lq http://j.mp/ocs-scripts | FLAVORS=upstart bash -e
ADD ./patches/etc/ /etc/


# Configure locales
RUN locale-gen en_US.UTF-8 && \
    locale-gen fr_FR.UTF-8 && \
    dpkg-reconfigure locales


# Clean rootfs from image-builder
RUN /usr/local/sbin/builder-leave