#
# Copyright (C) 2008 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
# Upgraded from 4.8.3 to 5.3.0

include $(TOPDIR)/rules.mk

PKG_NAME:=gcc
PKG_VERSION:=5.3.0
PKG_RELEASE:=3
PKG_SOURCE_URL:=ftp://ftp.fu-berlin.de/unix/languages/gcc/releases/gcc-$(PKG_VERSION) \
     http://mirrors.rcn.net/pub/sourceware/gcc/releases/gcc-$(PKG_VERSION) \
    ftp://ftp.gnu.org/gnu/gcc/releases/gcc-$(PKG_VERSION)

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.bz2
PKG_MD5SUM:=c9616fd448f980259c31de613e575719
PATCH_DIR:=$(TOPDIR)/toolchain/gcc/patches/$(PKG_VERSION)
PKG_INSTALL:=1
PKG_FIXUP:=libtool
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

TARGET_LANGUAGES:="c,c++"
BUGURL=https://dev.openwrt.org/
PKGVERSION=OpenWrt GCC $(PKG_VERSION)

# not using sstrip here as this fucks up the .so's somehow
STRIP:=$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)strip
RSTRIP:= \
       NM="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)nm" \
       STRIP="$(STRIP)" \
       STRIP_KMOD="$(STRIP) --strip-debug" \
       $(SCRIPT_DIR)/rstrip.sh


define Package/gcc
  SECTION:=devel
  CATEGORY:=Development
  TITLE:=gcc
  MAINTAINER:=Christian Beier <cb@shoutrlabs.com>
  DEPENDS:= +binutils +libstdcpp
endef

define Package/gcc/description
	build a native toolchain for compiling on target
endef

GMPSRC=gmp-4.3.2

define Download/gmp
  URL:=ftp://gcc.gnu.org/pub/gcc/infrastructure/
  FILE:=$(GMPSRC).tar.bz2
  MD5SUM:=dd60683d7057917e34630b4a787932e8
endef
$(eval $(call Download,gmp))

MPCSRC=mpc-0.8.1

define Download/mpc
  URL:=ftp://gcc.gnu.org/pub/gcc/infrastructure/
  FILE:=$(MPCSRC).tar.gz
  MD5SUM:=5b34aa804d514cc295414a963aedb6bf
endef
$(eval $(call Download,mpc))

MPFRSRC=mpfr-2.4.2

define Download/mpfr
  URL:=ftp://gcc.gnu.org/pub/gcc/infrastructure/
  FILE:=$(MPFRSRC).tar.bz2
  MD5SUM:=89e59fe665e2b3ad44a6789f40b059a0
endef
$(eval $(call Download,mpfr))

define Build/Prepare
	$(PKG_UNPACK)
# 	we have to download and unpack additional stuff before patching
	tar -C $(PKG_BUILD_DIR) -xvjf $(DL_DIR)/$(GMPSRC).tar.bz2
	ln -sf $(PKG_BUILD_DIR)/$(GMPSRC) $(PKG_BUILD_DIR)/gmp
	tar -C $(PKG_BUILD_DIR) -xvzf $(DL_DIR)/$(MPCSRC).tar.gz
	ln -sf $(PKG_BUILD_DIR)/$(MPCSRC) $(PKG_BUILD_DIR)/mpc
	tar -C $(PKG_BUILD_DIR) -xvjf $(DL_DIR)/$(MPFRSRC).tar.bz2
	ln -sf $(PKG_BUILD_DIR)/$(MPFRSRC) $(PKG_BUILD_DIR)/mpfr
	$(Build/Patch)
	patch -p1 -d "$(PKG_BUILD_DIR)" < ./patches/002-dont-choke-when-building-32bit-on-64bit.patch
	patch -p1 -d "$(PKG_BUILD_DIR)" < ./patches/003-mpfr-longlong-mips.patch
#uclibc needs -lm to build libcc1
ifdef CONFIG_USE_UCLIBC
	patch -p1 -d "$(PKG_BUILD_DIR)" < ./patches/004-libcc1_uclibc_lm.patch
