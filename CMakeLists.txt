cmake_minimum_required(VERSION 3.1 FATAL_ERROR)

set(PROJECT_VERSION "5.21.80") # Handled by release scripts
project(KWin VERSION ${PROJECT_VERSION})

set(QT_MIN_VERSION "5.15.0")
set(KF5_MIN_VERSION "5.78")

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules ${CMAKE_MODULE_PATH})

find_package(ECM 5.38 REQUIRED NO_MODULE)

include(FeatureSummary)
include(WriteBasicConfigVersionFile)
include(GenerateExportHeader)

# where to look first for cmake modules, before ${CMAKE_ROOT}/Modules/ is checked
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${ECM_MODULE_PATH} )
set(CMAKE_PREFIX_PATH "/opt/libhybris/")
find_package(Qt5 ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS
    Concurrent
    Core
    DBus
    Quick
    QuickWidgets
    Script
    UiTools
    Widgets
    X11Extras
)

find_package(Qt5Test ${QT_MIN_VERSION} CONFIG QUIET)
set_package_properties(Qt5Test PROPERTIES
    PURPOSE "Required for tests"
    TYPE OPTIONAL
)
add_feature_info("Qt5Test" Qt5Test_FOUND "Required for building tests")
if (NOT Qt5Test_FOUND)
    set(BUILD_TESTING OFF CACHE BOOL "Build the testing tree.")
endif()

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(KDEClangFormat)

include(ECMInstallIcons)
include(ECMOptionalAddSubdirectory)
include(ECMConfiguredInstall)

add_definitions(-DQT_DISABLE_DEPRECATED_BEFORE=0 -DQT_USE_QSTRINGBUILDER -DQT_NO_URL_CAST_FROM_STRING)

# Prevent EGL headers from including platform headers, in particular Xlib.h.
add_definitions(-DMESA_EGL_NO_X11_HEADERS)
add_definitions(-DEGL_NO_X11)
add_definitions(-DEGL_NO_PLATFORM_SPECIFIC_TYPES)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# required frameworks by Core
find_package(KF5 ${KF5_MIN_VERSION} REQUIRED COMPONENTS
    Config
    ConfigWidgets
    CoreAddons
    Crash
    GlobalAccel
    I18n
    IconThemes
    IdleTime
    Notifications
    Package
    Plasma
    Wayland
    WidgetsAddons
    WindowSystem
)
# required frameworks by config modules
find_package(KF5 ${KF5_MIN_VERSION} REQUIRED COMPONENTS
    Completion
    Declarative
    KCMUtils
    KIO
    NewStuff
    Service
    TextWidgets
    XmlGui
)

find_package(Threads)
set_package_properties(Threads PROPERTIES
    PURPOSE "Needed for VirtualTerminal support in KWin Wayland"
    TYPE REQUIRED
)

# optional frameworks
find_package(KF5Activities ${KF5_MIN_VERSION} CONFIG)
set_package_properties(KF5Activities PROPERTIES
    PURPOSE "Enable building of KWin with kactivities support"
    TYPE OPTIONAL
)
add_feature_info("KF5Activities" KF5Activities_FOUND "Enable building of KWin with kactivities support")

find_package(KF5DocTools ${KF5_MIN_VERSION} CONFIG)
set_package_properties(KF5DocTools PROPERTIES
    PURPOSE "Enable building documentation"
    TYPE OPTIONAL
)
add_feature_info("KF5DocTools" KF5DocTools_FOUND "Enable building documentation")

find_package(KF5Runner ${KF5_MIN_VERSION} CONFIG)
set_package_properties(KF5Runner PROPERTIES
    PURPOSE "Enable building of KWin with krunner support"
    TYPE OPTIONAL
    )
add_feature_info("KF5Runner" KF5Runner_FOUND "Enable building of KWin with krunner support")

find_package(KF5Kirigami2 ${KF5_MIN_VERSION} CONFIG)
set_package_properties(KF5Kirigami2 PROPERTIES
    DESCRIPTION "A QtQuick based components set"
    PURPOSE "Required at runtime for Virtual desktop KCM"
    TYPE RUNTIME
)

