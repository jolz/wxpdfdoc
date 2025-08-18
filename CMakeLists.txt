cmake_minimum_required(VERSION 3.14 FATAL_ERROR)

project(wxpdfdoc)

if(NOT WXPDFDOC_use_external_wxwidgets)
  # top-level project? then include wxwidgets

  include(FetchContent)

  set(CMAKE_CXX_STANDARD 14)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
  set(wxBUILD_SHARED OFF)

  message(STATUS "Fetching wxWidgets...")

  FetchContent_Declare(
    wxWidgets
    URL "https://github.com/wxWidgets/wxWidgets/releases/download/v3.3.1/wxWidgets-3.3.1.7z"
    URL_HASH SHA1=5e814eba0cb4bdb30b81d6ede28b00ee1384a3ce # 3.3.1
    DOWNLOAD_EXTRACT_TIMESTAMP ON # avoid warning
    # GIT_REPOSITORY https://github.com/wxWidgets/wxWidgets.git GIT_SHALLOW ON
    # GIT_TAG v3.3.1    
  )
  FetchContent_MakeAvailable(wxWidgets)

endif()

include(CheckCCompilerFlag)
set(ZINT_STATIC ON)
add_subdirectory(thirdparty/zint/backend)
set_property(TARGET zint-static PROPERTY POSITION_INDEPENDENT_CODE ON)

set(WOFF2_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/woff2)
file(GLOB woff2_files
    thirdparty/woff2/src/woff2_dec.cc
    thirdparty/woff2/src/woff2_common.cc
    thirdparty/woff2/src/woff2_out.cc
    thirdparty/woff2/src/table_tags.cc
    thirdparty/woff2/src/variable_length.cc
    #thirdparty/woff2/src/buffer.h
    #thirdparty/woff2/src/port.h
    #thirdparty/woff2/src/round.h
    #thirdparty/woff2/src/store_bytes.h
    #thirdparty/woff2/src/table_tags.h
    #thirdparty/woff2/src/variable_length.h
    #thirdparty/woff2/src/woff2_common.h
    #thirdparty/woff2/brotli/dec/*.c
    thirdparty/woff2/brotli/dec/bit_reader.c  thirdparty/woff2/brotli/dec/decode.c  thirdparty/woff2/brotli/dec/huffman.c  thirdparty/woff2/brotli/dec/state.c
    #thirdparty/woff2/brotli/dec/*.h
    #thirdparty/woff2/brotli/common/*.c
    thirdparty/woff2/brotli/common/constants.c  thirdparty/woff2/brotli/common/dictionary.c  thirdparty/woff2/brotli/common/shared_dictionary.c
    thirdparty/woff2/brotli/common/context.c    thirdparty/woff2/brotli/common/platform.c    thirdparty/woff2/brotli/common/transform.c
    #thirdparty/woff2/brotli/common/*.h
    )
add_library(wxpdfdoc-woff2 STATIC ${woff2_files})
target_include_directories(wxpdfdoc-woff2 PUBLIC ${WOFF2_SRC_DIR}/include)
set_property(TARGET wxpdfdoc-woff2 PROPERTY POSITION_INDEPENDENT_CODE ON)

# library libwxpdfdoc
# src/*.cpp src/*.inc src/crypto/*.cpp src/woff/*.cpp
SET (source_list src/crypto/random.cpp        src/pdfcmapdata.inc         src/pdffontdatacore.cpp        src/pdffontparsertype1.cpp     src/pdfocg.cpp
src/crypto/saslprep.cpp      src/pdfcodepagedata.inc     src/pdffontdata.cpp            src/pdffontsubsetcff.cpp       src/pdfparser.cpp
src/crypto/sha256.cpp        src/pdfcolour.cpp           src/pdffontdataopentype.cpp    src/pdffontsubsettruetype.cpp  src/pdfpattern.cpp
src/crypto/sha384.cpp        src/pdfcolourdata.inc       src/pdffontdatatruetype.cpp    src/pdffontvolt.cpp            src/pdfprint.cpp
src/crypto/sha512.cpp        src/pdfcorefontdata.inc     src/pdffontdatatype0.cpp       src/pdfform.cpp                src/pdfrijndael.cpp
src/crypto/unicode_norm.cpp  src/pdfcorefontkerning.inc  src/pdffontdatatype1.cpp       src/pdfglyphnames.inc          src/pdftemplate.cpp
src/pdfannotation.cpp        src/pdfdc.cpp               src/pdffontdescription.cpp     src/pdfgradient.cpp            src/pdfutility.cpp
src/pdfbarcode.cpp           src/pdfdecode.cpp           src/pdffontdetails.cpp         src/pdfgraphics.cpp            src/pdfxml.cpp
src/pdfbarcodezint.cpp       src/pdfdocument.cpp         src/pdffontextended.cpp        src/pdfimage.cpp               src/woff/woff2converter.cpp
src/pdfcffdecoder.cpp        src/pdfencoding.cpp         src/pdffontmanager.cpp         src/pdfkernel.cpp              src/woff/woffconverter.cpp
src/pdfcffindex.cpp          src/pdfencrypt.cpp          src/pdffontparser.cpp          src/pdflayer.cpp
src/pdfcjkfontdata.inc       src/pdffont.cpp             src/pdffontparsertruetype.cpp  src/pdfobjects.cpp)
add_library(wxpdfdoc SHARED ${source_list})
target_include_directories(
  wxpdfdoc PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/zint/backend/fonts)

target_include_directories(wxpdfdoc PUBLIC include/)
target_include_directories(wxpdfdoc PRIVATE src/crypto/)
target_link_libraries(wxpdfdoc PUBLIC wx::core wx::xml)
target_link_libraries(wxpdfdoc PRIVATE zint-static wxpdfdoc-woff2)
add_library(wxpdfdoc::wxpdfdoc ALIAS wxpdfdoc)

if(WXPDFDOC_samples)
  # sample minimal
  file(GLOB wxpdfdoc_minimal_source_list "samples/minimal/*.cpp"
       "samples/minimal/*.rc")
  add_executable(wxpdfdoc_minimal ${wxpdfdoc_minimal_source_list})

  target_link_directories(wxpdfdoc_minimal PRIVATE samples/minimal)
  target_link_libraries(wxpdfdoc_minimal wxpdfdoc::wxpdfdoc)

  # sample pdfdc
  add_executable(wxpdfdoc_pdfdc samples/pdfdc/printing.cpp
                                samples/pdfdc/printing.rc)
  target_link_directories(wxpdfdoc_pdfdc PRIVATE samples/pdfdc)
  target_link_libraries(wxpdfdoc_pdfdc wxpdfdoc::wxpdfdoc wx::adv wx::html
                        wx::richtext)
endif()

# utility makefont
add_executable(wxpdfdoc_makefont makefont/makefont.cpp makefont/makefont.rc)
target_link_directories(wxpdfdoc_makefont PRIVATE samples/makefont)
target_link_libraries(wxpdfdoc_makefont wxpdfdoc::wxpdfdoc)

# utility showfont
add_executable(wxpdfdoc_showfont showfont/showfont.cpp showfont/showfont.rc)
target_link_directories(wxpdfdoc_showfont PRIVATE samples/showfont)
target_link_libraries(wxpdfdoc_showfont wxpdfdoc::wxpdfdoc)
