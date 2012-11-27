cmake_minimum_required(VERSION 2.8)
project(BRDF_Viewer)

# Output location of the final C++ binary.
set(BINARY_NAME bin/brdf_viewer)

# Set this to "Debug" if you want debug symbols for gdb and no optimization.
set(CMAKE_BUILD_TYPE Release)

# This sets the location of the Find.*.cmake files.
set(CMAKE_MODULE_PATH CMakeModules)

# Locations of header files
include_directories(${BRDF_Viewer_SOURCE_DIR}/brdf)
include_directories(${BRDF_Viewer_SOURCE_DIR}/brdf/ptex)
include_directories(${BRDF_Viewer_SOURCE_DIR}/shaderTemplates)

# Lists of source files.
file(GLOB brdf_SRC            "src/brdf/*cpp"           )
file(GLOB brdf_ptex_SRC       "src/brdf/ptex/*cpp"      )
file(GLOB shaderTemplates_SRC "src/shaderTemplates/*cpp")

# Print all source files for debugging purposes.
message(STATUS "Source files:")
foreach(FILE ${brdf_SRC})
    message(STATUS "\t${FILE}")
endforeach()

# Add the binary --------------------------------------------------------------
add_executable(${BINARY_NAME}  ${brdf_SRC} ${brdf_ptex_SRC}  ${QT_MOCED_HEADERS})

# QT --------------------------------------------------------------------------
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
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(${BINARY_NAME} ${QT_LIBRARIES})

# pthreads --------------------------------------------------------------------
set(CMAKE_THREAD_PREFER_PTHREAD 1) #In the presence of choices, use pthreads
find_package(Threads REQUIRED)
if(${Threads_FOUND})
    message("Successfully found threading library.")
    target_link_libraries(${BINARY_NAME} ${CMAKE_THREAD_LIBS_INIT})
else()
    message(FATAL_ERROR "Could not find a suitable threads library(usually pthreads)!")
endif()

# GLEW ------------------------------------------------------------------------
find_package(GLEW REQUIRED)
if(${GLEW_FOUND})
    message("Found GLEW at ${GLEW_INCLUDE_PATH}")
    target_link_libraries(${BINARY_NAME} ${GLEW_LIBRARY})
    INCLUDE_DIRECTORIES(${GLEW_INCLUDE_PATH})
else()
    message(FATAL_ERROR "Failed to find GLEW!")
endif()


#Hack! TODO FIX
target_link_libraries(${BINARY_NAME} GL GLU glut)



