cmake_minimum_required(VERSION 2.8.3)
project(cloudwatch_logger)

## Compile as C++11, supported in ROS Kinetic and newer
add_compile_options(-std=c++11)

find_package(cloudwatch_logs_common REQUIRED)

## Find catkin macros and libraries
find_package(catkin REQUIRED COMPONENTS
    cloudwatch_logs_common
    roscpp
    aws_common
    aws_ros1_common
    rosgraph_msgs)

###################################
## catkin specific configuration ##
###################################

catkin_package()

###########
## Build ##
###########

## Declare a C++ executable
add_library(${PROJECT_NAME}_lib STATIC src/log_node.cpp src/log_node_param_helper.cpp)
target_include_directories(${PROJECT_NAME}_lib PUBLIC include ${catkin_INCLUDE_DIRS} ${cloudwatch_logs_common_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME}_lib ${catkin_LIBRARIES} ${cloudwatch_logs_common_LIBRARIES})

add_executable(${PROJECT_NAME} src/main.cpp)
target_include_directories(${PROJECT_NAME} PRIVATE include ${catkin_INCLUDE_DIRS} ${cloudwatch_logs_common_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} ${PROJECT_NAME}_lib ${cloudwatch_logs_common_LIBRARIES})

#############
## Install ##
#############
install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
        
install(DIRECTORY config launch
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
if(CATKIN_ENABLE_TESTING)
  find_package(rostest REQUIRED)
  find_package(GMock QUIET)
  if(GMOCK_FOUND)
      add_rostest_gmock(test_log_node test/test_log_node.test test/log_node_test.cpp)
      target_include_directories(test_log_node PRIVATE include ${catkin_INCLUDE_DIRS} ${cloudwatch_logger_INCLUDE_DIRS} ${GMOCK_INCLUDE_DIRS})
      target_link_libraries(test_log_node ${PROJECT_NAME}_lib ${GMOCK_BOTH_LIBRARIES} ${catkin_LIBRARIES})

      add_rostest_gmock(test_log_node_param_helper test/test_log_node_param_helper.test test/log_node_param_helper_test.cpp)
      target_include_directories(test_log_node_param_helper PRIVATE include ${catkin_INCLUDE_DIRS} ${cloudwatch_logger_INCLUDE_DIRS} ${GMOCK_INCLUDE_DIRS})
      target_link_libraries(test_log_node_param_helper ${PROJECT_NAME}_lib ${GMOCK_BOTH_LIBRARIES} ${catkin_LIBRARIES})
  else()
      include_directories(/usr/include/gmock /usr/src/gmock)
      add_library(${PROJECT_NAME}_libgmock /usr/src/gmock/src/gmock-all.cc)
  
      add_rostest_gtest(test_log_node test/test_log_node.test test/log_node_test.cpp)
      target_include_directories(test_log_node PRIVATE include ${catkin_INCLUDE_DIRS} ${cloudwatch_logger_INCLUDE_DIRS})
      target_link_libraries(test_log_node ${PROJECT_NAME}_lib ${PROJECT_NAME}_libgmock ${catkin_LIBRARIES})

      add_rostest_gtest(test_log_node_param_helper test/test_log_node_param_helper.test test/log_node_param_helper_test.cpp)
      target_include_directories(test_log_node_param_helper PRIVATE include ${catkin_INCLUDE_DIRS} ${cloudwatch_logger_INCLUDE_DIRS})
      target_link_libraries(test_log_node_param_helper ${PROJECT_NAME}_lib ${PROJECT_NAME}_libgmock ${catkin_LIBRARIES})
  endif()
endif()