find_package(KDecoration2 5.18.0 CONFIG REQUIRED)

find_package(KScreenLocker CONFIG REQUIRED)
set_package_properties(KScreenLocker PROPERTIES
    TYPE REQUIRED
    PURPOSE "For screenlocker integration in kwin_wayland"
)

find_package(KWaylandServer CONFIG REQUIRED)
set_package_properties(KWaylandServer PROPERTIES
    TYPE REQUIRED
    PURPOSE "For Wayland integration"
)

find_package(Breeze 5.9.0 CONFIG)
set_package_properties(Breeze PROPERTIES
    TYPE OPTIONAL
    PURPOSE "For setting the default window decoration plugin"
)
if (${Breeze_FOUND})
    if (${BREEZE_WITH_KDECORATION})
        set(HAVE_BREEZE_DECO true)
    else()
        set(HAVE_BREEZE_DECO FALSE)
    endif()
else()
    set(HAVE_BREEZE_DECO FALSE)
endif()
add_feature_info("Breeze-Decoration" HAVE_BREEZE_DECO "Default decoration plugin Breeze")

find_package(EGL)
set_package_properties(EGL PROPERTIES
    TYPE RUNTIME
    PURPOSE "Required to build KWin with EGL support"
)

find_package(epoxy)
set_package_properties(epoxy PROPERTIES
    DESCRIPTION "libepoxy"
    URL "https://github.com/anholt/libepoxy"
    TYPE REQUIRED
    PURPOSE "OpenGL dispatch library"
)

set(HAVE_DL_LIBRARY FALSE)
if (epoxy_HAS_GLX)
    find_library(DL_LIBRARY dl)
    if (DL_LIBRARY)
        set(HAVE_DL_LIBRARY TRUE)
    endif()
endif()

find_package(Wayland 1.2 OPTIONAL_COMPONENTS Egl)
set_package_properties(Wayland PROPERTIES
    TYPE REQUIRED
    PURPOSE "Required for building KWin with Wayland support"
)
add_feature_info("Wayland::EGL" Wayland_Egl_FOUND "Enable building of Wayland backend.")
set(HAVE_WAYLAND_EGL FALSE)
if (Wayland_Egl_FOUND)
    set(HAVE_WAYLAND_EGL TRUE)
endif()

find_package(XKB 0.7.0)
set_package_properties(XKB PROPERTIES
    TYPE REQUIRED
    PURPOSE "Required for building KWin with Wayland support"
)

find_package(Libinput 1.9)
set_package_properties(Libinput PROPERTIES TYPE REQUIRED PURPOSE "Required for input handling on Wayland.")

find_package(UDev)
set_package_properties(UDev PROPERTIES
    URL "https://www.freedesktop.org/wiki/Software/systemd/"
    DESCRIPTION "Linux device library."
    TYPE REQUIRED
    PURPOSE "Required for input handling on Wayland."
)

find_package(Libdrm 2.4.62)
set_package_properties(Libdrm PROPERTIES TYPE OPTIONAL PURPOSE "Required for drm output on Wayland.")
set(HAVE_DRM FALSE)
if (Libdrm_FOUND)
    set(HAVE_DRM TRUE)
endif()

find_package(gbm)
set_package_properties(gbm PROPERTIES TYPE OPTIONAL PURPOSE "Required for egl output of drm backend.")
set(HAVE_GBM FALSE)
if (HAVE_DRM AND gbm_FOUND)
    set(HAVE_GBM TRUE)
endif()

option(KWIN_BUILD_EGL_STREAM_BACKEND "Enable building of EGLStream based DRM backend" ON)
if (HAVE_DRM AND KWIN_BUILD_EGL_STREAM_BACKEND)
    set(HAVE_EGL_STREAMS TRUE)
endif()

find_package(libhybris)
set_package_properties(libhybris PROPERTIES TYPE OPTIONAL PURPOSE "Required for libhybris backend")
set(HAVE_LIBHYBRIS ${libhybris_FOUND})

