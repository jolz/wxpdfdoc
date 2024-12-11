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
    URL "https://github.com/wxWidgets/wxWidgets/releases/download/v3.2.6/wxWidgets-3.2.6.7z"
    URL_HASH SHA1=f377748dd4b571aaf68f34a40c7e4f390680f16c # 3.2.6
    DOWNLOAD_EXTRACT_TIMESTAMP ON # avoid warning
    # GIT_REPOSITORY https://github.com/wxWidgets/wxWidgets.git GIT_SHALLOW ON
    # GIT_TAG v3.2.6
  )
  FetchContent_MakeAvailable(wxWidgets)

endif()

# library libwxpdfdoc
file(GLOB source_list "src/*.cpp" "src/*.inc" "src/crypto/*.cpp")
add_library(wxpdfdoc SHARED ${source_list})

target_include_directories(wxpdfdoc PUBLIC SYSTEM include/)
target_include_directories(wxpdfdoc PRIVATE src/crypto/)
target_link_libraries(wxpdfdoc wx::core wx::xml)
add_library(wxpdfdoc::wxpdfdoc ALIAS wxpdfdoc)

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

# utility makefont
add_executable(wxpdfdoc_makefont makefont/makefont.cpp makefont/makefont.rc)
target_link_directories(wxpdfdoc_makefont PRIVATE samples/makefont)
target_link_libraries(wxpdfdoc_makefont wxpdfdoc::wxpdfdoc)

# utility showfont
add_executable(wxpdfdoc_showfont showfont/showfont.cpp showfont/showfont.rc)
target_link_directories(wxpdfdoc_showfont PRIVATE samples/showfont)
target_link_libraries(wxpdfdoc_showfont wxpdfdoc::wxpdfdoc)
