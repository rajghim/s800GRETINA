#----------------------------------------------------------------------------
# Setup the project
cmake_minimum_required(VERSION 2.6 FATAL_ERROR)
project(GRETINA+S800)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -D_FILE_OFFSET_64")

#Find ROOT
find_package(ROOT REQUIRED)
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${ROOT_CXX_FLAGS}")
include_directories(${ROOT_INCLUDE_DIR})
link_directories(${ROOT_LIBRARY_DIR})

#---Define useful ROOT functions and macros (e.g. ROOT_GENERATE_DICTIONARY)
include(${ROOT_USE_FILE})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)

#----------------------------------------------------------------------------
# Locate sources and headers for this project
# NB: headers are included so they will show up in IDEs
#
file(GLOB json_source ${PROJECT_SOURCE_DIR}/src/jsoncpp.cpp)
file(GLOB s800_sources src/S800Functions.cpp)

# Building GRETINADict and libGRETINA
ROOT_GENERATE_DICTIONARY(GRETINADict ${CMAKE_CURRENT_SOURCE_DIR}/include/GRETINA.h ${CMAKE_CURRENT_SOURCE_DIR}/include/SortingStructures.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/GRETINAWavefunction.h ${CMAKE_CURRENT_SOURCE_DIR}/include/INLCorrection.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/Histos.h ${CMAKE_CURRENT_SOURCE_DIR}/include/Track.h ${CMAKE_CURRENT_SOURCE_DIR}/include/UnpackUtilities.h 
    LINKDEF ${CMAKE_CURRENT_SOURCE_DIR}/include/LinkDefGRETINA.h)
ROOT_GENERATE_DICTIONARY(S800Dict ${CMAKE_CURRENT_SOURCE_DIR}/include/S800Parameters.h
    LINKDEF ${CMAKE_CURRENT_SOURCE_DIR}/include/LinkDefS800.h)
add_library(GRETINA SHARED GRETINADict ${CMAKE_CURRENT_SOURCE_DIR}/src/GRETINA.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/SortingStructures.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/SuperPulse.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/INLCorrection.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/G3Waveform.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Histos.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/Track.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/UnpackUtilities.cpp)
add_library(S800 SHARED S800Dict ${CMAKE_CURRENT_SOURCE_DIR}/src/S800Parameters.cpp)
target_link_libraries(GRETINA ${ROOT_LIBRARIES})
target_link_libraries(S800 ${ROOT_LIBRARIES})

file(GLOB gretina_sources src/Globals.cpp src/Unpack.cpp src/UnpackUtilities.cpp src/S800Functions.cpp)

# Add definition to incude S800
add_definitions(-DWITH_S800)

#
# Add the executable, and link it to the libraries
#
add_executable(s800GRETINA ${gretina_sources} ${s800_sources})
target_link_libraries(s800GRETINA ${ROOT_LIBRARIES} GRETINA S800)

add_executable(GEB_HFC ${CMAKE_CURRENT_SOURCE_DIR}/src/hfc/GEB_HFC.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/hfc/HFC.cpp)

#
# Copy over files to the build directory
#
set(_SCRIPTS
    #config.json
    crmat.dat
    gretina.set
    track.chat
    gretinaCalibrations/gCalibration.dat
    gretinaCalibrations/segmentCenters.dat
    s800Calibrations/s800.set
    s800Calibrations/S800Scaler.definition
)
foreach(_script ${_SCRIPTS})
    configure_file(
            ${PROJECT_SOURCE_DIR}/${_script}
            ${PROJECT_BINARY_DIR}/${_script}
            COPYONLY
)
endforeach()
