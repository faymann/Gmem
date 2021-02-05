CLANG    := clang-10
OPT      := opt-10
LLINK    := llvm-link-10
LLVMCONF := llvm-config-10

RW         ?= w
CHECK_METHOD ?= ifelseheap
CHECK_EXT_CALL_ARGS ?= false

CFLAGS  := -Wall -g -O3 -emit-llvm -c -I../
OPTFLAGS := -load ../obj/libGmem-pass.so -gmem 			\
            -Gmem-rw=$(RW)  							\
			-Gmem-check-method=$(CHECK_METHOD)  					\
			-Gmem-verify-external-call-args=$(CHECK_EXT_CALL_ARGS) 	\
			-Gmem-whitelist-section=safe_functions -O3

LDFLAGS := -g -O3
OBJDIR  ?= ./obj

PROGS := test
PROGS := $(patsubst %,$(OBJDIR)/%,$(PROGS))

AUXFILES := $(wildcard ../obj/Auxiliary.o)

.PHONY: all clean

all: $(PROGS)

$(OBJDIR)/%: $(OBJDIR)/%-opt.bc
	$(CLANG) -o $@ $^ $(LDFLAGS)

$(OBJDIR)/%-opt.bc: $(OBJDIR)/%-link.bc
	$(OPT) $(OPTFLAGS) < $< > $@

$(OBJDIR)/%-link.bc: $(OBJDIR)/%.bc $(AUXFILES)
	$(LLINK) $^ > $@

$(OBJDIR)/%.bc: %.c | $(OBJDIR)
	$(CLANG) $(CFLAGS) -o $@ $<

$(OBJDIR):
	mkdir -p $@

clean:
	rm -rf $(OBJDIR)
