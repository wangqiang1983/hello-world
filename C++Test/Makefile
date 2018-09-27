
OBJS=$(shell find . -regextype posix-egrep -regex ".*.cpp")
all:$(OBJS)
	g++ --std=c++0x $(OBJS) -o test
