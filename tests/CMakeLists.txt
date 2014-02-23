
if (Qt5Core_FOUND)
    find_package(Qt5Test)
endif()

set(testdir ${CMAKE_BINARY_DIR}/TestDir)
file(MAKE_DIRECTORY ${testdir})
add_definitions(-DTEST_DIR=${testdir})

set(existing_repository ${testdir}/existing_repository)
add_definitions(-DTEST_EXISTING_REPOSITORY=${existing_repository})
if(NOT EXISTS ${existing_repository})
    message(STATUS "Creating test repository ...")
    file(MAKE_DIRECTORY ${existing_repository})
    file(MAKE_DIRECTORY ${existing_repository}/.git)
    file(GLOB_RECURSE gitfiles RELATIVE ${CMAKE_SOURCE_DIR}/.git ${CMAKE_SOURCE_DIR}/.git/*)
    foreach(gitfile ${gitfiles})
        configure_file(${CMAKE_SOURCE_DIR}/.git/${gitfile} ${existing_repository}/.git/${gitfile} COPYONLY)
    endforeach()
endif()
message(STATUS "Using test repository at '${existing_repository}'")


include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(EXECUTABLE_OUTPUT_PATH  ${CMAKE_BINARY_DIR}/bin)

macro(addTest name)
    set(_source ${CMAKE_CURRENT_SOURCE_DIR}/${name}.cpp)
    set(_moced ${CMAKE_CURRENT_BINARY_DIR}/${name}.moc)
    set_source_files_properties(${_source} PROPERTIES OBJECT_DEPENDS "${_moced}")
    set(testname test${name})
    add_executable(${testname} ${_source})
    target_link_libraries(test${name} qgit2 git2 ${QT_QTCORE_LIBRARY} ${QT_QTTEST_LIBRARY})
    add_test(NAME ${testname} COMMAND ${testname})
    if(Qt5Core_FOUND)
        qt5_use_modules(test${name} Test)
    else()
        qt4_generate_moc(${_source} ${_moced})
    endif()
endmacro()


addTest(Init)
addTest(Revision)
addTest(StatusOptions)