find_package(libjingsideproxy)
set_package_properties(libjingsideproxy PROPERTIES TYPE OPTIONAL PURPOSE "Required for jingsideproxy")
set(HAVE_JINGSIDEPROXY ${libjingsideproxy_FOUND})

find_package(perfetto)
set_package_properties(libperfetto PROPERTIES TYPE OPTIONAL PURPOSE "Required for perfetto")
set(HAVE_PERFETTO ${libperfetto_FOUND})

find_package(X11)
set_package_properties(X11 PROPERTIES
    DESCRIPTION "X11 libraries"
    URL "https://www.x.org"
    TYPE REQUIRED
)
add_feature_info("XInput" X11_Xi_FOUND "Required for poll-free mouse cursor updates")
set(HAVE_X11_XINPUT ${X11_Xinput_FOUND})

find_package(lcms2)
set_package_properties(lcms2 PROPERTIES
    DESCRIPTION "Small-footprint color management engine"
    URL "http://www.littlecms.com"
    TYPE OPTIONAL
)
add_feature_info("lcms2" lcms2_FOUND "Required for the color management system")

# All the required XCB components
find_package(XCB 1.10 REQUIRED COMPONENTS
    COMPOSITE
    CURSOR
    DAMAGE
    GLX
    ICCCM
    IMAGE
    KEYSYMS
    RANDR
    RENDER
    SHAPE
    SHM
    SYNC
    XCB
    XFIXES
)
set_package_properties(XCB PROPERTIES TYPE REQUIRED)

# and the optional XCB dependencies
if (XCB_ICCCM_VERSION VERSION_LESS "0.4")
    set(XCB_ICCCM_FOUND FALSE)
endif()
add_feature_info("XCB-ICCCM" XCB_ICCCM_FOUND "Required for building test applications for KWin")

find_package(X11_XCB)
set_package_properties(X11_XCB PROPERTIES
    PURPOSE "Required for building X11 windowed backend of kwin_wayland"
    TYPE OPTIONAL
)

# dependencies for QPA plugin
find_package(Qt5FontDatabaseSupport REQUIRED)
find_package(Qt5ThemeSupport REQUIRED)
find_package(Qt5EventDispatcherSupport REQUIRED)

find_package(Freetype REQUIRED)
set_package_properties(Freetype PROPERTIES
    DESCRIPTION "A font rendering engine"
    URL "https://www.freetype.org"
    TYPE REQUIRED
    PURPOSE "Needed for KWin's QPA plugin."
)
find_package(Fontconfig REQUIRED)
set_package_properties(Fontconfig PROPERTIES
    TYPE REQUIRED
    PURPOSE "Needed for KWin's QPA plugin."
)

find_package(Xwayland)
set_package_properties(Xwayland PROPERTIES
    URL "https://x.org"
    DESCRIPTION "Xwayland X server"
    TYPE RUNTIME
    PURPOSE "Needed for running kwin_wayland"
)

find_package(Libcap)
set_package_properties(Libcap PROPERTIES
    TYPE OPTIONAL
    PURPOSE "Needed for running kwin_wayland with real-time scheduling policy"
)
set(HAVE_LIBCAP ${Libcap_FOUND})

find_package(hwdata)
set_package_properties(hwdata PROPERTIES
    TYPE RUNTIME
    PURPOSE "Runtime-only dependency needed for mapping monitor hardware vendor IDs to full names"
    URL "https://github.com/vcrhonek/hwdata"
)

find_package(QAccessibilityClient CONFIG)
set_package_properties(QAccessibilityClient PROPERTIES
    URL "https://www.kde.org"
    DESCRIPTION "KDE client-side accessibility library"
    TYPE OPTIONAL
    PURPOSE "Required to enable accessibility features"
)
set(HAVE_ACCESSIBILITY ${QAccessibilityClient_FOUND})

if(CMAKE_SYSTEM MATCHES "FreeBSD")
    find_package(epoll)
    set_package_properties(epoll PROPERTIES DESCRIPTION "I/O event notification facility"
        TYPE REQUIRED
        PURPOSE "Needed for running kwin_wayland"
    )
endif()

