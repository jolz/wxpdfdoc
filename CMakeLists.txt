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
SET(ZINT_STATIC ON)
add_subdirectory(thirdparty/zint/backend)
set_property(TARGET zint-static PROPERTY POSITION_INDEPENDENT_CODE ON)

# library libwxpdfdoc
file(GLOB source_list "src/*.cpp" "src/*.inc" "src/crypto/*.cpp")
add_library(wxpdfdoc SHARED ${source_list})
target_include_directories(wxpdfdoc PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/zint/backend/fonts)

target_include_directories(wxpdfdoc PUBLIC SYSTEM include/)
target_include_directories(wxpdfdoc PRIVATE src/crypto/)
target_link_libraries(wxpdfdoc wx::core wx::xml zint-static)
add_library(wxpdfdoc::wxpdfdoc ALIAS wxpdfdoc)

if (WXPDFDOC_samples)
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
