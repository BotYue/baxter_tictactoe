# Copyright: (C) 2016 Social Robotics Lab - Yale University
# Authors: Alessandro Roncone, Olivier Mangin, Álvaro Castro González
# CopyPolicy: Released under the terms of the GNU GPL v3.0.
cmake_minimum_required(VERSION 2.8.3)
project(baxter_tictactoe)

set(COMPILE_TESTS  false)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
             baxter_collaboration_lib
             baxter_collaboration_msgs
             baxter_core_msgs
             cv_bridge
             image_transport
             message_generation
             rosconsole
             sound_play
             std_msgs
)

find_package(Qt4 REQUIRED COMPONENTS QtCore QtGui)
find_package(OpenCV 2.4 REQUIRED)
find_package(Boost REQUIRED COMPONENTS thread)
find_package(Threads)

## Uncomment this if the package has a setup.py. This macro ensures
## modules and global scripts declared therein get installed
## See http://ros.org/doc/api/catkin/html/user_guide/setup_dot_py.html
# catkin_python_setup()

################################################
## Declare ROS messages, services and actions ##
################################################

## To declare and build messages, services or actions from within this
## package, follow these steps:
## * Let MSG_DEP_SET be the set of packages whose message types you use in
##   your messages/services/actions (e.g. std_msgs, actionlib_msgs, ...).
## * In the file package.xml:
##   * add a build_depend tag for "message_generation"
##   * add a build_depend and a run_depend tag for each package in MSG_DEP_SET
##   * If MSG_DEP_SET isn't empty the following dependency has been pulled in
##     but can be declared for certainty nonetheless:
##     * add a run_depend tag for "message_runtime"
## * In this file (CMakeLists.txt):
##   * add "message_generation" and every package in MSG_DEP_SET to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * add "message_runtime" and every package in MSG_DEP_SET to
##     catkin_package(CATKIN_DEPENDS ...)
##   * uncomment the add_*_files sections below as needed
##     and list every .msg/.srv/.action file to be processed
##   * uncomment the generate_messages entry below
##   * add every package in MSG_DEP_SET to generate_messages(DEPENDENCIES ...)

## Generate messages in the 'msg' folder
add_message_files(FILES
    MsgBoard.msg
    MsgCell.msg
    TTTBrainState.msg
)

## Generate added messages and services with any dependencies listed here
generate_messages(DEPENDENCIES
    baxter_core_msgs
    geometry_msgs
    std_msgs
)

################################################
## Declare ROS dynamic reconfigure parameters ##
################################################

## To declare and build dynamic reconfigure parameters within this
## package, follow these steps:
## * In the file package.xml:
##   * add a build_depend and a run_depend tag for "dynamic_reconfigure"
## * In this file (CMakeLists.txt):
##   * add "dynamic_reconfigure" to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * uncomment the "generate_dynamic_reconfigure_options" section below
##     and list every .cfg file to be processed

## Generate dynamic reconfigure parameters in the 'cfg' folder
# generate_dynamic_reconfigure_options(
#   cfg/DynReconf1.cfg
#   cfg/DynReconf2.cfg
# )

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if you package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
    INCLUDE_DIRS lib/include
    LIBRARIES baxter_tictactoe
    CATKIN_DEPENDS message_runtime
)

###########
## Build ##
###########

ADD_DEFINITIONS ( -Wall -Wextra -Wno-unused-parameter -Werror)

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(lib/include
                    ${OpenCV_INCLUDE_DIRS}
                    ${Boost_INCLUDE_DIRS}
                    ${eigen_INCLUDE_DIRS}
                    ${catkin_INCLUDE_DIRS}
                    ${CMAKE_CURRENT_SOURCE_DIR}
)

include(${QT_USE_FILE})
add_definitions(${QT_DEFINITIONS})

## Add libraries
add_subdirectory(lib)

