cmake_minimum_required(VERSION 3.15)
project(imgui)

set(CMAKE_CXX_STANDARD 17)

include(GNUInstallDirs)

set(IMGUI_HEADERS
        imconfig.h
        imgui.h
)
set(IMGUI_SOURCES
        imstb_rectpack.h
        imstb_textedit.h
        imstb_truetype.h
        imgui_internal.h
        imgui_demo.cpp
        imgui_draw.cpp
        imgui_tables.cpp
        imgui_widgets.cpp
        imgui.cpp
)

add_library(imgui STATIC ${IMGUI_HEADERS} ${IMGUI_SOURCES})
add_library(imgui::imgui ALIAS imgui)
target_include_directories(imgui PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/backends>"
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)
list(APPEND IMGUI_INSTALL_TARGETS imgui)
list(APPEND IMGUI_INSTALL_HEADERS ${IMGUI_HEADERS})

if (IMGUI_BACKEND_GLES3)
    set(IMGUI_GLES3_HEADERS
            backends/imgui_impl_opengl3.h
    )
    set(IMGUI_GLES3_SOURCES
            backends/imgui_impl_opengl3.cpp
    )
    find_package(glesx CONFIG REQUIRED)
    add_library(imgui_backend_gles3 STATIC ${IMGUI_GLES3_HEADERS} ${IMGUI_GLES3_SOURCES})
    add_library(imgui::imgui_backend_gles3 ALIAS imgui_backend_gles3)
    target_link_libraries(imgui_backend_gles3 PUBLIC imgui glesx::GLESv2)
    list(APPEND IMGUI_INSTALL_TARGETS imgui_backend_gles3)
    list(APPEND IMGUI_INSTALL_HEADERS ${IMGUI_GLES3_HEADERS})
endif()

if (IMGUI_BACKEND_GLFW)
    set(IMGUI_GLFW_HEADERS
            backends/imgui_impl_glfw.h
    )
    set(IMGUI_GLFW_SOURCES
            backends/imgui_impl_glfw.cpp
    )
    find_package(glfw3 CONFIG REQUIRED)
    add_library(imgui_backend_glfw STATIC ${IMGUI_GLFW_HEADERS} ${IMGUI_GLFW_SOURCES})
    add_library(imgui::imgui_backend_glfw ALIAS imgui_backend_glfw)
    target_link_libraries(imgui_backend_glfw PUBLIC imgui glfw)
    list(APPEND IMGUI_INSTALL_TARGETS imgui_backend_glfw)
    list(APPEND IMGUI_INSTALL_HEADERS ${IMGUI_GLFW_HEADERS})
endif()

include(CMakePackageConfigHelpers)
configure_package_config_file(
        "${CMAKE_CURRENT_LIST_DIR}/imgui-config.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/imgui-config.cmake"
        INSTALL_DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/imgui"
)
install(
        FILES
        "${CMAKE_CURRENT_BINARY_DIR}/imgui-config.cmake"
        DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/imgui"
)

install(TARGETS ${IMGUI_INSTALL_TARGETS} EXPORT imgui)
install(
        FILES ${IMGUI_INSTALL_HEADERS}
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)
install(
        EXPORT imgui
        FILE imgui-targets.cmake
        DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/imgui"
        NAMESPACE imgui::
)