include(ECMQMLModules)
ecm_find_qmlmodule(QtQuick 2.3)
ecm_find_qmlmodule(QtQuick.Controls 1.2)
ecm_find_qmlmodule(QtQuick.Layouts 1.3)
ecm_find_qmlmodule(QtQuick.Window 2.1)
ecm_find_qmlmodule(QtMultimedia 5.0)
ecm_find_qmlmodule(org.kde.kquickcontrolsaddons 2.0)
ecm_find_qmlmodule(org.kde.plasma.core 2.0)
ecm_find_qmlmodule(org.kde.plasma.components 2.0)

########### configure tests ###############
include(CMakeDependentOption)

option(KWIN_BUILD_DECORATIONS "Enable building of KWin decorations." ON)
option(KWIN_BUILD_KCMS "Enable building of KWin configuration modules." ON)
option(KWIN_BUILD_TABBOX "Enable building of KWin Tabbox functionality" ON)
option(KWIN_BUILD_XRENDER_COMPOSITING "Enable building of KWin with XRender Compositing support" ON)
cmake_dependent_option(KWIN_BUILD_ACTIVITIES "Enable building of KWin with kactivities support" ON "KF5Activities_FOUND" OFF)
cmake_dependent_option(KWIN_BUILD_RUNNERS "Enable building of KWin with krunner support" ON "KF5Runner_FOUND" OFF)
cmake_dependent_option(KWIN_BUILD_CMS "Enable building of KWin with CMS" ON "lcms2_FOUND" OFF)

# Binary name of KWin
set(KWIN_NAME "kwin")
set(KWIN_INTERNAL_NAME_X11 "kwin_x11")
set(KWIN_INTERNAL_NAME_WAYLAND "kwin_wayland")

# KWIN_HAVE_XRENDER_COMPOSITING - whether XRender-based compositing support is available: may be disabled
if (KWIN_BUILD_XRENDER_COMPOSITING)
    set(KWIN_HAVE_XRENDER_COMPOSITING 1)
endif()

include_directories(${XKB_INCLUDE_DIR})

set(HAVE_EPOXY_GLX ${epoxy_HAS_GLX})

# for things that are also used by kwin libraries
configure_file(libkwineffects/kwinconfig.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/libkwineffects/kwinconfig.h )
# for kwin internal things
set(HAVE_X11_XCB ${X11_XCB_FOUND})

include(CheckIncludeFile)
include(CheckIncludeFiles)
include(CheckSymbolExists)
check_include_files(unistd.h HAVE_UNISTD_H)
check_include_files(malloc.h HAVE_MALLOC_H)

check_include_file("sys/prctl.h" HAVE_SYS_PRCTL_H)
check_symbol_exists(PR_SET_DUMPABLE "sys/prctl.h" HAVE_PR_SET_DUMPABLE)
check_symbol_exists(PR_SET_PDEATHSIG "sys/prctl.h" HAVE_PR_SET_PDEATHSIG)
check_include_file("sys/procctl.h" HAVE_SYS_PROCCTL_H)
check_symbol_exists(PROC_TRACE_CTL "sys/procctl.h" HAVE_PROC_TRACE_CTL)
if (HAVE_PR_SET_DUMPABLE OR HAVE_PROC_TRACE_CTL)
    set(CAN_DISABLE_PTRACE TRUE)
endif()
add_feature_info("prctl/procctl tracing control"
                 CAN_DISABLE_PTRACE
                 "Required for disallowing ptrace on kwin_wayland process")

check_include_file("sys/sysmacros.h" HAVE_SYS_SYSMACROS_H)

check_include_file("linux/vt.h" HAVE_LINUX_VT_H)
add_feature_info("linux/vt.h"
                 HAVE_LINUX_VT_H
                 "Required for virtual terminal support under wayland")
check_include_file("linux/fb.h"    HAVE_LINUX_FB_H)
add_feature_info("linux/fb.h"
                 HAVE_LINUX_FB_H
                 "Required for the fbdev backend")

