## -*- docker-image-name: "scaleway/ubuntu:precise" -*-
FROM multiarch/ubuntu-debootstrap:amd64-precise
# following 'FROM' lines are used dynamically thanks do the image-builder
# which dynamically update the Dockerfile if needed.
#FROM multiarch/ubuntu-debootstrap:armhf-precise	# arch=armv7l
#FROM multiarch/ubuntu-debootstrap:i386-precise		# arch=i386
#FROM multiarch/ubuntu-debootstrap:mips-precise		# arch=mips


MAINTAINER Scaleway <opensource@scaleway.com> (@scaleway)


# Environment
ENV DEBIAN_FRONTEND=noninteractive \
    SCW_BASE_IMAGE=scaleway/ubuntu:precise


# Configure aptitude
# Note that, `${ARCH}` is set in the multiarch/ubuntu-debootstrap image
COPY ./overlay-${ARCH}/etc/apt/ /etc/apt/


# Adding and calling builder-enter
COPY ./overlay-image-tools/usr/local/sbin/scw-builder-enter /usr/local/sbin/
RUN /usr/local/sbin/scw-builder-enter


# Install packages
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
      tcpdump \
      tmux \
      uuid-runtime \
      vim \
      wget \
      whiptail \
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


# Fix permissions
RUN chown root:syslog /var/log \
 && chmod 755 /etc/default


# Clean rootfs from image-builder
RUN /usr/local/sbin/scw-builder-leave
