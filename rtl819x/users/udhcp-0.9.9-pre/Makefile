# udhcp makefile

prefix=/usr
SBINDIR=/sbin
USRSBINDIR=${prefix}/sbin
USRBINDIR=${prefix}/bin
USRSHAREDIR=${prefix}/share

# Uncomment this to get a shared binary. Call as udhcpd for the server,
# and udhcpc for the client
COMBINED_BINARY=1
# Uncomment this for extra output and to compile with debugging symbols
#DEBUG=1

# Uncomment this to output messages to syslog, otherwise, messages go to stdout
#CFLAGS += -DSYSLOG

#CROSS_COMPILE=mips-uclibc-
#CROSS_COMPILE=arm-uclibc-
#CC = $(CROSS_COMPILE)gcc
#LD = $(CROSS_COMPILE)gcc
INSTALL = install

VER := 0.9.8

CINIT = 1

OBJS_SHARED = options.o socket.o packet.o pidfile.o
DHCPD_OBJS = dhcpd.o arpping.o files.o leases.o serverpacket.o
DHCPC_OBJS = dhcpc.o clientpacket.o script.o

ifeq ($(wildcard $(DIR_LINUX)/.config),)
include $(DIR_ROOT)/linux-2.6.30/.config
else
include $(DIR_LINUX)/.config
endif

#include ../../linux-2.6.30/.config

include ../../users/.config

ifeq ($(CONFIG_APP_SIMPLE_CONFIG),y)
CFLAGS += -DCONFIG_APP_SIMPLE_CONFIG
endif

ifeq ($(CONFIG_RTL_ULINKER),y)
CFLAGS += -DCONFIG_RTL_ULINKER
endif
USE_BOUND_TEMPLETE=0
ifeq ($(CONFIG_APP_APPLE_MFI_WAC),y)
USE_BOUND_TEMPLETE=1
CFLAGS += -DCONFIG_APP_APPLE_MFI_WAC
endif

ifeq ($(CONFIG_LEADCORE_4G_SUPPORT),y)
CFLAGS += -DCONFIG_LEADCORE_4G_SUPPORT
endif

ifdef COMBINED_BINARY
EXEC1 = udhcpd
OBJS1 = $(DHCPD_OBJS) $(DHCPC_OBJS) $(OBJS_SHARED) frontend.o
CFLAGS += -DCOMBINED_BINARY
else
EXEC1 = udhcpd
OBJS1 = $(DHCPD_OBJS) $(OBJS_SHARED)

EXEC2 = udhcpc
OBJS2 = $(DHCPC_OBJS) $(OBJS_SHARED)
endif

EXEC3 = dumpleases
OBJS3 = dumpleases.o

BOOT_PROGRAMS = udhcpc
DAEMONS = udhcpd
COMMANDS = dumpleases

ifdef SYSLOG
CFLAGS += -DSYSLOG
endif

CFLAGS += -W -Wall -Wstrict-prototypes -DVERSION='"$(VER)"'

ifeq ($(RTL8186_KB),1)
CFLAGS+=-DCONFIG_RTL8186_KB -DCHECK_SERVER_ALIVE
endif

ifeq ($(RTL8186_TR),1)
########## Build with MIB Library ##################
GWSUPP	= -DHOME_GATEWAY -DWIFI_SIMPLE_CONFIG -DDOS_SUPPORT -DROUTE_SUPPORT -DUSE_SHARED_DAEMON
MIBLIBDIR	= $(PWD)/./tnet/boa/src
MIBLIBINC	= $(MIBLIBDIR)
MIBLIB		= $(MIBLIBDIR)/apmib.so
INCS = -I$(MIBLIBINC)
LIBS += $(MIBLIB)
CFLAGS+=-DCONFIG_RTL8186_TR -DSTATIC_LEASE $(GWSUPP) $(INCS)
STATIC_LEASE=1
RTL865x_TR_PKG=1
endif

ifeq ($(CONFIG_APP_TR069), y)
CFLAGS += -DCONFIG_APP_TR069 -D_PRMT_X_TELEFONICA_ES_DHCPOPTION_
CFLAGS += -DTR069_ANNEX_F
CFLAGS += -DSUPPORT_RESERVED_DHCP_IPADDR
#CFLAGS += -DSET_TELE_CONN_REQ_URL
MIBLIBDIR       = ../boa/apmib
MIBLIBINC       = $(MIBLIBDIR)
MIBLIB          = $(MIBLIBDIR)/libapmib.so
INCS = -I$(MIBLIBINC)
LIBS += $(MIBLIB)
CFLAGS += $(INCS)
endif