check_symbol_exists(SCHED_RESET_ON_FORK "sched.h" HAVE_SCHED_RESET_ON_FORK)
add_feature_info("SCHED_RESET_ON_FORK"
                 HAVE_SCHED_RESET_ON_FORK
                 "Required for running kwin_wayland with real-time scheduling")


pkg_check_modules(PipeWire IMPORTED_TARGET libpipewire-0.3)
add_feature_info(PipeWire PipeWire_FOUND "Required for Wayland screencasting")

configure_file(config-kwin.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-kwin.h)

########### global ###############
set(kwin_effects_dbus_xml ${CMAKE_CURRENT_SOURCE_DIR}/org.kde.kwin.Effects.xml)
qt5_add_dbus_interface(effects_interface_SRCS ${kwin_effects_dbus_xml} kwineffects_interface)
add_library(KWinEffectsInterface STATIC ${effects_interface_SRCS})
target_link_libraries(KWinEffectsInterface Qt::DBus)

include_directories(BEFORE
    ${CMAKE_CURRENT_BINARY_DIR}/libkwineffects
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/libkwineffects
    ${CMAKE_CURRENT_SOURCE_DIR}/effects
    ${CMAKE_CURRENT_SOURCE_DIR}/tabbox
    ${CMAKE_CURRENT_SOURCE_DIR}/platformsupport
)

add_subdirectory(libkwineffects)
if (KWIN_BUILD_KCMS)
    add_subdirectory(kcmkwin)
endif()

add_subdirectory(data)

add_subdirectory(effects)
add_subdirectory(scripts)
add_subdirectory(tabbox)
add_subdirectory(scripting)
add_subdirectory(helpers)

########### next target ###############

set(kwin_SRCS
    3rdparty/xcursor.c
    abstract_client.cpp
    abstract_opengl_context_attribute_builder.cpp
    abstract_output.cpp
    abstract_wayland_output.cpp
    activation.cpp
    appmenu.cpp
    atoms.cpp
    client_machine.cpp
    composite.cpp
    cursor.cpp
    dbusinterface.cpp
    debug_console.cpp
    decorations/decoratedclient.cpp
    decorations/decorationbridge.cpp
    decorations/decorationpalette.cpp
    decorations/decorationrenderer.cpp
    decorations/decorations_logging.cpp
    decorations/settings.cpp
    deleted.cpp
    dmabuftexture.cpp
    effectloader.cpp
    effects.cpp
    egl_context_attribute_builder.cpp
    events.cpp
    focuschain.cpp
    ftrace.cpp
    geometrytip.cpp
    gestures.cpp
    globalshortcuts.cpp
    group.cpp
    idle_inhibition.cpp
    input.cpp
    input_event.cpp
    input_event_spy.cpp
    inputmethod.cpp
    inputpanelv1client.cpp
    inputpanelv1integration.cpp
    internal_client.cpp
    keyboard_input.cpp
    keyboard_layout.cpp
    keyboard_layout_switching.cpp
    keyboard_repeat.cpp
    killwindow.cpp
    layers.cpp
    layershellv1client.cpp
    layershellv1integration.cpp
    libinput/connection.cpp
    libinput/context.cpp
    libinput/device.cpp
    libinput/events.cpp
    libinput/libinput_logging.cpp
    linux_dmabuf.cpp
    logind.cpp
    main.cpp
    globalmotions.cpp
    globalswipegestures.cpp
    globalpinchgestures.cpp
    globalgesture.cpp
    modifier_only_shortcuts.cpp
    moving_client_x11_filter.cpp
    netinfo.cpp
    onscreennotification.cpp
    options.cpp
    osd.cpp
    outline.cpp
    overlaywindow.cpp
    placement.cpp
    platform.cpp
    plugin.cpp
    pluginmanager.cpp
    pointer_input.cpp
    popup_input_filter.cpp
    renderjournal.cpp
    renderloop.cpp
    rootinfo_filter.cpp
    rulebooksettings.cpp
    rules.cpp
    scene.cpp
    screenedge.cpp
    screenlockerwatcher.cpp
    screens.cpp
    scripting/dbuscall.cpp
    scripting/meta.cpp
    scripting/screenedgeitem.cpp
    scripting/scriptedeffect.cpp
    scripting/scripting.cpp
    scripting/scripting_logging.cpp
    scripting/scripting_model.cpp
    scripting/scriptingutils.cpp
    scripting/timer.cpp
    scripting/workspace_wrapper.cpp
    shadow.cpp
    sm.cpp
    subsurfacemonitor.cpp
    syncalarmx11filter.cpp
    tablet_input.cpp
    thumbnailitem.cpp
    toplevel.cpp
    touch_hide_cursor_spy.cpp
    touch_input.cpp
    udev.cpp
    unmanaged.cpp
    useractions.cpp
    utils.cpp
    virtualdesktops.cpp
    virtualdesktopsdbustypes.cpp
    virtualkeyboard_dbus.cpp
    was_user_interaction_x11_filter.cpp
    wayland_server.cpp
    waylandclient.cpp
    waylandshellintegration.cpp
    window_property_notify_x11_filter.cpp
    workspace.cpp
    x11client.cpp
    x11eventfilter.cpp
    xcbutils.cpp
    xcursortheme.cpp
    xdgshellclient.cpp
    xdgshellintegration.cpp
    xkb.cpp
    xwaylandclient.cpp
    xwl/xwayland_interface.cpp
    3rdparty/jappmanager/jappmanagerclient.cpp
)

