include_directories(${CMAKE_SOURCE_DIR}/)

# Manually specify the paths for utf8proc
set(UTF8PROC_INCLUDE_DIR /usr/local/include)
set(UTF8PROC_LIB /usr/local/lib/libutf8proc.a) # or libutf8proc.so for the shared library
set(OATPP_INCLUDE_DIR /usr/local/include)
set(OATPP_LIB /usr/local/lib/libutf8proc.a) # or libutf8proc.so for the shared library

# Add the include directory
include_directories(${UTF8PROC_INCLUDE_DIR} ${OATPP_INCLUDE_DIR})

# If utf8proc is found, include its directories and link it
if(UTF8PROC_LIB AND UTF8PROC_INCLUDE_DIR)
    include_directories(${UTF8PROC_INCLUDE_DIR})
    message(STATUS "Found utf8proc library: ${UTF8PROC_LIB}")
    message(STATUS "Found utf8proc include directory: ${UTF8PROC_INCLUDE_DIR}")
else()
    message(FATAL_ERROR "utf8proc library not found")
endif()

if (MSVC)
    if (BERT_AVX2)
        add_compile_options(/arch:AVX2)
    elseif (BERT_AVX)
        add_compile_options(/arch:AVX)
    endif()
endif()

if(WIN32)
add_definitions(-DNOMINMAX=1)
endif()


add_executable(bert-main bert-main.cpp)
if(NOT WIN32)
    target_link_libraries(bert-main PRIVATE bertencoder bertbase tokenization ggml ${UTF8PROC_LIB})
else()
    target_link_libraries(bert-main PRIVATE bertencoder bertbase tokenization ggml ${UTF8PROC_LIB} ws2_32)
endif()

# bert-rest.cpp does not compile on ARM hence removing it from CMakelist.txt because of OATPP linking errors
# add_executable(bert-rest bert-rest.cpp)
# if(NOT WIN32)
#	target_link_libraries(bert-rest PRIVATE bertencoder bertbase tokenization ggml ${UTF8PROC_LIB} ${OATPP_LIB})
# else()
#	target_link_libraries(bert-rest PRIVATE bertencoder bertbase tokenization ggml ${UTF8PROC_LIB} ${OATPP_LIB} ws2_32)
# endif()
