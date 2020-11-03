cmake_minimum_required(VERSION 3.10)

project(build_libtorch)

set(PYTORCH_SRC_DIR ../pytorch)

set(ENV{DEBUG} 1) #will enable debug builds (-g -O0)
set(ENV{BUILD_TEST} 0) # will disable building C++ test binaries
set(ENV{USE_CUDA} 0) #disable compiling CUDA, to save compile time
set(ENV{USE_DISTRIBUTED} 0) #disable distributed (c10d, gloo, mpi, etc.) build
set(ENV{USE_MKLDNN} 0) #disable using MKL-DNN
set(ENV{USE_FBGEMM} 0) #disable using FBGEMM (quantized 8-bit server operators)
set(ENV{USE_NNPACK} 0) #disable compiling with NNPACK
set(ENV{USE_QNNPACK} 0) #disable QNNPACK build (quantized 8-bit operators)
set(ENV{USE_XNNPACK} 0) #disable compiling with XNNPACK
set(ENV{BUILD_CAFFE2_OPS} 0) #Build Caffe2 operators

if(NOT MSVC)
  file(READ ${PYTORCH_SRC_DIR}/CMakeLists.txt FILE_CONTENTS)
  if($ENV{DEBUG})
    string(FIND "${FILE_CONTENTS}" "-O2" POS)
    if(NOT ${POS} EQUAL -1)
      string(REPLACE "-O2" "-Og" FILE_CONTENTS "${FILE_CONTENTS}")
      file(WRITE ${PYTORCH_SRC_DIR}/CMakeLists.txt "${FILE_CONTENTS}")
    endif()
  else()
    string(FIND "${FILE_CONTENTS}" "-Og" POS)
    if(NOT ${POS} EQUAL -1)
      string(REPLACE "-O2" "-Og" FILE_CONTENTS "${FILE_CONTENTS}")
    file(WRITE ${PYTORCH_SRC_DIR}/CMakeLists.txt "${FILE_CONTENTS}")
    endif()
  endif()
endif()

file(READ ${PYTORCH_SRC_DIR}/cmake/Dependencies.cmake FILE_CONTENTS)
string(FIND "${FILE_CONTENTS}" "find_package(PythonLibs 3.0)" POS)
if(NOT ${POS} EQUAL -1)
  string(REPLACE "find_package(PythonLibs 3.0)" "find_package(Python COMPONENTS Interpreter)" FILE_CONTENTS "${FILE_CONTENTS}")
  string(REPLACE "PYTHONLIBS_VERSION_STRING" "Python_VERSION" FILE_CONTENTS "${FILE_CONTENTS}")
  file(WRITE ${PYTORCH_SRC_DIR}/cmake/Dependencies.cmake "${FILE_CONTENTS}")
endif()

find_package(Python)
if(Python_Interpreter_FOUND)
  execute_process(COMMAND ${Python_EXECUTABLE} setup.py build --cmake-only WORKING_DIRECTORY ${PYTORCH_SRC_DIR})
endif()
