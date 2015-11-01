## -*- docker-image-name: "scaleway/distrib-ubuntu:precise" -*-
FROM armbuild/ubuntu-debootstrap:precise
MAINTAINER Scaleway <opensource@scaleway.com> (@scaleway)


# Environment
ENV DEBIAN_FRONTEND noninteractive
# Useful for image inheritance
ENV SCW_BASE_IMAGE scaleway/ubuntu:precise


# ./patches/usr/local/bin/ is where the builder downloads qemu-*-static binaries
ADD ./patches/usr/local/bin/ /usr/local/bin/


# ./patches/etc/apt/ contains source.list + apt.conf tuning
ADD ./patches/etc/apt/ /etc/apt/


# Patch rootfs for docker-based builds
RUN apt-get -y -q update \
 && apt-get -y -qq install bash curl lsb-release tar \
 && apt-get -y clean \
 && curl -Lq http://j.mp/scw-skeleton | FLAVORS=common,docker-based,upstart,feature-motd bash -e \
 && /usr/local/sbin/builder-enter


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
      vim \
      wget \
      whiptail \
    && apt-get clean


# update-motd.d
RUN rm -f /etc/update-motd.d/10-help-text /etc/update-motd.d/00-header


# remove root password, it will be created by the initrd
RUN passwd -d root

# Patch rootfs
ADD ./patches/etc/ /etc/
ADD ./patches/usr/ /usr/


# Configure locales
RUN locale-gen en_US.UTF-8 && \
    locale-gen fr_FR.UTF-8 && \
    dpkg-reconfigure locales


# Fix permissions
RUN chown root:syslog /var/log


# Clean rootfs from image-builder
RUN /usr/local/sbin/builder-leave