if (HAVE_LINUX_VT_H)
    set(kwin_SRCS ${kwin_SRCS}
        virtual_terminal.cpp
    )
    set(KWIN_TTY_PREFIX "/dev/tty")
endif()

if(CMAKE_SYSTEM MATCHES "FreeBSD")
    # We know it has epoll, so supports VT as well
    set(kwin_SRCS ${kwin_SRCS}
        virtual_terminal.cpp
    )
    set(KWIN_TTY_PREFIX "/dev/ttyv")
endif()
if(KWIN_TTY_PREFIX)
    set_source_files_properties(virtual_terminal.cpp PROPERTIES COMPILE_DEFINITIONS KWIN_TTY_PREFIX="${KWIN_TTY_PREFIX}")
endif()    

kconfig_add_kcfg_files(kwin_SRCS settings.kcfgc)
kconfig_add_kcfg_files(kwin_SRCS rulesettings.kcfgc)
kconfig_add_kcfg_files(kwin_SRCS rulebooksettingsbase.kcfgc)

qt5_add_dbus_adaptor(kwin_SRCS org.kde.KWin.xml dbusinterface.h KWin::DBusInterface)
qt5_add_dbus_adaptor(kwin_SRCS org.kde.kwin.Compositing.xml dbusinterface.h KWin::CompositorDBusInterface)
qt5_add_dbus_adaptor(kwin_SRCS ${kwin_effects_dbus_xml} effects.h KWin::EffectsHandlerImpl)
qt5_add_dbus_adaptor(kwin_SRCS org.kde.KWin.VirtualDesktopManager.xml dbusinterface.h KWin::VirtualDesktopManagerDBusInterface)
qt5_add_dbus_adaptor(kwin_SRCS org.kde.KWin.Session.xml sm.h KWin::SessionManager)
qt5_add_dbus_adaptor(kwin_SRCS org.kde.KWin.Plugins.xml dbusinterface.h KWin::PluginManagerDBusInterface)

qt5_add_dbus_interface(kwin_SRCS ${KSCREENLOCKER_DBUS_INTERFACES_DIR}/kf5_org.freedesktop.ScreenSaver.xml screenlocker_interface)
qt5_add_dbus_interface(kwin_SRCS ${KSCREENLOCKER_DBUS_INTERFACES_DIR}/org.kde.screensaver.xml kscreenlocker_interface)
qt5_add_dbus_interface(kwin_SRCS org.kde.kappmenu.xml appmenu_interface)

ki18n_wrap_ui(kwin_SRCS
    debug_console.ui
    shortcutdialog.ui
)

########### target link libraries ###############

