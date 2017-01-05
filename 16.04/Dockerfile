## -*- docker-image-name: "scaleway/ubuntu:xenial" -*-
FROM multiarch/ubuntu-debootstrap:amd64-xenial
# following 'FROM' lines are used dynamically thanks do the image-builder
# which dynamically update the Dockerfile if needed.
#FROM multiarch/ubuntu-debootstrap:armhf-xenial	# arch=armv7l
#FROM multiarch/ubuntu-debootstrap:arm64-xenial	# arch=arm64
#FROM multiarch/ubuntu-debootstrap:i386-xenial	# arch=i386
#FROM multiarch/ubuntu-debootstrap:mips-xenial	# arch=mips


MAINTAINER Scaleway <opensource@scaleway.com> (@scaleway)


# Environment
ENV DEBIAN_FRONTEND=noninteractive \
    SCW_BASE_IMAGE=scaleway/ubuntu:xenial


# Configure aptitude
# Note that, `${ARCH}` is set in the multiarch/ubuntu-debootstrap image
COPY ./overlay-${ARCH}/etc/apt/ /etc/apt/


# Adding and calling builder-enter
COPY ./overlay-image-tools/usr/local/sbin/scw-builder-enter /usr/local/sbin/
RUN /usr/local/sbin/scw-builder-enter


# Install packages
RUN apt-get -q update && \
    apt-get                                \
      -o Dpkg::Options::='--force-confold' \
      -o Dpkg::Options::='--force-confdef' \
      install --only-upgrade base-files   && \
    rm -rf /etc/update-motd.d/*.dpkg-dist && \
    apt-get -y -qq upgrade && \
    apt-get -y -qq install \
	bash \
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
	haveged \
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
	module-init-tools \
	mosh \
	nano \
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
	systemd-sysv \
	tar \
	tcpdump \
	tmux \
	udev \
	uuid-runtime \
	vim \
	wget \
	whiptail \
	unattended-upgrades \
	&& apt-get clean


# Patch rootfs
# - Tweaks rootfs so it matches Scaleway infrastructure
RUN rm -f /etc/update-motd.d/10-help-text /etc/update-motd.d/00-header
COPY ./overlay/ ./overlay-image-tools/ ./overlay-${ARCH}/ /

# remove root password, it will be created by the initrd
RUN passwd -d root

# Configure locales
RUN locale-gen en_US.UTF-8 && \
	locale-gen fr_FR.UTF-8 && \
	dpkg-reconfigure locales


# Default target
RUN systemctl set-default multi-user


# Configure Systemd
RUN systemctl enable \
    scw-fetch-ssh-keys \
	scw-sync-kernel-modules \
	scw-generate-ssh-keys \
	scw-kernel-check \
 && systemctl disable \
	kmod-static-nodes.service \
	proc-sys-fs-binfmt_misc.automount \
	systemd-modules-load.service \
	systemd-update-utmp-runlevel


# make /sbin/init a relative symlink for initrd boot
RUN rm -f /sbin/init /bin/init \
 && ln -sf ../lib/systemd/systemd /sbin/init \
 && ln -sf ../lib/systemd/systemd /bin/init


RUN rm -f /etc/machine-id /var/lib/dbus/machine-id

# Fix permissions
RUN chown root:syslog /var/log \
 && chmod 755 /etc/default

# HotFix Reboot
RUN systemctl mask networking

# Clean rootfs from image-builder
RUN /usr/local/sbin/scw-builder-leave
