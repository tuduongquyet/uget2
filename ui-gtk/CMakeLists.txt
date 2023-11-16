cmake_minimum_required(VERSION 2.8)

if(CMAKE_VERSION VERSION_LESS "3.0")
    project(uget-gtk C CXX)
    set(PROJECT_VERSION ${UGET_VERSION})
else()
    cmake_policy(SET CMP0042 NEW)
    cmake_policy(SET CMP0048 NEW)
    project(uget-gtk VERSION ${UGET_VERSION} LANGUAGES C CXX)
endif()

include_directories(${CURL_INCLUDE_DIR}
    ${OPENSSL_INCLUDE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/../uglib
    ${CMAKE_CURRENT_SOURCE_DIR}/../uget
)

set(UGET_GTK_SOURCES
    UgtkUtil.c
    UgtkConfig.c
    UgtkSetting.c
    UgtkNodeList.c
    UgtkNodeTree.c
    UgtkNodeView.c
    UgtkTraveler.c
    UgtkSummary.c
    UgtkTrayIcon.c
    UgtkBanner.c
    UgtkSequence.c
    UgtkSelector.c
    UgtkProxyForm.c
    UgtkDownloadForm.c
    UgtkCategoryForm.c
    UgtkNodeDialog.c
    UgtkBatchDialog.c
    UgtkScheduleForm.c
    UgtkSettingForm.c
    UgtkSettingDialog.c
    UgtkConfirmDialog.c
    UgtkAboutDialog.c
    UgtkMenubar.c
    UgtkMenubar-ui.c
    UgtkApp.c
    UgtkApp-ui.c
    UgtkApp-callback.c
    UgtkApp-timeout.c
    UgtkApp-main.c
)

add_definitions(-DUG_DATADIR="${CMAKE_INSTALL_PREFIX}/share")
add_executable(uget-gtk ${UGET_GTK_SOURCES})

if(APPLE)
    set_source_files_properties(UgtkApp-ui.c
        PROPERTIES
        COMPILE_FLAGS "-x objective-c")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -framework Cocoa")
endif()

target_link_libraries(uget-gtk
    uget
    uglib
    ${GTKMACINTEGRATION3_LIBRARIES}
    ${GTK3_LIBRARIES}
    ${GSTREAMER_LIBRARIES}
    ${CURL_LIBRARIES}
    ${APPINDICATOR_LIBRARIES}
    ${OPENSSL_LIBRARIES}
    ${LIBNOTIFY_LIBRARIES}
    ${CMAKE_THREAD_LIBS_INIT}
    "-framework CoreFoundation"
)

# CPack
set(CPACK_SOURCE_GENERATOR "TGZ")
set(CPACK_SOURCE_PACKAGE_FILE_NAME ${PACKAGE_NAME}-${PACKAGE_VERSION})
set(CPACK_SOURCE_IGNORE_FILES "CMakeCache" "CMakeFiles" "CPackConfig" "CPackSourceConfig"
    "_CPack_Packages" "cmake_install" "install_manifest"
    "Makefile$" "test" "package-config" "misc/*"
    "src/task$" "src/libtask.a" "auto.h$"
    "/\\.gitignore" "/\\.git/" "swp$")
include(CPack)

set(PACKAGE_DATA_DIR "${CMAKE_INSTALL_PREFIX}/share")
set(LOCALEDIR "${CMAKE_INSTALL_PREFIX}/share/locale")

if(NOT MAN_ENTRY_PATH)
    set(MAN_ENTRY_PATH "${CMAKE_INSTALL_PREFIX}/share/man")
endif(NOT MAN_ENTRY_PATH)

if(NOT PIXMAPS_ENTRY_PATH)
    set(PIXMAPS_ENTRY_PATH "${CMAKE_INSTALL_PREFIX}/share/icons/hicolor")
endif(NOT PIXMAPS_ENTRY_PATH)

# pixmaps
set(icons_DIR "${PACKAGE_DATA_DIR}")

set(ugetpixmap_DATA "${CMAKE_CURRENT_SOURCE_DIR}/pixmaps/logo.png")
set(ugetpixmap_DIR "${PACKAGE_DATA_DIR}/pixmaps/uget")

# po
if(GETTEXT_FOUND)
    file(STRINGS ${CMAKE_SOURCE_DIR}/po/LINGUASX linguas)

    # The default build target (ALL) only creates .gmo files from .po files.
    foreach(lang ${linguas})
        if(NOT ${lang} MATCHES "#")
            gettext_process_po_files(${lang} ALL PO_FILES ${CMAKE_SOURCE_DIR}/po/${lang}.po)

            # Use custom install target instead of specifying the INSTALL_DESTINATION
            # option for gettext_process_po_files() because it gets naming of the
            # resulting .mo files wrong.
            install(FILES ${CMAKE_BINARY_DIR}/${lang}.gmo
                DESTINATION "${CMAKE_INSTALL_PREFIX}/share/locale/${lang}/LC_MESSAGES/" RENAME uget.mo)
        endif()
    endforeach()

    # Note: CMake currently (version 3.3) does not have any built-in support for
    # generating .pot files. Additionally, the way FindGettext works does not
    # allow to create targets for generating .po files because both
    # gettext_create_translations() and gettext_process_pot_file() always do
    # msgmerge+msgfmt and the msgfmt command creates a clashing rule with the one
    # produced by gettext_process_po_files().
    #
    # This means update-po functionality provided by the autotools build is not
    # currently available in the CMake build.
endif()

# sounds
set(sounds_DATA "${CMAKE_SOURCE_DIR}/sounds/notification.wav")
set(sounds_DIR "${PACKAGE_DATA_DIR}/sounds/uget")

# application
set(apps_DATA uget-gtk.desktop)
set(apps_DIR ${PACKAGE_DATA_DIR}/applications)

# Installation
# Executable
install(TARGETS uget-gtk DESTINATION "${CMAKE_INSTALL_PREFIX}/bin")
install(DIRECTORY "${CMAKE_SOURCE_DIR}/pixmaps/icons"
    DESTINATION "${icons_DIR}"
    FILES_MATCHING PATTERN "*.png")
install(FILES "${ugetpixmap_DATA}" DESTINATION "${ugetpixmap_DIR}")
install(FILES "${sounds_DATA}" DESTINATION "${sounds_DIR}")
install(FILES "${apps_DATA}" DESTINATION "${apps_DIR}")
