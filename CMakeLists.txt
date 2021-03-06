cmake_minimum_required(VERSION 2.8.3)
project(labelizer)

add_compile_options(-std=c++11)

set(catkin_RUN_PACKAGES
	cv_bridge
	rospy
	roscpp
	roslib
	rqt_gui
	rqt_gui_cpp
)

set(catkin_BUILD_PACKAGES
	${catkin_RUN_PACKAGES}
	cmake_modules
)

## Find catkin macros and libraries
find_package(catkin REQUIRED COMPONENTS
	${catkin_BUILD_PACKAGES}
)


## Find several needed libraries
# Qt development library
if("${qt_gui_cpp_USE_QT_MAJOR_VERSION} " STREQUAL "5 ")
  find_package(Qt5Widgets REQUIRED)
else()
  find_package(Qt4 COMPONENTS QtCore QtGui REQUIRED)
  include(${QT_USE_FILE})
endif()
# pkg-config package to include packages that don't provide a cmake config file
find_package(PkgConfig REQUIRED)
# jsoncpp --> json parser
pkg_check_modules(JSONCPP jsoncpp)
link_libraries(${JSONCPP_LIBRARIES})

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if you package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
# important: common/include needs to be here if you have such a directory
catkin_package(
INCLUDE_DIRS
LIBRARIES
	${PROJECT_NAME}
CATKIN_DEPENDS
	${catkin_RUN_PACKAGES}
DEPENDS
)

catkin_python_setup()

set(rqt_cpp_SRCS
  src/labelizer/labelizer_plugin.cpp
  src/labelizer/mouse_q_scene.cpp
)

set(rqt_cpp_HDRS
  include/labelizer/labelizer_plugin.hpp
  include/labelizer/mouse_q_scene.hpp
)

set(rqt_cpp_UIS
  include/labelizer/labelizer_form.ui
)

if("${qt_gui_cpp_USE_QT_MAJOR_VERSION} " STREQUAL "5 ")
  qt5_wrap_cpp(rqt_cpp_MOCS ${rqt_cpp_HDRS})
  qt5_wrap_ui(rqt_cpp_UIS_HPP ${rqt_cpp_UIS})
else()
  qt4_wrap_cpp(rqt_cpp_MOCS ${rqt_cpp_HDRS})
  qt4_wrap_ui(rqt_cpp_UIS_HPP ${rqt_cpp_UIS})
endif()

# Add custom (non compiling) targets so launch scripts and python files show up in QT Creator's project view.
file(GLOB_RECURSE EXTRA_FILES */*)
add_custom_target(${PROJECT_NAME}_OTHER_FILES ALL WORKING_DIRECTORY ${PROJECT_SOURCE_DIR} SOURCES ${EXTRA_FILES})

###########
## Build ##
###########
## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
	include
	${CMAKE_CURRENT_BINARY_DIR}/..
	${catkin_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME}
	${rqt_cpp_SRCS}
	${rqt_cpp_MOCS}
	${rqt_cpp_UIS_HPP}
)

target_link_libraries(${PROJECT_NAME}
  ${catkin_LIBRARIES}
  ${JSONCPP_LIBRARIES}
)
if("${qt_gui_cpp_USE_QT_MAJOR_VERSION} " STREQUAL "5 ")
  target_link_libraries(${PROJECT_NAME} Qt5::Widgets)
else()
  target_link_libraries(${PROJECT_NAME} ${QT_QTCORE_LIBRARY} ${QT_QTGUI_LIBRARY})
endif()

find_package(class_loader)
class_loader_hide_library_symbols(${PROJECT_NAME})

#############
## Install ##
#############
# Mark executables and/or libraries for installation
#install(TARGETS room_exploration_server room_exploration_client
#	ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#	LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#	RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#)

#uncomment this if you have a common-directory with header-files
#install(DIRECTORY common/include/${PROJECT_NAME}/
#	DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#	FILES_MATCHING PATTERN "*.h"
#	PATTERN ".svn" EXCLUDE
#)

#uncomment this if you have header-files in your project
#install(DIRECTORY ros/include/${PROJECT_NAME}/
#	DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#	FILES_MATCHING PATTERN "*.h"
#	PATTERN ".svn" EXCLUDE
#)

#install(DIRECTORY scripts
#	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
#	PATTERN ".svn" EXCLUDE
#)

# uncomment this if you have launch files
#install(DIRECTORY ros/launch
#	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/ros
#	PATTERN ".svn" EXCLUDE
#)

#install(DIRECTORY common/files
#	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/common
#	PATTERN ".svn" EXCLUDE
#)
