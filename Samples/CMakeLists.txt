file(GLOB SUB_DIRS *)
foreach(PATH ${SUB_DIRS})
  if(NOT IS_DIRECTORY ${PATH})
    continue()
  endif()

  get_filename_component(NAME ${PATH} NAME)
  if(EXISTS "${PATH}/CMakeLists.txt")
    add_subdirectory(${NAME})
    continue()
  endif()

  file(GLOB_RECURSE SRCS "${NAME}/*")
  foreach(SRC_PATH ${SRCS})
    if(SRC_PATH MATCHES ".*\\.xml$")
      get_filename_component(XML_FILE_NAME ${SRC_PATH} NAME)
      string(REGEX REPLACE "\\.(xml|XML)$" "" XML_NAME ${XML_FILE_NAME})
      set(BML_PATH    "${XML_NAME}.bml")
      set(RC_PATH     "${XML_NAME}.rc")
      set(HEADER_PATH "${XML_NAME}.h")
      add_custom_command(
        OUTPUT ${BML_PATH} ${RC_PATH} ${HEADER_PATH}
        COMMAND uicc.exe ${SRC_PATH} ${BML_PATH} /res:${RC_PATH} /header:${HEADER_PATH}
        DEPENDS "${SRC_PATH}" "${HEADER_PATH}"
        COMMENT "Compiling ${XML_FILE_NAME}"
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
      )
      list(APPEND SRCS ${BML_PATH} ${RC_PATH} ${HEADER_PATH})
    endif()
  endforeach()
  add_executable(${NAME} WIN32 ${SRCS})
  target_include_directories(${NAME} PUBLIC ../Include ${CMAKE_CURRENT_BINARY_DIR})
  target_compile_definitions(${NAME} PUBLIC _AFXDLL _UNICODE UNICODE _EMBEDDED_MANIFEST)
endforeach()
