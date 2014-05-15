cmake_minimum_required(VERSION 2.8)

# High Level setup ------------------------------------------------------------
project(BRDF_Viewer) # Project name.
set(BINARY_NAME brdf) # Output location of the final C++ binary.
set(CMAKE_BUILD_TYPE Release) # Choose from {Debug, Release}.  Affects
# optimization & debug symbols.
set(CMAKE_MODULE_PATH CMakeModules) # Location of the Find.*.cmake files.

# Find source and header files ------------------------------------------------

# Locations of header files.
include_directories(${BRDF_Viewer_SOURCE_DIR}/brdf)
include_directories(${BRDF_Viewer_SOURCE_DIR}/brdf/ptex)
include_directories(${BRDF_Viewer_SOURCE_DIR}/shaderTemplates)

# Lists of source files.
file(GLOB brdf_SRC            "src/brdf/*cpp"           )
file(GLOB brdf_ptex_SRC       "src/brdf/ptex/*cpp"      )
file(GLOB shaderTemplates_SRC "src/shaderTemplates/*cpp")

# Print debugging info --------------------------------------------------------
message(STATUS "Source files:")
foreach(FILE ${brdf_SRC})
    message(STATUS "\t${FILE}")
endforeach()

# QT MOC Related --------------------------------------------------------------
find_package(Qt4 COMPONENTS QtCore QtGui QtOpenGL REQUIRED)

# Find files we need to invoke QT MOC on.
set(MOC_FILES_LOC "src/brdf")
file(GLOB_RECURSE QT_FILES_TO_MOC ${MOC_FILES_LOC}/*.h)

# Invoke MOC.
message(STATUS "Will apply QT MOC to:")
foreach(FILE ${QT_FILES_TO_MOC})
    message(STATUS "\t${FILE}")
endforeach()
QT4_WRAP_CPP(QT_MOCED_HEADERS ${QT_FILES_TO_MOC})

#QT includes.
include(${QT_USE_FILE})
add_definitions(${QT_DEFINITIONS})

# Add the binary --------------------------------------------------------------
add_executable(${BINARY_NAME}  ${brdf_SRC} ${brdf_ptex_SRC}  ${QT_MOCED_HEADERS})

# QT library stuff ------------------------------------------------------------
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(${BINARY_NAME} ${QT_LIBRARIES})

# pthreads library ------------------------------------------------------------
set(CMAKE_THREAD_PREFER_PTHREAD 1) #In the presence of choices, use pthreads
find_package(Threads REQUIRED)
if(${Threads_FOUND})
    message("Successfully found threading library.")
    target_link_libraries(${BINARY_NAME} ${CMAKE_THREAD_LIBS_INIT})
else()
    message(FATAL_ERROR "Could not find a suitable threads library(usually pthreads)!")
endif()

find_package(PkgConfig REQUIRED)

# OpenGL and GLU libraries ----------------------------------------------------
find_package(OpenGL)
if(${OPENGL_FOUND})
    message("Found OpenGL at ${OPENGL_INCLUDE_DIR}")
    TARGET_LINK_LIBRARIES(${BINARY_NAME} ${OPENGL_LIBRARIES})
    INCLUDE_DIRECTORIES(${OPENGL_INCLUDE_DIR})
else()
    message(FATAL_ERROR "Failed to find OpenGL!")
endif()
if(${OPENGL_GLU_FOUND})
    message("Found OpenGL GLU at ${OEPNGL_glu_LIBRARY}")
    TARGET_LINK_LIBRARIES(${BINARY_NAME} ${OPENGL_glu_LIBRARY})
else()
    message(FATAL_ERROR "Failed to find OpenGL GLU!")
endif()

# GLEW library ----------------------------------------------------------------
pkg_search_module(GLEW REQUIRED glew)
if(${GLEW_FOUND})
    message("Found GLEW at ${GLEW_INCLUDEDIR}")
    target_link_libraries(${BINARY_NAME} ${GLEW_STATIC_LIBRARIES})
    INCLUDE_DIRECTORIES(${GLEW_INCLUDEDIR})
else()
    message(FATAL_ERROR "Failed to find GLEW!")
endif()

install(TARGETS ${BINARY_NAME} RUNTIME DESTINATION $ENV{HOME}/brdf)
