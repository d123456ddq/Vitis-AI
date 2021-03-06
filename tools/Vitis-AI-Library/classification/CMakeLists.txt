#
# Copyright 2019 Xilinx Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
cmake_minimum_required(VERSION 3.5)
get_filename_component(COMPONENT_NAME "${CMAKE_CURRENT_SOURCE_DIR}" NAME)

include(XilinxCommon)
include(XilinxVersion)

add_library(${COMPONENT_NAME} SHARED
  include/vitis/ai/classification.hpp
  src/classification.cpp
  src/classification_imp.cpp
  src/classification_imp.hpp
  ${CMAKE_CURRENT_BINARY_DIR}/version.c
  )

add_library(${PROJECT_NAME}::${COMPONENT_NAME} ALIAS ${COMPONENT_NAME})

target_link_libraries(${COMPONENT_NAME} ${PROJECT_NAME}::xnnpp ${PROJECT_NAME}::dpu_task ${PROJECT_NAME}::model_config ${OpenCV_LIBS} ${PROJECT_NAME}::math)

target_include_directories(${COMPONENT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

target_include_directories(${COMPONENT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
  $<INSTALL_INTERFACE:include>
)

set_target_properties(${COMPONENT_NAME} PROPERTIES
  VERSION "${PROJECT_VERSION}"
  SOVERSION "${PROJECT_VERSION_MAJOR}"
  LIBRARY_OUTPUT_NAME  ${PROJECT_NAME}-${COMPONENT_NAME}
  )

install(
  TARGETS ${COMPONENT_NAME}
  EXPORT ${COMPONENT_NAME}-targets
  RUNTIME DESTINATION ${INSTALL_BIN_DIR}
  LIBRARY DESTINATION ${INSTALL_LIB_DIR})

install(
  EXPORT ${COMPONENT_NAME}-targets
  NAMESPACE ${PROJECT_NAME}::
  DESTINATION ${INSTALL_CMAKE_DIR})

install(FILES include/vitis/ai/classification.hpp DESTINATION include/vitis/ai)

add_executable(test_classification test/test_classification.cpp)
add_executable(test_classification_batch test/test_classification_batch.cpp)
add_executable(test_classification_accuracy test/test_accuracy_classification.cpp)
add_executable(test_classification_speed test/test_classification_speed.cpp)
add_executable(test_classification_accuracy_mt test/test_accuracy_classification_mt.cpp)
target_link_libraries(test_classification ${COMPONENT_NAME} ${OpenCV_LIBS})
target_link_libraries(test_classification_batch ${COMPONENT_NAME} ${OpenCV_LIBS})
target_link_libraries(test_classification_accuracy ${COMPONENT_NAME} ${OpenCV_LIBS} protobuf::libprotobuf)
target_link_libraries(test_classification_speed ${COMPONENT_NAME} ${OpenCV_LIBS})
target_link_libraries(test_classification_accuracy_mt ${COMPONENT_NAME} ${OpenCV_LIBS} -lpthread -lglog protobuf::libprotobuf)