add_library(kwin SHARED ${kwin_SRCS})
if (Libinput_VERSION_STRING VERSION_GREATER 1.14)
    set_source_files_properties(libinput/connection.cpp PROPERTIES COMPILE_DEFINITIONS LIBINPUT_HAS_TOTEM)
endif ()

if(HAVE_JINGSIDEPROXY)
    target_link_libraries(kwin jing_android_proxy)
endif()

target_link_libraries(kwin jappm_cli)

set_target_properties(kwin PROPERTIES
   VERSION ${PROJECT_VERSION}
   SOVERSION ${PROJECT_VERSION_MAJOR}
)

target_link_libraries(kwin
    kwin4_effect_builtins
    kwineffects
    kwinglutils

    Qt::Concurrent
    Qt::DBus
    Qt::Quick
    Qt::Script

    KF5::ConfigCore
    KF5::ConfigWidgets
    KF5::CoreAddons
    KF5::GlobalAccel
    KF5::GlobalAccelPrivate
    KF5::I18n
    KF5::Notifications
    KF5::Package
    KF5::Plasma
    KF5::QuickAddons
    KF5::WaylandClient
    KF5::WindowSystem

    KDecoration2::KDecoration
    KDecoration2::KDecoration2Private

    PW::KScreenLocker
    Plasma::KWaylandServer

    XCB::COMPOSITE
    XCB::DAMAGE
    XCB::GLX
    XCB::ICCCM
    XCB::KEYSYMS
    XCB::RANDR
    XCB::RENDER
    XCB::SHAPE
    XCB::SHM
    XCB::SYNC
    XCB::XCB
    XCB::XFIXES

    Libinput::Libinput
    UDev::UDev
    XKB::XKB
    epoxy::epoxy

    ${CMAKE_THREAD_LIBS_INIT}
)

if (KWIN_BUILD_CMS)
    target_sources(kwin PRIVATE
        colordevice.cpp
        colormanager.cpp
    )
    target_link_libraries(kwin lcms2::lcms2)
endif()

if (KWIN_BUILD_ACTIVITIES)
    target_sources(kwin PRIVATE activities.cpp)
    target_link_libraries(kwin KF5::Activities)
endif()

if (KWIN_BUILD_TABBOX)
    target_sources(kwin PRIVATE
        tabbox/clientmodel.cpp
        tabbox/desktopchain.cpp
        tabbox/desktopmodel.cpp
        tabbox/switcheritem.cpp
        tabbox/tabbox.cpp
        tabbox/tabbox_logging.cpp
        tabbox/tabboxconfig.cpp
        tabbox/tabboxhandler.cpp
        tabbox/x11_filter.cpp
    )
    target_include_directories(kwin PRIVATE ${Qt5Gui_PRIVATE_INCLUDE_DIRS})
endif()

generate_export_header(kwin EXPORT_FILE_NAME kwin_export.h)

if(CMAKE_SYSTEM MATCHES "FreeBSD")
    # epoll is required, includes live under ${LOCALBASE}, separate library
    target_include_directories(kwin PUBLIC ${epoll_INCLUDE_DIRS})
    target_link_libraries(kwin ${epoll_LIBRARIES}) 
endif()

add_executable(kwin_x11 main_x11.cpp)
target_link_libraries(kwin_x11 kwin KF5::Crash Qt::X11Extras)

install(TARGETS kwin ${KDE_INSTALL_TARGETS_DEFAULT_ARGS} LIBRARY NAMELINK_SKIP)
install(TARGETS kwin_x11 ${KDE_INSTALL_TARGETS_DEFAULT_ARGS})

set(kwin_XWAYLAND_SRCS
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/clipboard.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/databridge.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/dnd.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/drag.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/drag_wl.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/drag_x.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/selection.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/selection_source.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/transfer.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/xwl/xwayland.cpp
)
include(ECMQtDeclareLoggingCategory)
ecm_qt_declare_logging_category(kwin_XWAYLAND_SRCS
    HEADER
        xwayland_logging.h
    IDENTIFIER
        KWIN_XWL
    CATEGORY_NAME
        kwin_xwl
    DEFAULT_SEVERITY
        Warning
)