ifeq ($(RTL865X_AC),1)
CFLAGS+=-DCONFIG_RTL865X_AC 
RTL865x_AC_PKG=1
endif

ifeq ($(RTL865X_KLD),1)
CFLAGS+=-DCONFIG_RTL865X_KLD
RTL865x_KLD_PKG=1
endif

ifeq ($(CONFIG_RTL865X_SC),1)
CFLAGS+=-DCONFIG_RTL865X_SC
RTL865x_SC_PKG=1
endif

ifeq ($(MD),615-96b)
CFLAGS+=-DGUEST_ZONE
endif

CFLAGS+=-DSTATIC_LEASE

#ifndef RTL865x_TR_PKG
#ifndef RTL865x_AC_PKG
#ifndef RTL865x_SC_PKG
#ifndef RTL865x_KLD_PKG
CFLAGS+=-DCHECK_SERVER_ALIVE
#endif
#endif
#endif
#endif
STATIC_LEASE=1


ifdef STATIC_LEASE
OBJS1 += static_leases.o
endif

ifdef DEBUG
CFLAGS += -g -DDEBUG
STRIP=true
else
CFLAGS += -Os -fomit-frame-pointer
STRIP=$(CROSS_COMPILE)strip
endif

ifeq ($(SLINK),1)
CFLAGS  += -ffunction-sections -fdata-sections
LDFLAGS += --static -Wl,--gc-sections
endif

CFLAGS += -DSUPPORT_T1_T2_OPTION
ifeq ($(CONFIG_SUPPORT_RUSSIA_FEATURES),y)
CFLAGS += -DSUPPORT_OPTION_33_121_249 -DSUPPORT_ZIONCOM_RUSSIA
endif
ifeq ($(CONFIG_RTK_SUPPORT_DHCP_PORT_IP_BIND),y)
CFLAGS += -DSUPPORT_DHCP_PORT_IP_BIND
endif
all: $(EXEC1) $(EXEC2) $(EXEC3)
	$(STRIP) --remove-section=.note --remove-section=.comment $(EXEC1) $(EXEC2) $(EXEC3)

$(OBJS1) $(OBJS2) $(OBJS3): *.h Makefile
$(EXEC1) $(EXEC2) $(EXEC3): Makefile

.c.o:
	$(CC) -c $(CFLAGS) $<
	
$(EXEC1): $(OBJS1)
	$(LD) $(LDFLAGS) $(OBJS1) $(LIBS) -o $(EXEC1)

$(EXEC2): $(OBJS2)
	$(LD) $(LDFLAGS) $(OBJS2) -o $(EXEC2)

$(EXEC3): $(OBJS3)
	$(LD) $(LDFLAGS) $(OBJS3) -o $(EXEC3)


install: all

	$(INSTALL) $(DAEMONS) $(USRSBINDIR)
	$(INSTALL) $(COMMANDS) $(USRBINDIR)
ifdef COMBINED_BINARY
	ln -sf $(USRSBINDIR)/$(DAEMONS) $(SBINDIR)/$(BOOT_PROGRAMS)
else
	$(INSTALL) $(BOOT_PROGRAMS) $(SBINDIR)
endif
	mkdir -p $(USRSHAREDIR)/udhcpc
	for name in bound deconfig renew script ; do \
		$(INSTALL) samples/sample.$$name \
			$(USRSHAREDIR)/udhcpc/default.$$name ; \
	done
	mkdir -p $(USRSHAREDIR)/man/man1
	$(INSTALL) dumpleases.1 $(USRSHAREDIR)/man/man1
	mkdir -p $(USRSHAREDIR)/man/man5
	$(INSTALL) udhcpd.conf.5 $(USRSHAREDIR)/man/man5
	mkdir -p $(USRSHAREDIR)/man/man8
	$(INSTALL) udhcpc.8 udhcpd.8 $(USRSHAREDIR)/man/man8

romfs:
	$(ROMFSINST) udhcpd /bin/udhcpd
	
	mkdir -p $(DIR_ROMFS)/usr/share
	mkdir -p $(DIR_ROMFS)/usr/share/udhcpc
