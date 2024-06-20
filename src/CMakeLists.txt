set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# core

add_library(imtui ${IMTUI_LIBRARY_TYPE}
    imtui-impl-text.cpp
    )

target_include_directories(imtui PUBLIC
    ../include
    )

target_link_libraries(imtui PUBLIC
    imgui-for-imtui
    )

target_link_libraries(imtui PRIVATE
    ${CMAKE_DL_LIBS}
    )

set_target_properties(imtui PROPERTIES PUBLIC_HEADER "../include/imtui/imtui.h;../include/imtui/imtui-impl-text.h")

if (MINGW)
    target_link_libraries(imtui PUBLIC stdc++)
endif()

# ncurses

if (IMTUI_SUPPORT_NCURSES)

    include_directories(../external/pdcurses/)

    file(GLOB PDCURSES_INCLUDES ../external/pdcurses/pdcurses/*.h)
    file(GLOB PDCURSES_SOURCES ../external/pdcurses/pdcurses/*.c)

    file(GLOB WINCON_INCLUDES ../external/pdcurses/wincon/*.h)
    file(GLOB WINCON_SOURCES ../external/pdcurses/wincon/*.c)

    add_library(imtui-pdcurses STATIC
        ${PDCURSES_INCLUDES}
        ${PDCURSES_SOURCES}
        ${WINCON_INCLUDES}
        ${WINCON_SOURCES}
        )

    target_link_libraries(imtui-pdcurses PUBLIC
        winmm
        )

    add_library(imtui-ncurses ${IMTUI_LIBRARY_TYPE} 
        imtui-impl-ncurses.cpp
        )

    target_include_directories(imtui-ncurses PRIVATE
        ${CURSES_INCLUDE_DIR}
        )

    set_target_properties(imtui-ncurses PROPERTIES PUBLIC_HEADER "../include/imtui/imtui-impl-ncurses.h")

    target_link_libraries(imtui-ncurses PUBLIC
        imtui
        imtui-pdcurses
        ${CURSES_LIBRARIES}
        )
endif()

# emscripten

if (EMSCRIPTEN)
    add_library(imtui-emscripten ${IMTUI_LIBRARY_TYPE}
        imtui-impl-emscripten.cpp
        )

    target_link_libraries(imtui-emscripten PUBLIC
        imtui
        )
endif()

if (IMTUI_STANDALONE AND NOT EMSCRIPTEN)
    install(TARGETS imtui
        EXPORT imtui
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/imtui
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/imtui
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )

    install(TARGETS imtui-ncurses
        EXPORT imtui-ncurses
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/imtui
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/imtui
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )
endif()
