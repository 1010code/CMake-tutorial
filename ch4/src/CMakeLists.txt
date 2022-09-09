# INCLUDE_DIRECTORIES(/usr/local/include/hello)
# LINK_DIRECTORIES(/usr/local/lib)
# ADD_EXECUTABLE(hello main.cpp)
# TARGET_LINK_LIBRARIES(hello libhello.dylib)


INCLUDE_DIRECTORIES(/usr/local/include/hello)
ADD_EXECUTABLE(hello main.cpp)

FIND_LIBRARY(HELLO_LIBRARY hello HINTS /usr/local/lib)
TARGET_LINK_LIBRARIES(hello PUBLIC ${HELLO_LIBRARY})