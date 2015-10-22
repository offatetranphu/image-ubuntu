NAME =			ubuntu
VERSION =		wily
VERSION_ALIASES =	15.10
TITLE =			Ubuntu Wily
DESCRIPTION =		Ubuntu Wily (15.10)
SOURCE_URL =		https://github.com/scaleway/image-ubuntu
VENDOR_URL =		http://www.ubuntu.com

IMAGE_NAME =		Ubuntu Wily (15.10 latest)
IMAGE_BOOTSCRIPT =	latest
IMAGE_VOLUME_SIZE =	50G


##
## Image tools  (https://github.com/scaleway/image-tools)
##
all:	../docker-rules.mk
../docker-rules.mk:
	$(MAKE) -C .. docker-rules.mk
-include ../docker-rules.mk
