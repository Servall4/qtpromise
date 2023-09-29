cmake_minimum_required(VERSION 3.8)

if(DEFINED PROJECT_NAME)
    set(SUBPROJECT ON)
endif()

project(qtpromise VERSION 0.7.0 LANGUAGES CXX)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

include(CMakeDependentOption)
include(CMakePackageConfigHelpers)
include(CTest)
include(GNUInstallDirs)

find_package(Qt6 REQUIRED COMPONENTS Core)

set(CMAKE_AUTOMOC ON)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_definitions(
    -DQT_DEPRECATED_WARNINGS
    -DQT_NO_KEYWORDS
)

set (QTPROMISE_SRCS
        include/QtPromise
        include/qpromise_p.h
        include/qpromiseglobal.h
        include/qpromiseresolver.h
        include/qpromisehelpers_p.h
        include/qpromisefuture.h
        include/qpromise.inl
        include/qpromiseconnections.h
        include/qpromiseexceptions.h
        include/qpromisehelpers.h
        include/qpromise.h
)

# SHARED Library
#target_compile_definitions(qtpromise PUBLIC QTPROMISE_SHARED_LIB)

# STATIC Library
add_library(qtpromise INTERFACE)
add_library(qtrpomisesrc STATIC ${QTPROMISE_SRCS})
add_library(qtpromise::qtpromise ALIAS qtpromise)

#add_library(qtpromise STATIC ${QTPROMISE_SRCS})
#add_library(qtpromise::qtpromise ALIAS qtpromise)

target_link_libraries(qtpromise INTERFACE Qt6::Core)

string(CONCAT prefix
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>"
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")

target_include_directories(qtpromise INTERFACE ${prefix})

CMAKE_DEPENDENT_OPTION(qtpromise.installation
        "Include qtpromise in the install set"
        "${INSTALL_SUBPROJECTS}" "subproject" ON)
mark_as_advanced(qtpromise.installation)

# https://github.com/simonbrunel/qtpromise/issues/10
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    # https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html
    add_compile_options(
        -Werror
        -Wpedantic
        -Wall
        -Wextra
        -Wconversion
        -Wdouble-promotion
        -Wformat=2
        -Wlogical-op
        -Wmissing-noreturn
        -Wold-style-cast
#        -Wshadow                   # disabled due to many findings in the current code
        -Wsign-conversion
        -Wswitch-default
        -Wunused-local-typedefs
        -pedantic-errors
    )

    # https://github.com/Barro/compiler-warnings/blob/master/gcc/warnings-gcc-6.txt
    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 6)
        add_compile_options(
            -Wduplicated-cond
        )
    endif()

    # https://github.com/Barro/compiler-warnings/blob/master/gcc/warnings-gcc-7.txt
    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 7)
        add_compile_options(
            -Wduplicated-branches
        )
    endif()
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    # https://clang.llvm.org/docs/DiagnosticsReference.html
    add_compile_options(
        -Wall
        -Wextra
        -Wpedantic
        -Wsuggest-destructor-override
        -Wsuggest-override
    )
elseif(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    # https://docs.microsoft.com/en-us/cpp/build/reference/compiler-option-warning-level
    add_compile_options(
        /WX
    )
endif()

if (qtpromise.installation)

    install(DIRECTORY include/
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
    )

    install(TARGETS qtpromise EXPORT qtpromiseTargets)

    install(EXPORT qtpromiseTargets
            FILE qtpromise-targets.cmake
            NAMESPACE qtpromise::
            DESTINATION share/cmake/qtpromise)

    write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/qtpromiseConfigVersion.cmake
            VERSION ${PROJECT_VERSION}
            COMPATIBILITY SameMajorVersion)

    configure_file(
            "${PROJECT_SOURCE_DIR}/cmake/qtpromise-config.cmake.in"
            "${PROJECT_BINARY_DIR}/qtpromise-config.cmake"
            @ONLY)

    install(FILES
            "${PROJECT_BINARY_DIR}/qtpromiseConfigVersion.cmake"
            "${PROJECT_BINARY_DIR}/qtpromise-config.cmake"
            DESTINATION share/cmake/qtpromise)

    if(NOT SUBPROJECT)
        set(CPACK_PACKAGE_VENDOR "simonbrunel")
        set(CPACK_PACKAGE_DESCRIPTION_SUMMARY
                "QPromise library")
        set(CMAKE_PROJECT_HOMEPAGE_URL "https://github.com/Servall4/qtpromise")
        set(CPACK_RESOURCE_FILE_LICENSE "${PROJECT_SOURCE_DIR}/LICENSE")
        set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
        set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
        set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
        include(CPack)
    endif()
endif (qtpromise.installation)