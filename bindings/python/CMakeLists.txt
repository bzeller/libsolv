IF (ENABLE_PYTHON3 AND NOT DEFINED PythonLibs_FIND_VERSION)
    # if we build both for python2 and python3, make this the python2 build.
    # see comment in the python3 CMakeLists.txt file
    SET (PythonLibs_FIND_VERSION 2)
    SET (PythonLibs_FIND_VERSION_MAJOR 2)
ENDIF (ENABLE_PYTHON3 AND NOT DEFINED PythonLibs_FIND_VERSION)

FIND_PACKAGE (PythonLibs REQUIRED)
IF(PYTHONLIBS_VERSION_STRING MATCHES "^([0-9.]+)")
    SET(python_version "${CMAKE_MATCH_1}")
ELSE()
    MESSAGE(FATAL_ERROR "PythonLibs version format unknown '${PYTHONLIBS_VERSION_STRING}'")
ENDIF()
FIND_PACKAGE (PythonInterp ${python_version} REQUIRED)

EXECUTE_PROCESS(COMMAND ${PYTHON_EXECUTABLE} -c "from sys import stdout; import sysconfig; stdout.write(sysconfig.get_path('platlib'))" OUTPUT_VARIABLE PYTHON_INSTALL_DIR)

IF (NOT DEFINED PYTHON_VERSION_MAJOR)
    SET (PYTHON_VERSION_MAJOR 2)
ENDIF (NOT DEFINED PYTHON_VERSION_MAJOR)
IF (${PYTHON_VERSION_MAJOR} GREATER 2)
    SET (SWIG_PY_FLAGS -DPYTHON3=1)
ENDIF (${PYTHON_VERSION_MAJOR} GREATER 2)
SET (SWIG_PY_FLAGS ${SWIG_PY_FLAGS} -DSWIG_PYTHON_LEGACY_BOOL=1)

MESSAGE (STATUS "Python executable: ${PYTHON_EXECUTABLE}")
MESSAGE (STATUS "Python installation dir: ${PYTHON_INSTALL_DIR}")
MESSAGE (STATUS "Python include path: ${PYTHON_INCLUDE_PATH}")

ADD_CUSTOM_COMMAND (
    OUTPUT solv_python.c
    COMMAND ${SWIG_EXECUTABLE} ${SWIG_FLAGS} -python ${SWIG_PY_FLAGS} -I${CMAKE_SOURCE_DIR}/src -o solv_python.c ${CMAKE_SOURCE_DIR}/bindings/solv.i
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    DEPENDS ${CMAKE_SOURCE_DIR}/bindings/solv.i
)

ADD_DEFINITIONS(-Wno-unused)
INCLUDE_DIRECTORIES (${PYTHON_INCLUDE_PATH})

ADD_LIBRARY (bindings_python MODULE solv_python.c)
SET_TARGET_PROPERTIES (bindings_python PROPERTIES PREFIX "" OUTPUT_NAME "_solv")
TARGET_LINK_LIBRARIES (bindings_python ${LIBSOLV_BINDINGS_LIBRARIES} ${SYSTEM_LIBRARIES})

INSTALL (TARGETS bindings_python LIBRARY DESTINATION ${PYTHON_INSTALL_DIR})
INSTALL (FILES ${CMAKE_CURRENT_BINARY_DIR}/solv.py DESTINATION ${PYTHON_INSTALL_DIR})

