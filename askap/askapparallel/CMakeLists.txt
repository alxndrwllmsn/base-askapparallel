#
# base/askapparallel
#
add_library(parallel
askap_askapparallel.cc
AskapParallel.cc
BlobIBufMW.cc
BlobOBufMW.cc
MPIComms.cc
)

install (FILES
AskapParallel.h
BlobIBufMW.h
BlobOBufMW.h
MPIComms.h
DESTINATION include/askap/askapparallel
)

set_target_properties(parallel PROPERTIES
    OUTPUT_NAME askap_askapparallel
)

target_link_libraries ( parallel 
	lofar::Blob
	askap::askap
	${CASACORE_LIBRARIES}
	${log4cxx_LIBRARY}
	${MPI_LIBRARIES}
)

if (OPENMP_FOUND)
    target_link_libraries(parallel OpenMP::OpenMP_CXX)
endif (OPENMP_FOUND)

add_library(askap::parallel ALIAS parallel)
target_include_directories(parallel PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/askap>
  $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}>
  $<INSTALL_INTERFACE:include>
  $<INSTALL_INTERFACE:include/askap>
  ${Boost_INCLUDE_DIRS}
  ${log4cxx_INCLUDE_DIRS}
  ${CASACORE_INCLUDE_DIRS}
)
target_compile_definitions(parallel PUBLIC
  casa=casacore
  HAVE_AIPSPP
  HAVE_BOOST
  HAVE_LOG4CXX
)

if (MPI_FOUND)
  target_include_directories(parallel PUBLIC
    ${MPI_INCLUDE_PATH}
  )
  target_compile_definitions(parallel PUBLIC
    HAVE_MPI
  )
  if(MPI_COMPILE_FLAGS)
    set_target_properties(parallel  PROPERTIES
    COMPILE_FLAGS "${MPI_COMPILE_FLAGS}")
  endif()

  if(MPI_LINK_FLAGS)
    set_target_properties(parallel PROPERTIES
    LINK_FLAGS "${MPI_LINK_FLAGS}")
  endif()

endif()

install (
	TARGETS parallel
	EXPORT askap-parallel-targets
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
	LIBRARY PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
)
