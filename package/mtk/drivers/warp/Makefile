# All rights reserved.
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=warp
PKG_VERSION:=7.6.7.2

PKG_SOURCE_VERSION:=20231229-5f71ec
PKG_SOURCE:=mt79xx_$(PKG_NAME)_$(PKG_SOURCE_VERSION).tar.xz
PKG_SOURCE_URL:=https://github.com/RuijieNetworksCommunity/mt79xx-$(PKG_NAME)-collections/releases/download/mt79xx_$(PKG_NAME)_$(PKG_SOURCE_VERSION)
PKG_HASH:=e7e2c2221207f4e72ff7f839ddfa5d7fb87b1be86dee608838a94f26124e6aaf

PKG_BUILD_DIR:=$(KERNEL_BUILD_DIR)/$(PKG_NAME)
PKG_KCONFIG:= \
	WARP_VERSION \
	WARP_DBG_SUPPORT \
	WED_HW_RRO_SUPPORT \
	MTK_MEMORY_SHRINK \
	WARP_CHIPSET \
	WARP_WO_EMBEDDED_LOAD \
	WARP_MEMORY_LEAK_DBG

PKG_CONFIG_DEPENDS:=$(foreach c, $(PKG_KCONFIG),$(if $(CONFIG_$c),CONFIG_$(c)))

include $(INCLUDE_DIR)/package.mk

TAR_CMD=$(HOST_TAR) -C $(1)/.. $(TAR_OPTIONS)

define KernelPackage/warp
	CATEGORY:=MTK Properties
	TITLE:=MTK warp driver
	DEPENDS:= +kmod-mediatek_hnat
	FILES:=$(PKG_BUILD_DIR)/mtk_warp.ko
	AUTOLOAD:=$(call AutoLoad,60,mtk_warp)
	SUBMENU:=Drivers
	MENU:=1
endef

define KernelPackage/warp/config
	source "$(SOURCE)/config.in"
endef

define Build/Compile
	$(MAKE) -C "$(LINUX_DIR)" V=1 \
		$(KERNEL_MAKE_FLAGS) \
		M="$(PKG_BUILD_DIR)" \
		$(foreach c, $(PKG_KCONFIG),$(if $(CONFIG_$c),CONFIG_$(c)=$(CONFIG_$(c)))) \
		modules
endef

TARGET_CFLAGS += \
	$(foreach c, $(PKG_KCONFIG),$(if $(CONFIG_$c),-DCONFIG_$(c)=$(CONFIG_$c)))

MAKE_FLAGS += \
	$(foreach c, $(PKG_KCONFIG),$(if $(CONFIG_$c),CONFIG_$(c)=$(CONFIG_$c))) 

TARGET_CFLAGS += -DCONFIG_SUPPORT_OPENWRT
MAKE_FLAGS += CONFIG_SUPPORT_OPENWRT=y

define KernelPackage/warp/install
	$(INSTALL_DIR) $(1)/lib/firmware
	if [ "$$(CONFIG_MTK_WARP_FW_BIN_LOAD)" = "y" ]; then \
		if [ "$$(CONFIG_MTK_WARP_CHIPSET_MT7986)" = "y" ] ; then \
			if [ "$$(CONFIG_MTK_WARP_FIRMWARE_PATH_MT7986)" != "" ]; then \
				$(INSTALL_BIN) ./files/$$(CONFIG_MTK_WARP_FIRMWARE_PATH_MT7986)/* $(1)/lib/firmware/; \
			else \
				cp $(PKG_BUILD_DIR)/bin/7986*WOCPU*_RAM_CODE_release.bin $(1)/lib/firmware/; \
			fi ; \
		fi ; \
		if [ "$$(CONFIG_MTK_WARP_CHIPSET_MT7981)" = "y" ] ; then \
			if [ "$$(CONFIG_MTK_WARP_FIRMWARE_PATH_MT7981)" != "" ]; then \
				$(INSTALL_BIN) ./files/$$(CONFIG_MTK_WARP_FIRMWARE_PATH_MT7981)/* $(1)/lib/firmware/; \
			else \
				cp $(PKG_BUILD_DIR)/bin/7981*WOCPU*_RAM_CODE_release.bin $(1)/lib/firmware/; \
			fi ; \
		fi ; \
	else \
		if [ "$$(CONFIG_WARP_CHIPSET)" = "mt7981" ]; then \
			cp $(PKG_BUILD_DIR)/bin/7981*WOCPU*_RAM_CODE_release.bin $(1)/lib/firmware/; \
		elif [ "$$(CONFIG_WARP_CHIPSET)" = "mt7986" ]; then \
			cp $(PKG_BUILD_DIR)/bin/7986*WOCPU*_RAM_CODE_release.bin $(1)/lib/firmware/; \
		fi; \
	fi
endef

$(eval $(call KernelPackage,warp))
