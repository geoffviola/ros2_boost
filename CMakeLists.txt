cmake_minimum_required(VERSION 3.5)
project(ros2_boost)

find_package(ament_cmake REQUIRED)

# add_custom_target(setup_git_boost ALL
  # ${CMAKE_COMMAND} -E make_directory boost)
# add_custom_target(git_boost_init ALL 
    # DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/boost/.git/config")
# add_dependencies(git_boost_init setup_git_boost)
# add_custom_command(
    # OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/boost/.git/config"
    # COMMAND git init
    # WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/boost")
# add_custom_target(git_boost_set_remote ALL 
    # DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/BOOST_GIT_REMOTE_SET.txt")
# add_dependencies(git_boost_set_remote git_boost_init)
# add_custom_command(
    # OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/BOOST_GIT_REMOTE_SET.txt"
    # COMMAND git remote add --no-tags -t master origin 
            # https://github.com/boostorg/boost.git
    # COMMAND echo placeholder 
            # >> "${CMAKE_CURRENT_BINARY_DIR}/BOOST_GIT_REMOTE_SET.txt"
    # WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/boost")
# add_custom_target(git_boost ALL
  # COMMAND git fetch --depth=1
  # COMMAND git checkout master
  # COMMAND git submodule update --init --merge
  # COMMAND git remote set-branches --add origin master
  # COMMAND git pull --recurse-submodules
  # COMMAND git submodule update --init
  # COMMAND git checkout master
  # COMMAND echo "git checkout master finished"
  # COMMAND git submodule foreach "git reset --quiet --hard; git clean -fxd"
  # COMMAND git reset --hard
  # COMMAND git clean -fxd
  # COMMENT "git boost"
  # WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/boost"
  # DEPENDS git_boost_set_remote)
# add_dependencies(git_boost git_boost_set_remote)

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/clone_boost_ignore_error.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/clone_boost_ignore_error.cmake
  COPYONLY
)
add_custom_target(git_boost_clone ALL
    ${CMAKE_COMMAND} -P 
    "${CMAKE_CURRENT_BINARY_DIR}/clone_boost_ignore_error.cmake")
add_custom_target(git_boost_checkout ALL
    git submodule foreach --recursive git checkout
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/boost")
add_dependencies(git_boost_checkout git_boost_clone)

add_custom_target(bootstrap_boost ALL
    DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/boost/b2${CMAKE_EXECUTABLE_SUFFIX}")
if (WIN32)
  set(SHELL_SUFFIX ".bat")
else ()
  set(SHELL_SUFFIX ".sh")
endif ()
add_custom_command(
    OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/boost/b2${CMAKE_EXECUTABLE_SUFFIX}"
    COMMAND "${CMAKE_CURRENT_BINARY_DIR}/boost/bootstrap${SHELL_SUFFIX}"
    COMMENT "git bootstrap"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/boost")
add_dependencies(bootstrap_boost git_boost_checkout)

add_custom_target(build_boost ALL
    "${CMAKE_CURRENT_BINARY_DIR}/boost/b2${CMAKE_EXECUTABLE_SUFFIX}" headers
    DEPENDS bootstrap_boost
    COMMENT "build boost"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/boost")
add_custom_target(copy_boost_includes ALL
    ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_BINARY_DIR}/boost/boost" 
        "${CMAKE_CURRENT_BINARY_DIR}/boost/include/boost"
    DEPENDS build_boost
    COMMENT "copy boost includes")

install(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/boost/include/" DESTINATION include)

ament_package()