set(kwin_WAYLAND_SRCS
   main_wayland.cpp
   tabletmodemanager.cpp
)

add_executable(kwin_wayland ${kwin_WAYLAND_SRCS} ${kwin_XWAYLAND_SRCS})
target_link_libraries(kwin_wayland
                      kwin
                      KF5::Crash
)

if (HAVE_PERFETTO)
    target_link_libraries(kwin_wayland libperfetto::sdk)
endif()

if (HAVE_LIBCAP)
    target_link_libraries(kwin_wayland ${Libcap_LIBRARIES})
endif()

install(TARGETS kwin_wayland ${KDE_INSTALL_TARGETS_DEFAULT_ARGS})
if (HAVE_LIBCAP)
    install(
    CODE "execute_process(
            COMMAND
                ${SETCAP_EXECUTABLE}
                CAP_SYS_NICE=+ep
                \$ENV{DESTDIR}${CMAKE_INSTALL_FULL_BINDIR}/kwin_wayland)"
    )
endif()

add_subdirectory(platformsupport)
add_subdirectory(plugins)

target_link_libraries(kwin_wayland
    KWinQpaPlugin
    KF5GlobalAccelKWinPlugin
    KF5WindowSystemKWinPlugin
    KF5IdleTimeKWinPlugin
)

if (PipeWire_FOUND)
    target_link_libraries(kwin_wayland KWinScreencastPlugin)
endif()

########### install files ###############

install(FILES kwin.kcfg DESTINATION ${KCFG_INSTALL_DIR} RENAME ${KWIN_NAME}.kcfg)
install(FILES kwin.notifyrc DESTINATION ${KNOTIFYRC_INSTALL_DIR} RENAME ${KWIN_NAME}.notifyrc)
install(
    FILES
        org.kde.KWin.VirtualDesktopManager.xml
        org.kde.KWin.xml
        org.kde.kwin.Compositing.xml
        org.kde.kwin.Effects.xml
        org.kde.KWin.Plugins.xml
    DESTINATION
        ${KDE_INSTALL_DBUSINTERFACEDIR}
)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/kwin_export.h DESTINATION ${INCLUDE_INSTALL_DIR} COMPONENT Devel)

# Install the KWin/Script service type
install(FILES scripting/kwinscript.desktop DESTINATION ${SERVICETYPES_INSTALL_DIR})

add_subdirectory(qml)

if (BUILD_TESTING)
    find_package(WaylandProtocols 1.19 REQUIRED)
    find_package(QtWaylandScanner ${QT_MIN_VERSION} REQUIRED)
    find_package(Wayland REQUIRED COMPONENTS Client)

    #add_subdirectory(autotests)
    #add_subdirectory(tests)
endif()

if (KF5DocTools_FOUND)
    add_subdirectory(doc)
endif()

add_subdirectory(kconf_update)

# add clang-format target for all our real source files
file(GLOB_RECURSE ALL_CLANG_FORMAT_SOURCE_FILES *.cpp *.h)
kde_clang_format(${ALL_CLANG_FORMAT_SOURCE_FILES})

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)

include(CMakePackageConfigHelpers)
set(CMAKECONFIG_INSTALL_DIR "${CMAKECONFIG_INSTALL_PREFIX}/KWinDBusInterface")
configure_package_config_file(KWinDBusInterfaceConfig.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/KWinDBusInterfaceConfig.cmake"
    PATH_VARS KDE_INSTALL_DBUSINTERFACEDIR
    INSTALL_DESTINATION ${CMAKECONFIG_INSTALL_DIR})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/KWinDBusInterfaceConfig.cmake
    DESTINATION ${CMAKECONFIG_INSTALL_DIR})


# plasma-kwin_wayland.service.in is currently disabled as plasma-workspace currently launches
# kwin-wayland explicitly. Porting this is currently blocked on propogating DISPLAY
ecm_install_configured_files(INPUT plasma-kwin_x11.service.in @ONLY
            DESTINATION ${SYSTEMD_USER_UNIT_INSTALL_DIR})

find_package(KF5I18n CONFIG REQUIRED)
ki18n_install(po)
