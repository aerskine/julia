JULIAHOME = $(abspath ../..)
include $(JULIAHOME)/Make.inc

override CFLAGS += $(JCFLAGS)
override CXXFLAGS += $(JCXXFLAGS)

NAME = flisp
EXENAME = $(NAME)
LIBTARGET = lib$(NAME)

SRCS = flisp.c builtins.c string.c equalhash.c table.c iostream.c \
	julia_extensions.c
OBJS = $(SRCS:%.c=%.o)
DOBJS = $(SRCS:%.c=%.do)
LLTDIR = ../support
LLT = $(LLTDIR)/libsupport.a

ifeq ($(USEEMCC), 1)
OBJS = $(SRCS:%.c=%-emcc.o)
DOBJS = $(SRCS:%.c=%-emcc.do)
LLT = $(LLTDIR)/libsupport-emcc.a
endif

FLAGS = -Wall -Wno-strict-aliasing -I$(LLTDIR) $(CFLAGS) \
	-DUSE_COMPUTED_GOTO $(HFILEDIRS:%=-I%) $(LIBDIRS:%=-L%) -fvisibility=hidden
LIBFILES = $(LLT)
LIBS = $(LIBFILES) -lm

DEBUGFLAGS += $(FLAGS)
SHIPFLAGS += $(FLAGS)

default: release

release: $(EXENAME)

debug: $(EXENAME)-debug

%.o %-emcc.o: %.c
	$(QUIET_CC) $(CC) $(SHIPFLAGS) -c $< -o $@
%.do %-emcc.do: %.c
	$(QUIET_CC) $(CC) $(DEBUGFLAGS) -c $< -o $@

flisp.o flisp-emcc.o:   flisp.c cvalues.c types.c flisp.h print.c read.c equal.c
flisp.do flisp-emcc.do:  flisp.c cvalues.c types.c flisp.h print.c read.c equal.c
flmain.o flmain-emcc.o:  flmain.c flisp.h
flmain.do flmain-emcc.do: flmain.c flisp.h

$(LLT): $(LLTDIR)/*.h $(LLTDIR)/*.c
	cd $(LLTDIR) && make $(LLT)

libsupport-emcc-debug.a: $(LLTDIR)/*.h $(LLTDIR)/*.c
	cd $(LLTDIR) && make libsupport-emcc-debug.a

$(LIBTARGET)-debug.a: $(DOBJS)
	rm -rf $@
	$(QUIET_LINK) ar -rcs $@ $(DOBJS)
	ln -sf $@ $(LIBTARGET).a

$(LIBTARGET).a: $(OBJS)
	rm -rf $@
	$(QUIET_LINK) ar -rcs $@ $(OBJS)

$(LIBTARGET)-emcc-debug.a: $(DOBJS)
	rm -rf $@
	$(QUIET_LINK) $(AR) rcs $@ $(DOBJS)

$(LIBTARGET)-emcc.a: $(OBJS)
	rm -rf $@
	$(QUIET_LINK) $(AR) rcs $@ $(OBJS)

$(EXENAME)-debug: $(DOBJS) $(LIBFILES) $(LIBTARGET)-debug.a flmain.do
	$(QUIET_CC) $(CC) $(DEBUGFLAGS) $(DOBJS) flmain.do -o $(EXENAME)-debug $(LIBS) $(LIBTARGET).a $(OSLIBS)
	./$(EXENAME)-debug unittest.lsp
$(EXENAME)-debug.js: $(DOBJS) libsupport-emcc-debug.a $(LIBTARGET)-emcc-debug.a flmain-emcc.do
	@echo $(CC) $(DEBUGFLAGS) $(DOBJS) flmain-emcc.do -o $(EXENAME)-debug.bc $(LIBS) $(LIBTARGET)-emcc-debug.a $(OSLIBS)
	$(CC) $(DEBUGFLAGS) $(DOBJS) flmain-emcc.do -o $(EXENAME)-debug.js $(LIBS) $(LIBTARGET)-emcc-debug.a $(OSLIBS)
	$(NODEJS) ./$(EXENAME)-debug.js unittest.lsp
$(EXENAME)-debug.html: $(DOBJS) libsupport-emcc-debug.a $(LIBTARGET)-emcc-debug.a flmain-emcc.do
	@echo $(CC) $(DEBUGFLAGS) $(DOBJS) flmain-emcc.do -o $(EXENAME)-debug.bc $(LIBS) $(LIBTARGET)-emcc-debug.a $(OSLIBS)
	$(CC) $(DEBUGFLAGS) $(DOBJS) flmain-emcc.do -o $(EXENAME)-debug.html $(LIBS) $(LIBTARGET)-emcc-debug.a $(OSLIBS)


$(EXENAME): $(OBJS) $(LIBFILES) $(LIBTARGET).a flmain.o
	$(QUIET_CC) $(CC) $(SHIPFLAGS) $(OBJS) flmain.o -o $(EXENAME) $(LIBS) $(LIBTARGET).a $(OSLIBS)
	./$(EXENAME) unittest.lsp
$(EXENAME).js: $(OBJS) $(LIBFILES) $(LIBTARGET)-emcc.a flmain.bc
	$(QUIET_CC) $(CC) $(SHIPFLAGS) $(OBJS) flmain.bc -o $(EXENAME).js $(LIBS) $(LIBTARGET)-emcc.a $(OSLIBS)
	$(NODEJS) ./$(EXENAME).js unittest.lsp

clean:
	rm -f *.o *.bc
	rm -f *.do *.dbc
	rm -f *.a
	rm -f $(EXENAME)
	rm -f $(EXENAME)-debug
	rm -f $(EXENAME).js
	rm -f $(EXENAME)-debug.js
