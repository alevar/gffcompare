cmake_minimum_required(VERSION 3.10)
project(GFFCompare)

# Define C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Include directories
set(GCLIB ${GCLIB} CACHE PATH "Path to GCLIB" FORCE)
if(NOT GCLIB)
    set(GCLIB "../gclib")
endif()
include_directories(${GCLIB})

# Set compiler flags
set(BASEFLAGS "-Wall -Wextra -D_REENTRANT -fno-exceptions -fno-rtti")
if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 8)
    set(BASEFLAGS "${BASEFLAGS} -Wno-class-memaccess")
endif()

set(CMAKE_CXX_FLAGS_DEBUG "${BASEFLAGS} -g -O0 -DDEBUG -D_DEBUG -DGDEBUG")
set(CMAKE_CXX_FLAGS_RELEASE "${BASEFLAGS} -O3 -DNDEBUG")

# Define source files
set(OBJS
    ${GCLIB}/GFastaIndex.cpp
    ${GCLIB}/GFaSeqGet.cpp
    ${GCLIB}/gff.cpp
    ${GCLIB}/gdna.cpp
    ${GCLIB}/codons.cpp
    ${GCLIB}/GBase.cpp
    ${GCLIB}/GStr.cpp
    ${GCLIB}/GArgs.cpp
)

add_library(gclib STATIC ${OBJS})

# Executable targets
add_executable(gffcompare gtf_tracking.cpp gffcompare.cpp)
target_link_libraries(gffcompare gclib)

add_executable(trmap trmap.cpp)
target_link_libraries(trmap gclib)

# Memcheck options
if(CMAKE_BUILD_TYPE STREQUAL "MemCheck" OR CMAKE_BUILD_TYPE STREQUAL "MemDebug")
    if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9)
        message(FATAL_ERROR "gcc version 4.9 or greater is required for MemCheck/MemDebug")
    endif()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-omit-frame-pointer -fsanitize=undefined -fsanitize=address")
    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 5)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=bounds -fsanitize=float-divide-by-zero -fsanitize=vptr")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=float-cast-overflow -fsanitize=object-size")
    endif()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-common -fstack-protector")
    target_link_libraries(gffcompare asan ubsan dl)
    target_link_libraries(trmap asan ubsan dl)
endif()

# Custom target to clone gclib if it doesn't exist
add_custom_target(clone_gclib
    COMMAND git clone https://github.com/gpertea/gclib.git ${GCLIB}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMENT "Cloning GCLIB..."
    VERBATIM
)

# Clean target
set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "core;core.*;gffcompare;gffcompare.exe;trmap;trmap.exe;*.o*")
