# Python 3 modules for LeapMotion

CMake project for building LeapMotion Python 3 modules.

## Getting started

The instructions below assumes you have a Slicer build tree compiled using `Visual Studio 17 2022`.

* Set variables

  ```
  set CMAKE_EXECUTABLE=C:/cmake-3.22.1/bin/cmake.exe
  set Slicer_BUILD_DIR=D:/D/S4R
  set WORK_DIR=D:/T/

  set LEAPMOTION_PYTHON_DISTRIBUTIONS_SOURCE_DIR=%WORK_DIR%/leapmotion-python-distributions
  set LEAPMOTION_PYTHON_DISTRIBUTIONS_BUILD_DIR=%WORK_DIR%/leapmotion-python-distributions-Release
  set LEAPMOTION_PYTHON_DISTRIBUTIONS_INSTALL_DIR=%WORK_DIR%/leapmotion-python-distributions-install
  ```

* Download source

  ```
  cd D:\T
  git clone git@github.com:Slicer/leapmotion-python-distributions.git %LEAPMOTION_PYTHON_DISTRIBUTIONS_SOURCE_DIR%
  ```

* Configure project

  ```
  %CMAKE_EXECUTABLE% ^
    -G "Visual Studio 17 2022" ^
    -A x64 ^
    -T v143 ^
    -DCMAKE_INSTALL_PREFIX:PATH=%LEAPMOTION_PYTHON_DISTRIBUTIONS_INSTALL_DIR% ^
    -DSWIG_DIR:PATH=%Slicer_BUILD_DIR%/swigwin-4.0.2/Lib  ^
    -DSWIG_EXECUTABLE:PATH=%Slicer_BUILD_DIR%/swigwin-4.0.2/swig.exe ^
    -DPYTHON_EXECUTABLE:FILEPATH=%Slicer_BUILD_DIR%/python-install/bin/PythonSlicer.exe ^
    -DPYTHON_INCLUDE_DIR:PATH=%Slicer_BUILD_DIR%/python-install/include ^
    -DPYTHON_LIBRARY:FILEPATH=%Slicer_BUILD_DIR%/python-install/libs/python39.lib ^
    -S %LEAPMOTION_PYTHON_DISTRIBUTIONS_SOURCE_DIR% ^
    -B %LEAPMOTION_PYTHON_DISTRIBUTIONS_BUILD_DIR%
  ```

* Build

  ```
  %CMAKE_EXECUTABLE% ^
    --build %LEAPMOTION_PYTHON_DISTRIBUTIONS_BUILD_DIR% ^
    --config Release ^
    -- /maxcpucount:4
  ```

* Install

  ```
  %CMAKE_EXECUTABLE% ^
    --build %LEAPMOTION_PYTHON_DISTRIBUTIONS_BUILD_DIR%/LeapMotionPythonDistributions-build ^
    --config Release ^
    --target INSTALL ^
    -- /maxcpucount:4 

  ```

  Example of output:

  ```
    -- Install configuration: "Release"
    -- Installing: D:/T/leapmotion-python-distributions-install/lib/LeapC.dll
    -- Install component: "RuntimeLibraries"
    -- Installing: D:/T/leapmotion-python-distributions-install/lib/Leap.py
    -- Installing: D:/T/leapmotion-python-distributions-install/lib/_LeapPython.pyd
  ```

* Verify

  ```
  %Slicer_BUILD_DIR%/\python-install\bin\PythonSlicer.exe ^
    -c "import sys; sys.path.insert(0, 'D:/T/leapmotion-python-distributions-install/lib/'); import Leap; from pprint import pprint as pp; pp(dir(Leap))"
  ```

  Expected output:

  ```python
  ['Arm',
   'Bone',
   'Config',
   'Controller',
   'DEG_TO_RAD',
   'Device',
   'DeviceList',
   'EPSILON',
   'FailedDevice',
   'FailedDeviceList',
   'FailedDevice_invalid',
   'Finger',
   'FingerList',
   'Frame',
   'Hand',
   'HandList',
   'HeadPose',
   'Image',
   'ImageList',
   'Interface',
   'Listener',
   'MESSAGE_CRITICAL',
   'MESSAGE_INFORMATION',
   'MESSAGE_UNKNOWN',
   'MESSAGE_WARNING',
   'MapPoint',
   'MapPointList',
   'MapPoint_invalid',
   'Matrix',
   'PI',
   'Quaternion',
   'Quaternion_zero',
   'RAD_TO_DEG',
   'SwigPyIterator',
   'Vector',
   '_LeapPython',
   '_SwigNonDynamicMeta',
   '__builtin__',
   '__builtins__',
   '__cached__',
   '__doc__',
   '__file__',
   '__loader__',
   '__name__',
   '__package__',
   '__spec__',
   '_swig_add_metaclass',
   '_swig_python_version_info',
   '_swig_repr',
   '_swig_setattr_nondynamic_class_variable',
   '_swig_setattr_nondynamic_instance_variable',
   'byte_array',
   'byte_array_frompointer',
   'cvar',
   'float_array',
   'float_array_frompointer',
   'weakref']
  ```

## Licenses

This project is distributed under the Apache 2.0 license. Please see the _LICENSE_ file for details.

The LeapCXX project is distributed under the Apache 2.0 license. Please, see [LICENSE](https://github.com/leapmotion/LeapCxx/blob/master/LICENSE.txt) file for details.

Use subject to the terms of the Leap Motion SDK Agreement available at https://developer.leapmotion.com/sdk_agreement.
