set(WAYLAND_BACKEND_SOURCES
    logging.cpp
    scene_qpainter_wayland_backend.cpp
    wayland_backend.cpp
    wayland_output.cpp
)

if (HAVE_WAYLAND_EGL)
    set(WAYLAND_BACKEND_SOURCES egl_wayland_backend.cpp ${WAYLAND_BACKEND_SOURCES})
    if (HAVE_GBM)
        list(APPEND WAYLAND_BACKEND_SOURCES ../drm/gbm_dmabuf.cpp)
    endif()
endif()

add_library(KWinWaylandWaylandBackend MODULE ${WAYLAND_BACKEND_SOURCES})
set_target_properties(KWinWaylandWaylandBackend PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/org.kde.kwin.waylandbackends/")
target_link_libraries(KWinWaylandWaylandBackend kwin KF5::WaylandClient SceneQPainterBackend)

if (HAVE_WAYLAND_EGL)
    target_link_libraries(KWinWaylandWaylandBackend SceneOpenGLBackend Wayland::Egl)
    if (HAVE_GBM)
       target_link_libraries(KWinWaylandWaylandBackend gbm::gbm)
    endif()
endif()

if (HAVE_PERFETTO)
    target_link_libraries(KWinWaylandWaylandBackend libperfetto::sdk)
endif()

install(
    TARGETS
        KWinWaylandWaylandBackend
    DESTINATION
        ${KDE_INSTALL_PLUGINDIR}/org.kde.kwin.waylandbackends/
)
