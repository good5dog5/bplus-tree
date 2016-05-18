CC ?= gcc
CFLAGS_common ?= -Wall -std=gnu99 -g
CPPFLAGS_common ?= -I../../include -Wall -g
CFLAGS_orig = -O0
CFLAGS_opt  = -O0
LDFLAGS += -lpthread

MODE ?= debug
SNAPPY ?= 1

EXEC = phonebook_orig phonebook_opt phonebook_hash phonebook_trie phonebook_rbtree phonebook_bptree phonebook_bptree_bulk
all: $(EXEC)

SRCS_common = main.c memory_pool.c

phonebook_orig: $(SRCS_common) phonebook_orig.c phonebook_orig.h
	$(CC) $(CFLAGS_common) $(CFLAGS_orig) \
		-DIMPL="\"$@.h\"" -o $@ \
		$(SRCS_common) $@.c

phonebook_opt: $(SRCS_common) phonebook_opt.c phonebook_opt.h
	$(CC) $(CFLAGS_common) $(CFLAGS_opt) \
		-DIMPL="\"$@.h\"" -DOPT -o $@ \
		$(SRCS_common) $@.c

phonebook_hash: $(SRCS_common) phonebook_opt.c phonebook_opt.h
	$(CC) $(CFLAGS_common) $(CFLAGS_opt) \
		-DIMPL="\"phonebook_opt.h\"" -DHASH -o $@ \
		$(SRCS_common) phonebook_opt.c

phonebook_trie: $(SRCS_common) phonebook_trie.c phonebook_trie.h
	$(CC) $(CFLAGS_common) $(CFLAGS_opt) \
		-DIMPL="\"$@.h\"" -DTRIE -o $@ \
		$(SRCS_common) $@.c

phonebook_rbtree: $(SRCS_common) phonebook_rbtree.c phonebook_rbtree.h
	$(CC) $(CFLAGS_common) $(CFLAGS_opt) \
		-DIMPL="\"$@.h\"" -DRBTREE -o $@ \
		$(SRCS_common) $@.c

phonebook_bptree: $(SRCS_common) ../../bplus.a
	$(CXX) $(CPPFLAGS_common) $(CFLAGS_opt) \
		-DIMPL="\"phonebook_orig.h\"" -DBPTREE -o $@ \
		$(SRCS_common) ../../bplus.a $(LDFLAGS)

phonebook_bptree_bulk: $(SRCS_common) ../../bplus.a
	$(CXX) $(CPPFLAGS_common) $(CFLAGS_opt) \
		-DIMPL="\"phonebook_orig.h\"" -DBPTREE_BULK -o $@ \
		$(SRCS_common) ../../bplus.a $(LDFLAGS)

../../bplus.a:
	$(MAKE) -C ../.. MODE=$(MODE) SNAPPY=$(SNAPPY)

run: $(EXEC)
	echo 3 | sudo tee /proc/sys/vm/drop_caches
	watch -d -t "./phonebook_orig && echo 3 | sudo tee /proc/sys/vm/drop_caches"

# $(call perf-stat, repeat-times, executable)
define perf-stat
	perf stat --repeat $1 								 \
	-e cache-misses,cache-references,instructions,cycles \
	./$(strip $2)
endef

cache-test: $(EXEC)
	$(call perf-stat, 100, phonebook_orig)
	$(call perf-stat, 100, phonebook_opt)
	$(call perf-stat, 100, phonebook_hash)
	$(call perf-stat, 100, phonebook_trie)
	$(call perf-stat, 100, phonebook_rbtree)
	$(call perf-stat, 10, phonebook_bptree)
	$(call perf-stat, 10, phonebook_bptree_bulk)

output.txt: cache-test calculate
	./calculate

plot: output.txt
	gnuplot scripts/runtime.gp

calculate: calculate.c
	$(CC) $(CFLAGS_common) $^ -o $@

.PHONY: clean
clean:
	$(RM) $(EXEC) *.o perf.* \
	      	calculate rbtree.txt bptree.txt bptree_bulk.txt trie.txt hash.txt orig.txt opt.txt output.txt runtime.png
	$(MAKE) -C ../.. clean
