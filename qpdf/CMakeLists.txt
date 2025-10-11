set(MAIN_CXX_PROGRAMS qpdf)
set(MAIN_C_PROGRAMS)

foreach(PROG ${MAIN_CXX_PROGRAMS})
  add_executable(${PROG} ${PROG}.cc)
  target_link_libraries(${PROG} libqpdf)
endforeach()
foreach(PROG ${MAIN_C_PROGRAMS})
  add_executable(${PROG} ${PROG}.c)
  target_link_libraries(${PROG} libqpdf)
  set_property(TARGET ${PROG} PROPERTY LINKER_LANGUAGE CXX)
endforeach()


foreach(B qpdf test_unicode_filenames fix-qdf test_shell_glob)
  if(TARGET ${B})
    if(WINDOWS_WMAIN_COMPILE)
      target_compile_options(${B} PRIVATE ${WINDOWS_WMAIN_COMPILE})
    endif()
    if(WINDOWS_WMAIN_LINK)
      target_link_options(${B} PRIVATE ${WINDOWS_WMAIN_LINK})
    endif()
  endif()
endforeach()


install(TARGETS qpdf
  TYPE RUNTIME
  COMPONENT ${COMPONENT_CLI})

if(MINGW)
  # For MSVC, including InstallRequiredSystemLibraries in the
  # top-level CMakeLists.txt is sufficient. For mingw, we have to copy
  # mingw libraries in ourselves.
  set(ONE_GNU_DLL extra-dlls/libstdc++-6.dll)
  add_custom_command(OUTPUT ${ONE_GNU_DLL}
    COMMAND
    pwsh.exe -NoProfile -ExecutionPolicy Bypass -Command
    "${qpdf_SOURCE_DIR}/copy_dlls.ps1"
    -exe $<TARGET_FILE:qpdf>
    -dest "${CMAKE_CURRENT_BINARY_DIR}/extra-dlls"
    -mingwBinDir "${MINGW_BIN_DIR}"
    VERBATIM
    COMMAND_EXPAND_LISTS
    DEPENDS qpdf)
  add_custom_target(extra_dlls ALL DEPENDS ${ONE_GNU_DLL})
  add_dependencies(extra_dlls qpdf)
  if(BUILD_SHARED_LIBS)
    set(LIBQPDF_NAME get_filename_component($<TARGET_FILE:libqpdf> NAME))
    add_custom_command(
        TARGET qpdf POST_BUILD
        COMMAND
            ${CMAKE_COMMAND} -E copy_if_different
                $<TARGET_FILE:libqpdf>
                $<TARGET_FILE_DIR:qpdf>
        VERBATIM
        COMMAND_EXPAND_LISTS)
    set(EXTRA_DLL_COMPONENT ${COMPONENT_LIB})
  else()
    set(EXTRA_DLL_COMPONENT ${COMPONENT_CLI})
  endif()
  # The trailing / prevents "extra-dlls" from being created in bin.
  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/extra-dlls/
    TYPE BIN
    COMPONENT ${EXTRA_DLL_COMPONENT})
endif()
