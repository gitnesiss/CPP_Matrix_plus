CC                =g++
CFLAGS            =-Werror -Wextra -Wall -std=c++17
OS_NAME           =$(shell uname)
SRC_DIR           =s21_matrix_oop/
BUILD_DIR         =build/
BUILD_LIB         =build/lib/
BUILD_DOC         =build/doc/
BUILD_ARCHIVE     =build/archive/
TEST_DIR          =s21_matrix_oop/tests/
TEMP_DIR          =s21_matrix_oop/tests/temp/
REPORT_DIR_GCOV   =s21_matrix_oop/tests/report_gcov/

ifeq ($(OS_NAME), Linux)
	OPEN        =xdg-open
	TEST_FLAGS  =--coverage $(shell pkg-config --libs check)
	LEAK_CHECK  =valgrind --trace-children=yes --track-fds=yes \
	--track-origins=yes --leak-check=full --show-leak-kinds=all \
	--log-file="report.txt" ./$(TEST_DIR)tests.out
	CFLAGS_LIBS =-lm -lsubunit -lpthread -lrt
else
	OPEN        =open -a "Google Chrome"
	TEST_FLAGS  =--coverage -lm -lcheck
	LEAK_CHECK  =CK_FORK=no leaks --atExit -- ./$(TEST_DIR)tests.out
	CFLAGS_LIBS =-lm -L /usr/local/lib #-lcheck
endif

all: s21_matrix_oop.a

main:
	$(CC) $(CFLAGS) s21_matrix_oop/s21_matrix_oop.cc
	./a.out

debug:
	$(CC) $(CFLAGS) s21_matrix_oop/s21_matrix_oop.cc -g
	./a.out

dist:
	mkdir -p $(BUILD_ARCHIVE)
	rm -rf $(BUILD_ARCHIVE)s21_matrix_oop.tar.gz
	tar -cvf $(BUILD_ARCHIVE)s21_matrix_oop.tar.gz $(SRC_DIR)

dvi:
	mkdir -p $(BUILD_DOC)
	makeinfo -o $(BUILD_DOC) --html --no-warn --force \
	$(SRC_DIR)s21_matrix_oop_info.texi
	$(OPEN) $(BUILD_DOC)index.html

s21_matrix_oop.a:
	mkdir -p $(BUILD_LIB)
	$(CC) $(CFLAGS_LIBS) -c $(SRC_DIR)*.cc
	ar rcs $(BUILD_LIB)$@ *.o
	ranlib $(BUILD_LIB)$@
	rm -f *.o

test: clean
	@$(CC) $(FLAGS) $(SRC_DIR)*.cc $(TEST_DIR)tests.cc $(TEST_FLAGS) -o \
	$(TEST_DIR)tests.out -lgtest -pthread
	@./$(TEST_DIR)/tests.out
	@mkdir -p $(TEMP_DIR)
ifeq ($(OS_NAME), Linux)
	@mv $(TEST_DIR)*.gcda $(TEMP_DIR)
	@mv $(TEST_DIR)*.gcno $(TEMP_DIR)
else
	@mv *.gcda $(TEMP_DIR)
	@mv *.gcno $(TEMP_DIR)
endif

gcov_report: test
	@mkdir -p $(REPORT_DIR_GCOV)
	@mv $(TEMP_DIR)*.gcda $(REPORT_DIR_GCOV)
	@mv $(TEMP_DIR)*.gcno $(REPORT_DIR_GCOV)
	@rm -fR $(TEMP_DIR)
	@gcovr -r $(REPORT_DIR_GCOV) -r . --html-details -o test_report_gcov.html
	@mv test_report_gcov.* $(REPORT_DIR_GCOV)
	open $(REPORT_DIR_GCOV)test_report_gcov.html

clean: clean_build clean_dest clean_dvi clean_deb clean_lib clean_test
	@rm -f *.out

clean_build:
	@rm -fR $(BUILD_DIR)

clean_dest:
	@rm -fR $(BUILD_ARCHIVE)

clean_dvi:
	@rm -fR $(BUILD_DOC)

clean_deb:
	@rm -fR *.dSYM

clean_lib:
	@rm -f *.a

clean_test:
	@rm -rf *.gcno *.gcda
	@rm -rf $(TEST_DIR)/*.gcno $(TEST_DIR)/*.gcda $(TEST_DIR)/tests.out
	@rm -Rf $(TEMP_DIR)
	@rm -Rf $(TEST_DIR)/report_gcov

check:
	cp ../materials/linters/.clang-format ./
	clang-format -i $(SRC_DIR)*.cc
	clang-format -i $(SRC_DIR)*.h
	clang-format -n $(SRC_DIR)*.cc
	clang-format -n $(SRC_DIR)*.h
	rm -f .clang-format

leaks: test
	$(LEAK_CHECK)

cppcheck:
	cppcheck --language=c++ $(SRC_DIR)*.cc $(SRC_DIR)*.h
