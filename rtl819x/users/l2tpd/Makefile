#
# Layer Two Tunneling Protocol Daemon
# Copyright (C)1998 Adtran, Inc.
#
# Mark Spencer <markster@marko.net>
#
# This is free software.  You may distribute it under
# the terms of the GNU General Public License,
# version 2, or at your option any later version.
#
# Note on debugging flags:
# -DDEBUG_ZLB shows all ZLB exchange traffic
# -DDEBUG_CONTROL debugs the /var/run/l2tp-control pipe interface
# -DDEBUG_HELLO debugs when hello messages are sent
# -DDEBUG_CLOSE debugs call and tunnel closing
# -DDEBUG_FLOW debugs flow control system
# -DDEBUG_FILE debugs file format
# -DDEBUG_AAA debugs authentication, accounting, and access control
# -DDEBUG_PAYLOAD shows info on every payload packet
# -DDEBUG_CONTROL shows info on every control packet
# -DDEBUG_PPPD shows the command line of pppd
# -DDEBUG_HIDDEN debugs hidden AVP's
# -DDEBUG_ENTROPY debug entropy generation
# -DTEST_HIDDEN makes Assigned Call ID sent as a hidden AVP
#
# Also look at the top of network.c for some other (eventually to 
# become runtime options) debugging flags
#
##For test
##DFLAGS= -g -O2 -DDEBUG_PPPD
#DFLAGS=-DDEBUG_MAGIC -DDEBUG_FLOW -DDEBUG_CLOSE
#DFLAGS= -O2
#
# Uncomment the next line for Linux
#
OSFLAGS= -DLINUX
#
# Uncomment the following to use the kernel interface under Linux
#
#OSFLAGS+= -DUSE_KERNEL
#
# Uncomment the next line for FreeBSD
#
#OSFLAGS= -DFREEBSD
#
# Uncomment the next line for Solaris. For solaris, at least,
# we don't want to specify -I/usr/include because it is in
# the basic search path, and will over-ride some gcc-specific
# include paths and cause problems.
#
#OSFLAGS= -DSOLARIS
#OSLIBS= -lnsl -lsocket
#
# Feature flags
#
# Comment the following line to disable l2tpd maintaining IP address
# pools to pass to pppd to control IP address allocation
##CC= mips-uclibc-gcc
FFLAGS= -DIP_ALLOCATION

CFLAGS= $(DFLAGS) -fno-builtin -Wall -DSANITY $(OSFLAGS) $(FFLAGS) -DFOR_KEEP_ALIVE
CFLAGS += $(EXTRA_CFLAGS)

#reduce code size
CFLAGS += -Os

ifeq ($(wildcard $(DIR_LINUX)/.config),)
include ../../linux-2.6.30/.config
else
KERNEL_VER=$(patsubst $(DIR_ROOT)/%,%,$(DIR_LINUX))
ifeq ($(KERNEL_VER),linux-2.6.30)
include ../../linux-2.6.30/.config
endif
ifeq ($(KERNEL_VER),linux-3.10)
include ../../linux-3.10/.config
endif
endif
#include ../../linux-2.6.30/.config

ifeq ($(CONFIG_SUPPORT_RUSSIA_FEATURES),y)
CFLAGS += -DSUPPORT_ZIONCOM_RUSSIA
endif
ifeq ($(CONFIG_RTL_8198C),y)
     CFLAGS += -DCONFIG_RTL_8198C
endif
HDRS=l2tp.h avp.h misc.h control.h call.h scheduler.h file.h aaa.h md5.h
OBJS=l2tpd.o pty.o misc.o control.o avp.o call.o network.o avpsend.o scheduler.o file.o aaa.o md5.o
#LIBS= $(OSLIB) # -lefence # efence for malloc checking
EXEC=l2tpd

all: $(EXEC)

clean:
	rm -f $(OBJS) $(EXEC)

$(EXEC): $(OBJS) $(HDRS)
	$(CC) $(LDFLAGS) -o $@ $(OBJS) $(LDLIBS)
	$(STRIP) $@

romfs:
	$(ROMFSINST) /bin/$(EXEC)
	cp ./script_check_l2tp_status.sh $(DIR_ROMFS)/bin/

