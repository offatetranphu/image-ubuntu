NAME =			ubuntu
VERSION =		precise
VERSION_ALIASES =	12.04.5 12.04
TITLE =			Ubuntu Precise
DESCRIPTION =		Ubuntu Precise (12.04)
SOURCE_URL =		https://github.com/scaleway/image-ubuntu
VENDOR_URL =		http://www.ubuntu.com
DEFAULT_IMAGE_ARCH =	x86_64

IMAGE_NAME =		Ubuntu Precise (12.04)
IMAGE_BOOTSCRIPT =	stable
IMAGE_VOLUME_SIZE =	50G

# This is specific to distribution images
# -- to fetch latest code, run 'make sync-image-tools'
IMAGE_TOOLS_FLAVORS =	common,docker-based,upstart,feature-motd
IMAGE_TOOLS_CHECKOUT =	764a58e59811d5f6217a3f71a4e009d6ec3e5138


##
## Image tools  (https://github.com/scaleway/image-tools)
##
all:	../docker-rules.mk
../docker-rules.mk:
	$(MAKE) -C .. docker-rules.mk
-include ../docker-rules.mk
