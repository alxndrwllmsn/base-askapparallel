#
# base/askapparallel
#
add_library(askap_askapparallel
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
if (MPI_FOUND)
  if(MPI_COMPILE_FLAGS)
    set_target_properties(askap_askapparallel  PROPERTIES
    COMPILE_FLAGS "${MPI_COMPILE_FLAGS}")
  endif()

  if(MPI_LINK_FLAGS)
    set_target_properties(askap_askapparallel PROPERTIES
    LINK_FLAGS "${MPI_LINK_FLAGS}")
  endif()

endif()

target_link_libraries (
	askap_askapparallel
	lofar::Blob
	askap::askap
	${CASACORE_LIBRARIES}
	${log4cxx_LIBRARY}
	${MPI_LIBRARIES}
)

target_include_directories(askap_askapparallel PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/askap>
  $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}>
  $<INSTALL_INTERFACE:include>
  $<INSTALL_INTERFACE:include/askap>
)

install (
	TARGETS askap_askapparallel
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
	LIBRARY PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
)