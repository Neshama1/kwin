# KI18N Translation Domain for this library
add_definitions(-DTRANSLATION_DOMAIN=\"kcmkwinscreenedges\")

include_directories(${KWin_SOURCE_DIR}/effects)
set(kcm_screenedges_SRCS
    monitor.cpp
    screenpreviewwidget.cpp
    kwinscreenedge.cpp
    kwinscreenedgeconfigform.cpp
)

qt5_add_dbus_interface(kcm_screenedges_SRCS ${KWin_SOURCE_DIR}/org.kde.kwin.Effects.xml kwin_effects_interface)

set(kcm_kwinscreenedges_PART_SRCS main.cpp ${kcm_screenedges_SRCS})

kcmutils_generate_module_data(
    kcm_kwinscreenedges_PART_SRCS
    MODULE_DATA_HEADER kwinscreenedgedata.h
    MODULE_DATA_CLASS_NAME KWinScreenEdgeData
    SETTINGS_HEADERS kwinscreenedgesettings.h
    SETTINGS_CLASSES KWinScreenEdgeSettings
    NAMESPACE KWin
)

ki18n_wrap_ui(kcm_kwinscreenedges_PART_SRCS main.ui)
kconfig_add_kcfg_files(kcm_kwinscreenedges_PART_SRCS kwinscreenedgesettings.kcfgc kwinscreenedgescriptsettings.kcfgc)
add_library(kcm_kwinscreenedges MODULE ${kcm_kwinscreenedges_PART_SRCS})
set(kcm_screenedges_LIBS
    Qt::DBus

    KF5::Completion
    KF5::ConfigCore
    KF5::ConfigWidgets
    KF5::KCMUtils
    KF5::I18n
    KF5::Package
    KF5::Plasma
    KF5::Service

    kwin4_effect_builtins
)
target_link_libraries(kcm_kwinscreenedges ${X11_LIBRARIES} ${kcm_screenedges_LIBS})

set(kcm_kwintouchscreenedges_PART_SRCS
    touch.cpp
    kwintouchscreenedgeconfigform.cpp
    ${kcm_screenedges_SRCS}
)

kcmutils_generate_module_data(
    kcm_kwintouchscreenedges_PART_SRCS
    MODULE_DATA_HEADER kwintouchscreendata.h
    MODULE_DATA_CLASS_NAME KWinTouchScreenData
    SETTINGS_HEADERS kwintouchscreensettings.h
    SETTINGS_CLASSES KWinTouchScreenSettings
    NAMESPACE KWin
)

ki18n_wrap_ui(kcm_kwintouchscreenedges_PART_SRCS main.ui touch.ui)
kconfig_add_kcfg_files(kcm_kwintouchscreenedges_PART_SRCS kwintouchscreensettings.kcfgc kwintouchscreenscriptsettings.kcfgc)
add_library(kcm_kwintouchscreen MODULE ${kcm_kwintouchscreenedges_PART_SRCS})
target_link_libraries(kcm_kwintouchscreen ${X11_LIBRARIES} ${kcm_screenedges_LIBS})

install(TARGETS kcm_kwinscreenedges kcm_kwintouchscreen DESTINATION ${KDE_INSTALL_PLUGINDIR})
install(FILES kwinscreenedges.desktop kwintouchscreen.desktop DESTINATION ${SERVICES_INSTALL_DIR})
