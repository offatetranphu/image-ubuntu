NAME =			ubuntu
VERSION =		xenial
VERSION_ALIASES =	16.04 latest
TITLE =			Ubuntu Xenial
DESCRIPTION =		Ubuntu Xenial (16.04 LTS)
SOURCE_URL =		https://github.com/scaleway/image-ubuntu
VENDOR_URL =		http://www.ubuntu.com
DEFAULT_IMAGE_ARCH =	x86_64

IMAGE_NAME =		Ubuntu Xenial (16.04 latest)
IMAGE_BOOTSCRIPT =	latest
IMAGE_VOLUME_SIZE =	50G


# This is specific to distribution images
# -- to fetch latest code, run 'make sync-image-tools'
IMAGE_TOOLS_FLAVORS =	common,docker-based,systemd,feature-motd
IMAGE_TOOLS_CHECKOUT =	93bbd407c6e9d3ac2fb418c48ab1a68d3e702a9d


##
## Image tools  (https://github.com/scaleway/image-tools)
##
all:	../docker-rules.mk
../docker-rules.mk:
	$(MAKE) -C .. docker-rules.mk
-include ../docker-rules.mk
