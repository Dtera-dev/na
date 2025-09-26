cmake_minimum_required(VERSION 3.16)

# ==========================================================================
# noandishan board cmake build file
#
# Most important configuration variables:
#   CMAKE_INSTALL_PREFIX   
#       Path to install prefix, defaults to /usr/local (see cmake documentation)
#       Typically set to /usr or /opt
#   QT_VERSION
#       Qt Version to use
#       Set to empty, 5 or 6, defaults to auto-selection with preference to 5
#
# Typical invocation
#   cmake -S <srcdir> -B <builddir> -DCMAKE_INSTALL_PREFIX:PATH=/usr
#       or
#   cmake -S <srcdir> -B <builddir> -DCMAKE_INSTALL_PREFIX:PATH=/opt
#
# Build
#   cd <builddir>
#   cmake --build . [-j<n>]
#
# Package
#    cpack -G <DEB|RPM>
# ==========================================================================

# ==========================================================================
# Basic project information
# ==========================================================================

include(cmake/Version.cmake)

project(noandishan board
    VERSION ${VERSION_NUMBER}
    DESCRIPTION "noandishan board is an open source cross-platform interactive white board application designed primarily for use in schools. It was originally forked from Open-Sankoré, which was itself based on Uniboard."
    LANGUAGES CXX
)

add_compile_definitions(UBVERSION="${VERSION}")


# ==========================================================================
# Configuration options
# ==========================================================================

set(QT_VERSION "" CACHE STRING "Qt major version number to use - empty, 5 or 6")

# Internal setting
set(QAPPLICATION_CLASS QApplication CACHE STRING "Inheritance class for SingleApplication - do not change")


# ==========================================================================
# Non-source file locations
# ==========================================================================

set(noandishan board_QRC_FILE      resources/noandishan board.qrc)

set(noandishan board_FORMS_DIR     resources/forms src/web/simplebrowser)
set(noandishan board_TS_DIR        resources/i18n)
set(noandishan board_FONT_DIR      resources/customizations resources/fonts)
set(noandishan board_ETC_DIR       resources/etc)
set(noandishan board_LIBRARY_DIR   resources/library)
set(noandishan board_STARTUP_HINTS resources/startupHints)

set(noandishan board_ETC_FILE      ${noandishan board_ETC_DIR}/noandishan board.config
                            ${noandishan board_ETC_DIR}/noandishan board.css
                            ${noandishan board_ETC_DIR}/widgets.md5sum)
set(noandishan board_TEMPLATE_FILE ${noandishan board_ETC_DIR}/asyncAPI.js
                            ${noandishan board_ETC_DIR}/intranet-podcast-metadata.template
                            ${noandishan board_ETC_DIR}/npapi-wrapper.config.xml)

set(noandishan board_ICON_FILE     resources/images/ch.noandishan board.noandishan board.svg)
set(noandishan board_DESKTOP_FILE  resources/linux/ch.noandishan board.noandishan board.desktop)
set(noandishan board_MIMETYPE_FILE resources/linux/ch.noandishan board.noandishan board-ubz.xml)
set(noandishan board_MIMEICON_FILE resources/linux/ch.noandishan board.application-ubz.svg)


# ==========================================================================
# Basic compiler settings
# ==========================================================================

set(CMAKE_CXX_STANDARD 17 CACHE STRING "C++ standard to use - defaults to C++17")
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)

list(APPEND CMAKE_AUTOUIC_SEARCH_PATHS 
    ${noandishan board_FORMS_DIR}
)

message(STATUS "Using C++" ${CMAKE_CXX_STANDARD})

# OpenMP support
include(FindOpenMP)

if(OPENMP_FOUND)
    string(APPEND CMAKE_CXX_FLAGS " " ${OpenMP_CXX_FLAGS})
endif()


# ==========================================================================
# Target and dependencies
#
# Note: the executable is using lowercase on all platforms
# ==========================================================================

add_executable(${PROJECT_NAME})

# manage dependencies
include(cmake/DependencyQt.cmake)
include(cmake/DependencyOpenSSL.cmake)
include(cmake/DependencyPoppler.cmake)
include(cmake/DependencyZlib.cmake)
include(cmake/DependencyQuaZip.cmake)
include(cmake/DependencyFFmpeg.cmake)
include(cmake/DependencyX11.cmake)

# add source path to include directories
target_include_directories(${PROJECT_NAME} PRIVATE
    src
)

# add sources from subdirectories
add_subdirectory(src/adaptors)
add_subdirectory(src/api)
add_subdirectory(src/board)
add_subdirectory(src/core)
add_subdirectory(src/desktop)
add_subdirectory(src/document)
add_subdirectory(src/domain)
add_subdirectory(src/frameworks)
add_subdirectory(src/globals)
add_subdirectory(src/gui)
add_subdirectory(src/network)
add_subdirectory(src/pdf)
add_subdirectory(src/pdf-merger)
add_subdirectory(src/podcast)
add_subdirectory(src/singleapplication)
add_subdirectory(src/tools)
add_subdirectory(src/web)
add_subdirectory(plugins/cffadaptor/src)
add_subdirectory(resources/forms)

