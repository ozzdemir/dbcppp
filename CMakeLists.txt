cmake_minimum_required(VERSION 3.10)

project("libdbcppp" VERSION 0.1.0)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_STATIC_LIBRARY_PREFIX "")
set(CMAKE_SHARED_LIBRARY_PREFIX "")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

if (!${Boost_USE_STATIC_LIBS})
	add_definitions(-DBOOST_LOG_DYN_LINK)
endif()

#find_package(Boost 1.72.0 REQUIRED COMPONENTS program_options)
find_package(LibXml2 REQUIRED)
#find_package(LLVM REQUIRED CONFIG)

#message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")
#message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")
#add_definitions(${LLVM_DEFINITIONS})
# Find the libraries that correspond to the LLVM components
# that we wish to use
#llvm_map_components_to_libnames(llvm_libs support core irreader mcjit jitlink native nativecodegen orcjit)

# Download Boost 1.72
set(BOOST_1_72_COMPRESSED_FILENAME "${PROJECT_SOURCE_DIR}/third_party/boost_1_72_0.tar.bz2")
set(BOOST_1_72_UNCOMPRESS_DIR "${PROJECT_SOURCE_DIR}/third_party/")
set(BOOST_1_72_URL "https://sourceforge.net/projects/boost/files/boost/1.72.0/boost_1_72_0.tar.bz2/download")

if(NOT EXISTS ${BOOST_1_72_COMPRESSED_FILENAME})
  message("Downloading Boost 1.72.")
  file(DOWNLOAD ${BOOST_1_72_URL} ${BOOST_1_72_COMPRESSED_FILENAME}
       EXPECTED_MD5 cb40943d2a2cb8ce08d42bc48b0f84f0
       SHOW_PROGRESS)
endif()

add_custom_target(unzip_boost_1_72 ALL
    COMMAND if [ ! -f '${BOOST_1_72_UNCOMPRESS_DIR}/boost_1_72_0/b2' ]\; then tar -xvjf ${BOOST_1_72_COMPRESSED_FILENAME} --directory ${BOOST_1_72_UNCOMPRESS_DIR}\; cd ${BOOST_1_72_UNCOMPRESS_DIR}/boost_1_72_0/\; ./bootstrap.sh\; fi\;
)

if (Boost_FOUND)
	include_directories(${Boost_INCLUDE_DIRS})
	link_directories(${Boost_LIBRARY_DIRS})
endif()

if (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
	add_definitions("/bigobj")
endif()

include_directories(${LIBXML2_INCLUDE_DIR})
include_directories(${LLVM_INCLUDE_DIRS})
include_directories("third_party")
include_directories("include")
include_directories("third_party/libxmlmm/include")
include_directories("third_party/boost_1_72_0")

#add_subdirectory(src) dbcppp fails due to gcc version, so build only the library, btw, isn't the main better suited as an example?
add_subdirectory(src/libdbcppp)

if(BUILD_TESTING)
    add_subdirectory(tests)
    add_subdirectory(examples)
endif()

set(CMAKE_STATIC_LIBRARY_PREFIX "")
set(CMAKE_SHARED_LIBRARY_PREFIX "")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)


add_library("libxmlmm" SHARED "")

target_link_libraries("libxmlmm" ${LIBXML2_LIBRARIES})

file(GLOB header
    "third_party/libxmlmm/include/*.h"
)
file(GLOB src
    "third_party/libxmlmm/src/*.cpp"
)

target_sources("libxmlmm"
    PRIVATE ${header}
    PUBLIC ${header_interface}
    PRIVATE ${src}
)

# install
install(
    TARGETS "libxmlmm"
    DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(
    DIRECTORY "third_party/libxmlmm/include"
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libxmlmm
    FILES_MATCHING PATTERN "*.h")
