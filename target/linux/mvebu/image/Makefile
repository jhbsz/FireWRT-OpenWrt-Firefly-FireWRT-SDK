#
# Copyright (C) 2012-2014 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/image.mk

TARGET_DTBS := armada-xp-db armada-370-db armada-xp-openblocks-ax3-4 armada-370-mirabox \
		armada-370-rd armada-xp-gp armada-xp-mamba

NANDBOARDS := armada-xp-mamba

LOADADDR:=0x00008000

JFFS2_BLOCKSIZE = 128k

UBIFS_OPTS = -F -m 2048 -e 124KiB -c 4096 -U
UBI_OPTS = -m 2048 -p 128KiB -s 512 -O 2048

KDIR_TMP:=$(KDIR)/tmp


UIMAGE:=$(BIN_DIR)/$(IMG_PREFIX)-uImage

define Image/Build/DTB
	cp $(KDIR)/zImage$(2) $(KDIR)/zImage$(2)-$(1);
	cat $(DTS_DIR)/$(1).dtb >> $(KDIR)/zImage$(2)-$(1);
	$(call Image/BuildKernel/MkuImage, \
		none, $(LOADADDR), $(LOADADDR), \
		$(KDIR)/zImage$(2)-$(1), $(KDIR)/uImage$(2)-$(1))
	cp $(KDIR)/uImage$(2)-$(1) $(UIMAGE)$(2)-$(1);
endef

define Image/BuildKernel
	$(foreach dtb,$(TARGET_DTBS),$(call Image/Build/DTB,$(dtb)))
 ifneq ($(CONFIG_TARGET_ROOTFS_INITRAMFS),)
	$(call Image/Build/Initramfs)
 endif
endef

define Image/Build/squashfs
	$(call prepare_generic_squashfs,$(KDIR)/root.squashfs)
	$(foreach nandboard,$(NANDBOARDS),$(call Image/Build/UbinizeImage,$(nandboard),,squashfs,$(UBI_OPTS));)
	( \
		dd if=$(KDIR)/uImage-armada-xp-mamba bs=3072k conv=sync; \
		dd if=$(KDIR)/$(IMG_PREFIX)-armada-xp-mamba-squashfs-ubinized.bin \
		bs=2048 conv=sync; \
	) > $(BIN_DIR)/$(IMG_PREFIX)-armada-xp-mamba-squashfs-factory.img
endef

define Image/Build/Initramfs
	$(foreach dtb,$(TARGET_DTBS),$(call Image/Build/DTB,$(dtb),-initramfs))
endef

define BuildSysupgrade
	$(call Image/Build/SysupgradeNAND,$(2),$(1),$(KDIR)/uImage-$(2))
endef

define Image/Build
	$(call Image/Build/$(1))
	dd if=$(KDIR)/root.$(1) of=$(BIN_DIR)/$(IMG_PREFIX)-root.$(1) bs=128k conv=sync
	$(foreach nandboard,$(NANDBOARDS),$(call BuildSysupgrade,$(1),$(nandboard));)
ifneq ($(CONFIG_TARGET_ROOTFS_INITRAMFS),)
	$(call Image/Build/Initramfs)
endif
endef

$(eval $(call BuildImage))
