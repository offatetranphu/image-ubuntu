NAME =			ubuntu
VERSION =		trusty
VERSION_ALIASES =	14.04.1 14.04 lts
TITLE =			Ubuntu Trusty
DESCRIPTION =		Ubuntu Trusty (14.04 LTS)
SOURCE_URL =		https://github.com/scaleway/image-ubuntu
VENDOR_URL =		http://www.ubuntu.com
DEFAULT_IMAGE_ARCH =	x86_64

IMAGE_NAME =		Ubuntu Trusty (14.04 LTS)
IMAGE_BOOTSCRIPT =	stable
IMAGE_VOLUME_SIZE =	50G


##
## Image tools  (https://github.com/scaleway/image-tools)
##
all:	../docker-rules.mk
../docker-rules.mk:
	$(MAKE) -C .. docker-rules.mk
-include ../docker-rules.mk