## Declare a C++ executables
add_executable(tictactoe_brain            src/tictactoe_brain/tictactoeBrain.h
                                          src/tictactoe_brain/tictactoeBrain.cpp
                                          src/tictactoe_brain/tictactoe_brain.cpp)
add_executable(hsv_range_finder           src/hsv_range_finder/hsv_range_finder.cpp)
add_executable(baxterDisplay              src/baxterDisplay/baxterDisplay.cpp)
add_executable(board_state_sensor         src/board_state_sensor/boardState.h
                                          src/board_state_sensor/boardState.cpp
                                          src/board_state_sensor/board_state_sensor.cpp)

## Add cmake target dependencies of the executable
add_dependencies(tictactoe_brain          baxter_tictactoe_generate_messages_cpp
                                          ${${PROJECT_NAME}_EXPORTED_TARGETS}
                                          ${catkin_EXPORTED_TARGETS})
add_dependencies(hsv_range_finder         baxter_tictactoe_generate_messages_cpp
                                          ${${PROJECT_NAME}_EXPORTED_TARGETS}
                                          ${catkin_EXPORTED_TARGETS})
add_dependencies(baxterDisplay            baxter_tictactoe_generate_messages_cpp
                                          ${${PROJECT_NAME}_EXPORTED_TARGETS}
                                          ${catkin_EXPORTED_TARGETS})
add_dependencies(board_state_sensor       baxter_tictactoe_generate_messages_cpp
                                          ${${PROJECT_NAME}_EXPORTED_TARGETS}
                                          ${catkin_EXPORTED_TARGETS})

## Specify libraries to link a library or executable target against
target_link_libraries(tictactoe_brain      baxter_tictactoe
                                           ${catkin_LIBRARIES}
                                           ${CMAKE_THREAD_LIBS_INIT})
target_link_libraries(hsv_range_finder     baxter_tictactoe
                                           ${OpenCV_LIBS}
                                           ${catkin_LIBRARIES})
target_link_libraries(baxterDisplay        ${OpenCV_LIBS}
                                           ${catkin_LIBRARIES})
target_link_libraries(board_state_sensor   baxter_tictactoe
                                           ${OpenCV_LIBS}
                                           ${QT_LIBRARIES}
                                           ${catkin_LIBRARIES})

# Compile tests if required
IF(COMPILE_TESTS STREQUAL true)
    add_executable(test_vacuum_gripper          test/test_vacuum_gripper.cpp)
    add_executable(test_ttt_controller          test/test_ttt_controller.cpp)

    add_dependencies(test_vacuum_gripper        baxter_tictactoe_generate_messages_cpp)
    add_dependencies(test_ttt_controller        baxter_tictactoe_generate_messages_cpp)

    target_link_libraries(test_vacuum_gripper                     ${catkin_LIBRARIES})
    target_link_libraries(test_ttt_controller   baxter_tictactoe  ${catkin_LIBRARIES})
#ELSE()
#    message(${PROJECT_NAME} ": Tests will not be compiled")
ENDIF()

#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation

#install(TARGETS ${PROJECT_NAME}
#  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#)

# install(DIRECTORY include/
#     DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
# )

## Mark other files for installation (e.g. launch and bag files, etc.)
install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
)

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
if(CATKIN_ENABLE_TESTING)
  # find_package(rostest            REQUIRED)
  # find_package(baxter_description REQUIRED)

  catkin_add_gtest(test_utils test/test_utils_lib.cpp)
  add_dependencies(test_utils   baxter_tictactoe_generate_messages_cpp
                                ${${PROJECT_NAME}_EXPORTED_TARGETS}
                                ${catkin_EXPORTED_TARGETS})
  target_link_libraries(test_utils ${PROJECT_NAME} ${catkin_LIBRARIES})

  # add_rostest_gtest(test_ipopt test/test_ipopt.launch test/test_ipopt.cpp)
  # target_link_libraries(test_ipopt  react_controller ${catkin_LIBRARIES})
endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)