ifneq ($(MP),1)
	cp ./mips-scripts/* $(DIR_ROMFS)/usr/share/udhcpc
ifeq ($(CINIT), 1)
	cp ../script/cinit/eth1.bound $(DIR_ROMFS)/usr/share/udhcpc/eth1.bound
	cp ../script/cinit/eth1.renew $(DIR_ROMFS)/usr/share/udhcpc/eth1.renew
	cp ../script/cinit/wlan0.bound $(DIR_ROMFS)/usr/share/udhcpc/wlan0.bound
	cp ../script/cinit/wlan0.renew $(DIR_ROMFS)/usr/share/udhcpc/wlan0.renew
	cp ../script/cinit/wlan1.bound $(DIR_ROMFS)/usr/share/udhcpc/wlan1.bound
	cp ../script/cinit/wlan1.renew $(DIR_ROMFS)/usr/share/udhcpc/wlan1.renew
	cp ../script/cinit/wlan0-vxd.bound $(DIR_ROMFS)/usr/share/udhcpc/wlan0-vxd.bound
	cp ../script/cinit/wlan0-vxd.renew $(DIR_ROMFS)/usr/share/udhcpc/wlan0-vxd.renew
	cp ../script/cinit/wlan1-vxd.bound $(DIR_ROMFS)/usr/share/udhcpc/wlan1-vxd.bound
	cp ../script/cinit/wlan1-vxd.renew $(DIR_ROMFS)/usr/share/udhcpc/wlan1-vxd.renew
	cp ../script/cinit/br0.bound $(DIR_ROMFS)/usr/share/udhcpc/br0.bound
	cp ../script/cinit/br0.renew $(DIR_ROMFS)/usr/share/udhcpc/br0.renew
	cp ../script/cinit/usb0.bound $(DIR_ROMFS)/usr/share/udhcpc/usb0.bound
	cp ../script/cinit/usb0.renew $(DIR_ROMFS)/usr/share/udhcpc/usb0.renew
	cp ../script/cinit/eth1.1.bound $(DIR_ROMFS)/usr/share/udhcpc/eth1.1.bound
	cp ../script/cinit/eth1.1.renew $(DIR_ROMFS)/usr/share/udhcpc/eth1.1.renew
	cp ../script/cinit/eth1.2.bound $(DIR_ROMFS)/usr/share/udhcpc/eth1.2.bound
	cp ../script/cinit/eth1.2.renew $(DIR_ROMFS)/usr/share/udhcpc/eth1.2.renew
	cp ../script/cinit/eth1.3.bound $(DIR_ROMFS)/usr/share/udhcpc/eth1.3.bound
	cp ../script/cinit/eth1.3.renew $(DIR_ROMFS)/usr/share/udhcpc/eth1.3.renew
	cp ../script/cinit/eth1.4.bound $(DIR_ROMFS)/usr/share/udhcpc/eth1.4.bound
	cp ../script/cinit/eth1.4.renew $(DIR_ROMFS)/usr/share/udhcpc/eth1.4.renew
endif
	$(ROMFSINST) -s /var/udhcpc /etc/
	rm $(DIR_ROMFS)/usr/share/udhcpc/*.deconfig
	$(ROMFSINST) -s /var/udhcpc/br0.deconfig /usr/share/udhcpc/br0.deconfig
	$(ROMFSINST) -s /var/udhcpc/eth0.deconfig /usr/share/udhcpc/eth0.deconfig
	$(ROMFSINST) -s /var/udhcpc/eth1.deconfig /usr/share/udhcpc/eth1.deconfig
	$(ROMFSINST) -s /var/udhcpc/wlan0.deconfig /usr/share/udhcpc/wlan0.deconfig
	$(ROMFSINST) -s /var/udhcpc/wlan1.deconfig /usr/share/udhcpc/wlan1.deconfig
	$(ROMFSINST) -s /var/udhcpc/wlan0-vxd.deconfig /usr/share/udhcpc/wlan0-vxd.deconfig
	$(ROMFSINST) -s /var/udhcpc/wlan1-vxd.deconfig /usr/share/udhcpc/wlan1-vxd.deconfig
	$(ROMFSINST) -s /var/udhcpc/usb0.deconfig /usr/share/udhcpc/usb0.deconfig
	$(ROMFSINST) -s /var/udhcpc/eth1.1.deconfig /usr/share/udhcpc/eth1.1.deconfig
	$(ROMFSINST) -s /var/udhcpc/eth1.2.deconfig /usr/share/udhcpc/eth1.2.deconfig
	$(ROMFSINST) -s /var/udhcpc/eth1.3.deconfig /usr/share/udhcpc/eth1.3.deconfig
	$(ROMFSINST) -s /var/udhcpc/eth1.4.deconfig /usr/share/udhcpc/eth1.4.deconfig
	$(ROMFSINST) -s udhcpd /bin/udhcpc
	$(ROMFSINST) -s /var/udhcpd /etc
endif

clean:
	-rm -f udhcpd udhcpc dumpleases *.o core