# statically link singleapplication
target_link_libraries(${PROJECT_NAME}
    SingleApplication::SingleApplication
)


# ==========================================================================
# Resources
# ==========================================================================

if(Qt5_VERSION AND Qt5_VERSION VERSION_LESS "5.15")
    qt5_add_resources(noandishan board_RESOURCES ${noandishan board_QRC_FILE})
else()
    qt_add_resources(noandishan board_RESOURCES ${noandishan board_QRC_FILE})
endif()

target_sources(${PROJECT_NAME} PRIVATE ${noandishan board_RESOURCES})


# ==========================================================================
# Translations
# ==========================================================================

file(GLOB noandishan board_TS_FILES ${noandishan board_TS_DIR}/*.ts)
set_source_files_properties(${noandishan board_TS_FILES} PROPERTIES OUTPUT_LOCATION ${PROJECT_BINARY_DIR}/i18n)

if(Qt5_VERSION AND Qt5_VERSION VERSION_LESS "5.15")
    qt5_add_translation(QM_FILES ${noandishan board_TS_FILES})
else()
    qt_add_translation(QM_FILES ${noandishan board_TS_FILES})
endif()

target_sources(${PROJECT_NAME} PRIVATE ${QM_FILES} ${noandishan board_TS_FILES})


# ==========================================================================
# Installation
# ==========================================================================

include(GNUInstallDirs)

# set relative install paths
if(CMAKE_INSTALL_PREFIX STREQUAL "/opt")
    set(noandishan board_INSTALL_SYSCONFDIR ${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}/etc)
    set(noandishan board_INSTALL_DATADIR ${PROJECT_NAME})
    set(CMAKE_INSTALL_BINDIR "/usr/bin")
    set(CMAKE_INSTALL_DATAROOTDIR "/usr/share")
else()
    set(SYSCONF_INSTALL_DIR ${CMAKE_INSTALL_FULL_SYSCONFDIR} CACHE PATH
        "Install directory for system-wide configuration files")
    set(noandishan board_INSTALL_SYSCONFDIR ${SYSCONF_INSTALL_DIR}/${PROJECT_NAME})
    set(noandishan board_INSTALL_DATADIR ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME})
endif()

# set absolute install paths
set(noandishan board_ETC_PREFIX "${noandishan board_INSTALL_SYSCONFDIR}")
set(noandishan board_APP_PREFIX "${CMAKE_INSTALL_PREFIX}/${noandishan board_INSTALL_DATADIR}")
set(noandishan board_TPL_PREFIX "${noandishan board_APP_PREFIX}/template")

message(STATUS "noandishan board ETC_PREFIX is " ${noandishan board_ETC_PREFIX})
message(STATUS "noandishan board APP_PREFIX is " ${noandishan board_APP_PREFIX})
message(STATUS "noandishan board TPL_PREFIX is " ${noandishan board_TPL_PREFIX})

target_compile_definitions(${PROJECT_NAME} PRIVATE
    noandishan board_FHS_LAYOUT
    noandishan board_ETC_PREFIX=${noandishan board_ETC_PREFIX}
    noandishan board_APP_PREFIX=${noandishan board_APP_PREFIX}
    noandishan board_TPL_PREFIX=${noandishan board_TPL_PREFIX}
)

# install files
install(TARGETS   ${PROJECT_NAME}               RUNTIME)
install(DIRECTORY ${noandishan board_FONT_DIR}         DESTINATION ${noandishan board_INSTALL_DATADIR})
install(DIRECTORY ${noandishan board_LIBRARY_DIR}      DESTINATION ${noandishan board_INSTALL_DATADIR})
install(DIRECTORY ${noandishan board_STARTUP_HINTS}    DESTINATION ${noandishan board_INSTALL_DATADIR})
install(DIRECTORY ${PROJECT_BINARY_DIR}/i18n    DESTINATION ${noandishan board_INSTALL_DATADIR})
install(FILES     ${noandishan board_ETC_FILE}         DESTINATION ${noandishan board_INSTALL_SYSCONFDIR})
install(FILES     ${noandishan board_TEMPLATE_FILE}    DESTINATION ${noandishan board_INSTALL_DATADIR}/template)
install(FILES     ${noandishan board_ICON_FILE}        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/icons/hicolor/scalable/apps)
install(FILES     ${noandishan board_DESKTOP_FILE}     DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/applications)
install(FILES     ${noandishan board_MIMETYPE_FILE}    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/mime/packages)
install(FILES     ${noandishan board_MIMEICON_FILE}    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/icons/hicolor/scalable/mimetypes)


# ==========================================================================
# Packaging
# ==========================================================================

include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_PACKAGE_VERSION "${VERSION}")
set(CPACK_PACKAGE_CONTACT "The noandishan board team")
set(CPACK_STRIP_FILES TRUE)

# Debian specific settings
set(CPACK_DEBIAN_FILE_NAME "DEB-DEFAULT")
set(CPACK_DEBIAN_PACKAGE_SUGGESTS onboard)

# RPM specific settings
set(CPACK_RPM_FILE_NAME "RPM-DEFAULT")
set(CPACK_RPM_PACKAGE_SUGGESTS onboard)

# create packager
include(CPack)
