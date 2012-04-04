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
OBJS = $(SRCS:%.c=%.bc)
DOBJS = $(SRCS:%.c=%.dbc)
LLT = $(LLTDIR)/libsupport-js.a
LIBTARGET = lib$(NAME)-js
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

%.o %.bc: %.c
	$(QUIET_CC) $(CC) $(SHIPFLAGS) -c $< -o $@
%.do %.dbc: %.c
	$(QUIET_CC) $(CC) $(DEBUGFLAGS) -c $< -o $@

flisp.o flisp.bc:   flisp.c cvalues.c types.c flisp.h print.c read.c equal.c
flisp.do flisp.dbc:  flisp.c cvalues.c types.c flisp.h print.c read.c equal.c
flmain.o flmain.bc:  flmain.c flisp.h
flmain.do flmain.dbc: flmain.c flisp.h

$(LLT): $(LLTDIR)/*.h $(LLTDIR)/*.c
	cd $(LLTDIR) && make $(LLT)

$(LIBTARGET)-debug.a: $(DOBJS)
	rm -rf $@
	$(QUIET_LINK) ar -rcs $@ $(DOBJS)
	ln -sf $@ $(LIBTARGET).a

$(LIBTARGET).a: $(OBJS)
	rm -rf $@
	$(QUIET_LINK) ar -rcs $@ $(OBJS)

$(LIBTARGET)-js-debug.a: $(DOBJS)
	rm -rf $@
	$(QUIET_LINK) $(AR) rcs $@ $(DOBJS)
	ln -sf $@ $(LIBTARGET)-js.a

$(LIBTARGET)-js.a: $(OBJS)
	rm -rf $@
	$(QUIET_LINK) $(AR) rcs $@ $(OBJS)

$(EXENAME)-debug: $(DOBJS) $(LIBFILES) $(LIBTARGET)-debug.a flmain.do
	$(QUIET_CC) $(CC) $(DEBUGFLAGS) $(DOBJS) flmain.do -o $(EXENAME)-debug $(LIBS) $(LIBTARGET).a $(OSLIBS)
	./$(EXENAME)-debug unittest.lsp
$(EXENAME)-debug.js: $(DOBJS) $(LIBFILES) $(LIBTARGET)-js-debug.a flmain.dbc
	$(QUIET_CC) $(CC) $(DEBUGFLAGS) $(DOBJS) flmain.dbc -o $(EXENAME)-debug.js $(LIBS) $(LIBTARGET)-js.a $(OSLIBS)
	$(NODEJS) ./$(EXENAME)-debug.js unittest.lsp

$(EXENAME): $(OBJS) $(LIBFILES) $(LIBTARGET).a flmain.o
	$(QUIET_CC) $(CC) $(SHIPFLAGS) $(OBJS) flmain.o -o $(EXENAME) $(LIBS) $(LIBTARGET).a $(OSLIBS)
	./$(EXENAME) unittest.lsp
$(EXENAME).js: $(OBJS) $(LIBFILES) $(LIBTARGET)-js.a flmain.bc
	$(QUIET_CC) $(CC) $(SHIPFLAGS) $(OBJS) flmain.bc -o $(EXENAME).js $(LIBS) $(LIBTARGET)-js.a $(OSLIBS)
	$(NODEJS) ./$(EXENAME).js unittest.lsp

clean:
	rm -f *.o *.bc
	rm -f *.do *.dbc
	rm -f *.a
	rm -f $(EXENAME)*
	rm -f $(EXENAME)-debug*
