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

# This is specific to distribution images
# -- to fetch latest code, run 'make sync-image-tools'
IMAGE_TOOLS_FLAVORS =	common,docker-based,upstart,feature-motd
IMAGE_TOOLS_CHECKOUT =	e62f185d21b86a8c09a9aef717c589d43a317da4


##
## Image tools  (https://github.com/scaleway/image-tools)
##
all:	../docker-rules.mk
../docker-rules.mk:
	$(MAKE) -C .. docker-rules.mk
-include ../docker-rules.mk
