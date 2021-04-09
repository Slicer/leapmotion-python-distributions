cmake_minimum_required(VERSION 3.18)

#
# For more details, see docs/building.rst
#

project(LeapMotionPythonDistributions)

# Set a default build type if none was specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to 'Release' as none was specified.")
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build." FORCE)
  mark_as_advanced(CMAKE_BUILD_TYPE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

if(NOT DEFINED ${PROJECT_NAME}_SUPERBUILD)
  set(${PROJECT_NAME}_SUPERBUILD 1)
endif()

if(${PROJECT_NAME}_SUPERBUILD)

  #-----------------------------------------------------------------------------
  # Options

  # NA

  #-----------------------------------------------------------------------------
  include(ExternalProject)

  # Add an empty external project
  function(lmpd_ExternalProject_Add_Empty proj depends)
    set(depends_args)
    if(NOT depends STREQUAL "")
      set(depends_args DEPENDS ${depends})
    endif()
    ExternalProject_add(${proj}
      SOURCE_DIR ${CMAKE_BINARY_DIR}/${proj}
      DOWNLOAD_COMMAND ""
      UPDATE_COMMAND ""
      CONFIGURE_COMMAND ""
      BUILD_COMMAND ""
      BUILD_IN_SOURCE 1
      INSTALL_COMMAND ""
      ${depends_args}
      )
  endfunction()

  set(_ep_common_arguments "")
  foreach(_cmake_option IN ITEMS
    CMAKE_BUILD_PARALLEL_LEVEL
    CMAKE_BUILD_TYPE
    CMAKE_EXPORT_COMPILE_COMMANDS
    CMAKE_JOB_POOL_COMPILE
    CMAKE_JOB_POOL_LINK
    CMAKE_JOB_POOLS
    CMAKE_TOOLCHAIN_FILE
    )
    if(DEFINED ${_cmake_option})
      list(APPEND _ep_common_arguments CMAKE_CACHE_ARGS
        -D${_cmake_option}:BOOL=${${_cmake_option}}
        )
    endif()
  endforeach()

  message(STATUS "SuperBuild - ${PROJECT_NAME} => Requires LeapCxx")
  message(STATUS "SuperBuild -   LeapCxx => Requires LeapSDK, SWIG")

  #
  # LeapSDK
  #
  if(APPLE)
    set(LeapSDK_URL "https://warehouse.leapmotion.com/apps/4181/download")
    set(LeapSDK_FILENAME "Leap_Motion_SDK_Mac_2.3.1.tgz")
    set(LeapSDK_HASH "SHA256=0b33130d3e781c6a5eb5bf513e839f56acac897cfc71643f81ea6db5b8cbd292")
  elseif(WIN32)
    set(LeapSDK_URL "https://warehouse.leapmotion.com/apps/4668/download")
    set(LeapSDK_FILENAME "LeapDeveloperKit_4.1.0+52211_win.zip")
    set(LeapSDK_HASH "SHA256=065f8202b5b1c8d6e4080acbae8c696967c70cb9c2a754b5c592f435cb85ee7d")
  else()
    set(LeapSDK_URL "https://warehouse.leapmotion.com/apps/4185/download")
    set(LeapSDK_FILENAME "Leap_Motion_SDK_Linux_2.3.1.tgz")
    set(LeapSDK_HASH "SHA256=f686766fef7144e137e85114db73539a6cc2c5c0bdad2379352041fc54f04391")
  endif()

  ExternalProject_add(LeapSDK-binary-download
    SOURCE_DIR ${CMAKE_BINARY_DIR}/LeapSDK
    URL ${LeapSDK_URL}
    URL_HASH ${LeapSDK_HASH}
    DOWNLOAD_NAME ${LeapSDK_FILENAME}
    DOWNLOAD_DIR ${CMAKE_BINARY_DIR}
    USES_TERMINAL_DOWNLOAD 1
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    BUILD_IN_SOURCE 1
    INSTALL_COMMAND ""
    )
  set(LeapSDK_DIR "${CMAKE_BINARY_DIR}/LeapSDK/LeapSDK/lib/cmake/LeapSDK/")

  message(STATUS "SuperBuild -     LeapSDK[OK]")
  message(STATUS "SuperBuild -     LeapSDK_URL: ${LeapSDK_URL}")
  message(STATUS "SuperBuild -     LeapSDK_FILENAME: ${LeapSDK_FILENAME}")
  message(STATUS "SuperBuild -     LeapSDK_HASH: ${LeapSDK_HASH}")
  message(STATUS "SuperBuild -     LeapSDK_DIR: ${LeapSDK_DIR}")

  #
  # SWIG
  #
  if(WIN32)
    set(SWIG_SHA512 "b8f105f9b9db6acc1f6e3741990915b533cd1bc206eb9645fd6836457fd30789b7229d2e3219d8e35f2390605ade0fbca493ae162ec3b4bc4e428b57155db03d")
    set(SWIG_URL "https://simpleitk.org/SimpleITKExternalData/SHA512/${SWIG_SHA512}")
    set(SWIG_FILENAME "swigwin-4.0.2.zip")
    set(SWIG_HASH "SHA512=${SWIG_SHA512}")
    ExternalProject_add(SWIG
      SOURCE_DIR ${CMAKE_BINARY_DIR}/SWIG
      URL ${SWIG_URL}
      URL_HASH ${SWIG_HASH}
      DOWNLOAD_NAME ${SWIG_FILENAME}
      DOWNLOAD_DIR ${CMAKE_BINARY_DIR}
      USES_TERMINAL_DOWNLOAD 1
      CONFIGURE_COMMAND ""
      BUILD_COMMAND ""
      BUILD_IN_SOURCE 1
      INSTALL_COMMAND ""
      )
    set(SWIG_EXECUTABLE ${CMAKE_BINARY_DIR}/SWIG/swig.exe)
    message(STATUS "SuperBuild -     SWIG[OK]")
    message(STATUS "SuperBuild -     SWIG_URL: ${LeapSDK_URL}")
    message(STATUS "SuperBuild -     SWIG_FILENAME: ${LeapSDK_FILENAME}")
    message(STATUS "SuperBuild -     SWIG_HASH: ${LeapSDK_HASH}")
    message(STATUS "SuperBuild -     SWIG_EXECUTABLE: ${SWIG_EXECUTABLE}")
  else()
    lmpd_ExternalProject_Add_Empty(SWIG "")
    find_package(SWIG 3.0 REQUIRED)
    message(STATUS "SuperBuild -     SWIG[OK]")
    message(STATUS "SuperBuild -     SWIG_EXECUTABLE: ${SWIG_EXECUTABLE}")
  endif()

  #
  # LeapCxx
  #
  ExternalProject_add(LeapCxx
    SOURCE_DIR LeapCxx
    BINARY_DIR ${CMAKE_BINARY_DIR}/LeapCxx-build
    GIT_REPOSITORY git://github.com/Slicer/LeapCxx
    GIT_TAG 79b9909f03dd3fff94b84bb395fe7bd9a2a65b60 # leapmotion-python-distributions-0.1.0-2018-08-08-bc659293c
    ${_ep_common_arguments}
    CMAKE_CACHE_ARGS
      -DBUILD_PYTHON_SWIG_BINDINGS:BOOL=ON
      -DBUILD_SAMPLES:BOOL=OFF
      -DBUILD_TESTING:BOOL=OFF
      -DLeapSDK_DIR:PATH=${LeapSDK_DIR}
      -DSWIG_EXECUTABLE:FILEPATH=${SWIG_EXECUTABLE}
    USES_TERMINAL_DOWNLOAD 1
    USES_TERMINAL_CONFIGURE 1
    USES_TERMINAL_BUILD 1
    INSTALL_COMMAND ""
    DEPENDS
      LeapSDK-binary-download
      SWIG
    )
  set(LeapCxx_DIR "${CMAKE_BINARY_DIR}/LeapCxx-build")

  message(STATUS "SuperBuild -   LeapCxx[OK]")
  message(STATUS "SuperBuild -   LeapCxx_DIR: ${LeapCxx_DIR}")

  #
  # Inner
  #
  ExternalProject_add(${PROJECT_NAME}
    SOURCE_DIR ${CMAKE_SOURCE_DIR}
    BINARY_DIR ${CMAKE_BINARY_DIR}/${PROJECT_NAME}-build
    DOWNLOAD_COMMAND ""
    UPDATE_COMMAND ""
    ${_ep_common_arguments}
    CMAKE_CACHE_ARGS
      -D${PROJECT_NAME}_SUPERBUILD:BOOL=0
      -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
      -DLeapCxx_DIR:PATH=${LeapCxx_DIR}
    USES_TERMINAL_CONFIGURE 1
    USES_TERMINAL_BUILD 1
    INSTALL_COMMAND ""
    DEPENDS
      LeapCxx
    )

  message(STATUS "SuperBuild - ${PROJECT_NAME}[OK]")

else()

  # Install LeapCxx
  message(STATUS "Adding install rules for components:")
  foreach(component IN ITEMS RuntimeLibraries)
    message(STATUS "  ${component}")
    install(CODE "
unset(CMAKE_INSTALL_COMPONENT)
set(COMPONENT \"${component}\")
set(CMAKE_INSTALL_DO_STRIP 1)
message(STATUS \"Install component '${component}' of LeapCxx project\")
include\(\"${LeapCxx_DIR}/cmake_install.cmake\")
unset(CMAKE_INSTALL_COMPONENT)
")
  endforeach()

endif()
