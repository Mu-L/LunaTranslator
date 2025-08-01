
include(protobuf.cmake)
include(FetchContent)

FetchContent_Declare(wechat_ocr
    GIT_REPOSITORY https://github.com/swigger/wechat-ocr
    GIT_TAG master
    GIT_SHALLOW TRUE
)
FetchContent_Populate(wechat_ocr)


function(FIX_CPP17 FILE_PATH)

    file(READ ${FILE_PATH} FILE_CONTENT)

    string(REPLACE "std::span" "std::basic_string_view" FILE_CONTENT "${FILE_CONTENT}")
    string(REPLACE "requires" "//requires" FILE_CONTENT "${FILE_CONTENT}")
    string(REPLACE "#include <span>" "//#include <span>" FILE_CONTENT "${FILE_CONTENT}")
    string(REPLACE "__declspec(dllexport)" "" FILE_CONTENT "${FILE_CONTENT}")

    file(WRITE ${FILE_PATH} "${FILE_CONTENT}")
endfunction()

FIX_CPP17("${wechat_ocr_SOURCE_DIR}/src/mojocall.cpp")
FIX_CPP17("${wechat_ocr_SOURCE_DIR}/src/mojocall.h")
FIX_CPP17("${wechat_ocr_SOURCE_DIR}/src/wechatocr.cpp")
FIX_CPP17("${wechat_ocr_SOURCE_DIR}/src/wechatocr.h")
FIX_CPP17("${wechat_ocr_SOURCE_DIR}/vs.proj/stdafx.h")

set(wcocr_src ${wechat_ocr_SOURCE_DIR}/src/mojocall.cpp ${wechat_ocr_SOURCE_DIR}/src/ocr_common.pb.cc  ${wechat_ocr_SOURCE_DIR}/src/ocr_wx3.pb.cc ${wechat_ocr_SOURCE_DIR}/src/ocr_wx4.pb.cc ${wechat_ocr_SOURCE_DIR}/src/wechatocr.cpp)


add_library(wcocrpch ${wechat_ocr_SOURCE_DIR}/vs.proj/stdafx.cpp)
target_precompile_headers(wcocrpch PUBLIC ${wechat_ocr_SOURCE_DIR}/vs.proj/stdafx.h)
add_library(wcocr_1 wcocr_1.cpp ${wcocr_src})
target_include_directories(wcocr_1 PRIVATE ${wechat_ocr_SOURCE_DIR}/vs.proj PRIVATE ${wechat_ocr_SOURCE_DIR}/src)
target_precompile_headers(wcocr_1 REUSE_FROM wcocrpch)


target_link_libraries(wcocr_1 PRIVATE libprotobuf-lite)


file(GLOB glensproto proto/*.proto)
PROTOBUF_GENERATE_CPP(glensprotocpp ${CMAKE_CURRENT_BINARY_DIR} ${wechat_ocr_SOURCE_DIR}/spt/protoc.exe ${CMAKE_CURRENT_LIST_DIR}/proto "${glensproto}")

add_library(glensproto glens_1.cpp "${glensprotocpp}")
target_include_directories(glensproto PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(glensproto PRIVATE libprotobuf-lite)


add_library(needprotobuf INTERFACE)
target_sources(needprotobuf INTERFACE wcocr.cpp)
target_sources(needprotobuf INTERFACE glens.cpp)
target_link_libraries(needprotobuf INTERFACE wcocr_1)
target_link_libraries(needprotobuf INTERFACE glensproto)