endif
#	poor man's fix for `none-openwrt-linux' not recognized when building with musl
	cp $(PKG_BUILD_DIR)/config.sub $(PKG_BUILD_DIR)/mpfr/
	cp $(PKG_BUILD_DIR)/config.sub $(PKG_BUILD_DIR)/gmp/
	cp $(PKG_BUILD_DIR)/config.sub $(PKG_BUILD_DIR)/mpc/
endef


define Build/Configure
	(cd $(PKG_BUILD_DIR); rm -f config.cache; \
		SHELL="$(BASH)" \
		$(TARGET_CONFIGURE_OPTS) \
		$(PKG_BUILD_DIR)/configure \
			$(CONFIGURE_ARGS) \
			--build=$(GNU_HOST_NAME) \
			--host=$(REAL_GNU_TARGET_NAME) \
			--target=$(REAL_GNU_TARGET_NAME) \
			--enable-languages=$(TARGET_LANGUAGES) \
			--with-bugurl=$(BUGURL) \
			--with-pkgversion="$(PKGVERSION)" \
			--enable-shared \
			$(if $(CONFIG_LIBC_USE_GLIBC),--enable,--disable)-__cxa_atexit \
			--with-default-libstdcxx-abi=gcc4-compatible \
			--enable-target-optspace \
			--with-gnu-ld \
			--disable-nls \
			--disable-libmudflap \
			--disable-multilib \
			--disable-libgomp \
			--disable-libquadmath \
			--disable-libssp \
			--disable-decimal-float \
			--disable-libstdcxx-pch \
			--with-host-libstdcxx=-lstdc++ \
			--prefix=/usr \
			--libexecdir=/usr/lib \
			$(SOFT_FLOAT_CONFIG_OPTION) \
			$(call qstrip,$(CONFIG_EXTRA_GCC_CONFIG_OPTIONS)) \
	);
endef

define Build/Compile
	export SHELL="$(BASH)"; $(MAKE_VARS) $(MAKE) -C $(PKG_BUILD_DIR) \
			DESTDIR="$(PKG_INSTALL_DIR)" $(MAKE_ARGS) all install
endef

define Package/gcc/extra_provides
    echo 'libm.so.6'; \
    echo 'libc.so.6';
endef

define Package/gcc/install
	$(INSTALL_DIR) $(1)/usr/bin $(1)/usr/lib $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	cp -ar $(PKG_INSTALL_DIR)/usr/include $(1)/usr
	cp -a $(PKG_INSTALL_DIR)/usr/bin/{$(REAL_GNU_TARGET_NAME)-{g++,gcc},cpp,gcov} $(1)/usr/bin
	ln -s $(REAL_GNU_TARGET_NAME)-g++ $(1)/usr/bin/c++
	ln -s $(REAL_GNU_TARGET_NAME)-g++ $(1)/usr/bin/g++
	ln -s $(REAL_GNU_TARGET_NAME)-g++ $(1)/usr/bin/$(REAL_GNU_TARGET_NAME)-c++
	ln -s $(REAL_GNU_TARGET_NAME)-gcc $(1)/usr/bin/gcc
	ln -s $(REAL_GNU_TARGET_NAME)-gcc $(1)/usr/bin/$(REAL_GNU_TARGET_NAME)-gcc-$(PKG_VERSION)
	cp -ar $(PKG_INSTALL_DIR)/usr/lib/gcc $(1)/usr/lib
	$(RM) $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)/*.a
	cp -ar $(TOOLCHAIN_DIR)/include $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	cp -a $(TOOLCHAIN_DIR)/lib/*.{o,so*} $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	cp -a $(TOOLCHAIN_DIR)/lib/*nonshared*.a  $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	grep "GROUP.*-lgcc" $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)/libgcc_s.so && cp -a $(PKG_INSTALL_DIR)/usr/lib/gcc/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)/libgcc.a $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)/ ; true
endef

$(eval $(call BuildPackage,gcc))
