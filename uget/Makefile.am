## To enable LFS (Large File Support) in 32bit platform
## add `getconf LFS_CFLAGS`  to CFLAGS
## add `getconf LFS_LDFLAGS` to LDFLAGS

## static library ---

#  lib_LIBRARIES = libuget.a
noinst_LIBRARIES = libuget.a

libuget_a_CPPFLAGS = -I$(top_srcdir)/uglib  -I$(top_srcdir)/uget
libuget_a_CFLAGS   = @PTHREAD_CFLAGS@  @GLIB_CFLAGS@ @LIBCRYPTO_CFLAGS@ @LIBGCRYPT_CFLAGS@

libuget_a_SOURCES = \
	UgetSequence.c  \
	UgetRss.c     \
	UgetRpc.c     \
	UgetOption.c  \
	UgetData.c    \
	UgetFiles.c   \
	UgetNode.c    \
	UgetNode-compare.c  \
	UgetNode-filter.c   \
	UgetTask.c    \
	UgetHash.c    \
	UgetSite.c    \
	UgetApp.c     \
	UgetEvent.c   \
	UgetPlugin.c  \
	UgetA2cf.c    \
	UgetCurl.c    \
	UgetAria2.c   \
	UgetMedia.c   \
	UgetMedia-youtube.c   \
	UgetPluginCurl.c   \
	UgetPluginAria2.c  \
	UgetPluginMedia.c  \
	UgetPluginAgent.c  \
	UgetPluginMega.c   \
	UgetPluginEmpty.c

noinst_HEADERS = \
	UgetSequence.h  \
	UgetRss.h     \
	UgetRpc.h     \
	UgetOption.h  \
	UgetData.h    \
	UgetFiles.h   \
	UgetNode.h    \
	UgetTask.h    \
	UgetHash.h    \
	UgetSite.h    \
	UgetApp.h     \
	UgetEvent.h   \
	UgetPlugin.h  \
	UgetA2cf.h    \
	UgetCurl.h    \
	UgetAria2.h   \
	UgetMedia.h   \
	UgetPluginCurl.h   \
	UgetPluginAria2.h  \
	UgetPluginMedia.h  \
	UgetPluginAgent.h  \
	UgetPluginMega.h   \
	UgetPluginEmpty.h

if WITH_LIBPWMD
libuget_a_CFLAGS += @LIBPWMD_CFLAGS@
libuget_a_SOURCES += pwmd.c
noinst_HEADERS += pwmd.h
endif

EXTRA_DIST = \
	Android.mk     \
	CMakeLists.txt
