# SPDX-License-Identifier: Apache-2.0
if (CONFIG_RCUTILS)

zephyr_library_named(rcutils)

target_compile_definitions(rcutils PUBLIC
  )
target_compile_options(rcutils PRIVATE -Wno-unused-variable -Wno-implicit-function-declaration -Wno-int-conversion)

zephyr_include_directories(
  ${ZEPHYR_CURRENT_MODULE_DIR}/include
  ${CMAKE_CURRENT_BINARY_DIR}/include
  )

zephyr_library_include_directories(
  ${ZEPHYR_CURRENT_MODULE_DIR}/include
  ${CMAKE_CURRENT_BINARY_DIR}/include
  )

# "watch" template/inputs for changes
configure_file(
  "${ZEPHYR_CURRENT_MODULE_DIR}/resource/logging_macros.h.em"
  "logging_macros.h.em.watch"
  COPYONLY)

configure_file(
  "${ZEPHYR_CURRENT_MODULE_DIR}/rcutils/logging.py"
  "logging.py.watch"
  COPYONLY)

# generate header with logging macros
set(rcutils_module_path ${ZEPHYR_CURRENT_MODULE_DIR})
set(python_code "import em"  # implicitly added ; between python statements due to CMake list
  "\
em.invoke( \
  [ \
    '-o', '${CMAKE_CURRENT_BINARY_DIR}/include/rcutils/logging_macros.h', \
    '-D', 'rcutils_module_path=\"${rcutils_module_path}\"', \
    '${ZEPHYR_CURRENT_MODULE_DIR}/resource/logging_macros.h.em' \
  ] \
)")
string(REPLACE ";" "$<SEMICOLON>" python_code "${python_code}")

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/include/rcutils/logging_macros.h
  COMMAND touch ${CMAKE_CURRENT_BINARY_DIR}/include/rcutils/logging_macros.h
  COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_CURRENT_BINARY_DIR}/include/rcutils"
  COMMAND Python3::Interpreter ARGS -c "${python_code}"
  DEPENDS
    "${CMAKE_CURRENT_BINARY_DIR}/logging_macros.h.em.watch"
    "${CMAKE_CURRENT_BINARY_DIR}/logging.py.watch"
  COMMENT "Expanding logging_macros.h.em"
  VERBATIM
)

file(GLOB_RECURSE Sources
  "${ZEPHYR_CURRENT_MODULE_DIR}/src/*.c"
)

add_custom_target(gen_rcutils ALL DEPENDS
	${CMAKE_CURRENT_BINARY_DIR}/include/rcutils/logging_macros.h)


target_sources(rcutils PRIVATE
  ${Sources}
  ${CMAKE_CURRENT_BINARY_DIR}/include/rcutils/logging_macros.h
)

endif()
