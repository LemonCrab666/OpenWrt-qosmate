include $(TOPDIR)/rules.mk

PKG_NAME:=qosmate
PKG_VERSION:=0.5.26
PKG_RELEASE:=1

include $(INCLUDE_DIR)/package.mk

define Package/qosmate
  SECTION:=net
  CATEGORY:=Network
  TITLE:=QoS Mate
  DEPENDS:=+tc +iptables
endef

define Package/qosmate/description
  QoS Mate is a lightweight traffic control tool for OpenWrt.
endef

# Installation instructions
define Build/Compile
  # No compile step required for this package
endef

define Package/qosmate/install
  # Install configuration files
  $(INSTALL_DIR) $(1)/etc/init.d
  $(INSTALL_BIN) ./etc/init.d/qosmate.sh $(1)/etc/init.d/qosmate

  $(INSTALL_DIR) $(1)/etc/hotplug.d/iface
  $(INSTALL_BIN) ./etc/hotplug.d/iface/00-qosmate $(1)/etc/hotplug.d/iface/00-qosmate

  $(INSTALL_DIR) $(1)/usr/lib/tc
  $(INSTALL_BIN) ./usr/lib/tc/m_xt.so $(1)/usr/lib/tc/m_xt.so
  $(INSTALL_DATA) ./usr/lib/tc/*.dist $(1)/usr/lib/tc/
endef

# Package installation steps
$(eval $(call BuildPackage,qosmate))
