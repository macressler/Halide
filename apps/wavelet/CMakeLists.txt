# Generator
halide_project(wavelet "apps" wavelet.cpp)

set(_HL_MODULES daubechies_x.o inverse_daubechies_x.o haar_x.o inverse_haar_x.o)
set(HL_MODULES "")
set(_HL_HEADER_FILES daubechies_x.h inverse_daubechies_x.h haar_x.h inverse_haar_x.h)
set(HL_HEADER_FILES "")
set(HALIDE_RUNTIME_OBJ "${CMAKE_CURRENT_BINARY_DIR}/halide_runtime.o")

foreach(object_file ${_HL_MODULES})
  list(APPEND HL_MODULES "${CMAKE_CURRENT_BINARY_DIR}/${object_file}")
endforeach()
foreach(header_file ${_HL_HEADER_FILES})
  list(APPEND HL_HEADER_FILES "${CMAKE_CURRENT_BINARY_DIR}/${header_file}")
endforeach()

# Final executable
add_executable(wavelet_filter filter.cpp ${HL_HEADER_FILES})
target_link_libraries(wavelet_filter PRIVATE ${HL_MODULES} "${HALIDE_RUNTIME_OBJ}" ${PNG_LIBRARIES})
target_include_directories(wavelet_filter PRIVATE "${CMAKE_CURRENT_BINARY_DIR}"
                           ${PNG_INCLUDE_DIRS})
target_compile_definitions(wavelet_filter PRIVATE ${PNG_DEFINITIONS})

if (NOT WIN32)
  target_link_libraries(wavelet_filter PRIVATE dl pthread)
endif()
set_target_properties(wavelet_filter wavelet PROPERTIES RUNTIME_OUTPUT_DIRECTORY
                      "${CMAKE_CURRENT_BINARY_DIR}")


# FIXME: Can't use halide_add_generator_dependency() becuase it does not
# support a seperate halide_runtime object. It also passes command line
# parameters to the generator which "wavelet" doesn't explicitly support.
add_custom_command(OUTPUT ${HL_MODULES} ${HL_HEADER_FILES} "${HALIDE_RUNTIME_OBJ}"
 COMMAND wavelet
 COMMENT "Generating wavelet object files"
 WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
)
