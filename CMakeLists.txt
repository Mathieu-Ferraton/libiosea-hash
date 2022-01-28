cmake_minimum_required(VERSION 2.6.4)
cmake_policy(SET CMP0017 NEW)

PROJECT(libiosea-hash C)
set(LIBIOSEA-HASH_MAJOR_VERSION 1)
set(LIBIOSEA-HASH_MINOR_VERSION 3)
set(LIBIOSEA-HASH_PATCH_LEVEL 1)
set(LIBIOSEA-HASH_EXTRA_VERSION "")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I${CMAKE_SOURCE_DIR}/include -fPIC -g")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Werror")

set(LIBIOSEA-HASH_BASE_VERSION ${LIBIOSEA-HASH_MAJOR_VERSION}.${LIBIOSEA-HASH_MINOR_VERSION}.${LIBIOSEA-HASH_PATCH_LEVEL})

message( STATUS "cmake version ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}" )
message( STATUS "libiosea-hash version ${LIBIOSEA-HASH_BASE_VERSION}")

# Final tuning

include(CheckIncludeFiles)
include(CheckLibraryExists)

add_subdirectory(hash)

# CPack / rpmbuild specific stuff
set(CPACK_PACKAGE_FILE_NAME "libiosea-hash-Source" )
set(CPACK_PACKAGE_VENDOR "IOSEA-HASH")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "IOSEA-HASH - an object store abstraction for KVSNS")
SET(CPACK_PACKAGE_VERSION_MAJOR ${LIBIOSEA-HASH_MAJOR_VERSION})
SET(CPACK_PACKAGE_VERSION_MINOR ${LIBIOSEA-HASH_MINOR_VERSION})
SET(CPACK_PACKAGE_VERSION_PATCH ${LIBIOSEA-HASH_PATCH_LEVEL})

# Tell CPack the kind of packages to be generated
set(CPACK_GENERATOR "TGZ")
set(CPACK_SOURCE_GENERATOR "TGZ")

set(CPACK_SOURCE_IGNORE_FILES
  "/.git/;/.gitignore/;/build/;/.bzr/;~$;${CPACK_SOURCE_IGNORE_FILES}")

include(CPack)

set(PKG_NAME "${CPACK_PACKAGE_NAME}.tar.gz")
if(NOT WITHIN_IOSEA_NAMESPACE_REPO)
  add_custom_target(dist COMMAND ${CMAKE_MAKE_PROGRAM} package_source)
endif(NOT WITHIN_IOSEA_NAMESPACE_REPO)

# Now create a useable specfile
configure_file(
  "${PROJECT_SOURCE_DIR}/libiosea-hash.spec-in.cmake"
  "${PROJECT_SOURCE_DIR}/libiosea-hash.spec"
)

configure_file(
  "${PROJECT_SOURCE_DIR}/libiosea-hash.pc-in.cmake"
  "${PROJECT_SOURCE_DIR}/libiosea-hash.pc"
)

set(RPMDEST "--define '_srcrpmdir ${CMAKE_CURRENT_BINARY_DIR}'")

if(NOT WITHIN_IOSEA_NAMESPACE_REPO)
  add_custom_target( rpms DEPENDS dist)
  add_custom_command(TARGET rpms
                    COMMAND sh -c "rpmbuild ${RPMDEST} -ta ${CPACK_SOURCE_PACKAGE_FILE_NAME}.tar.gz"
                    VERBATIM
                    DEPENDS dist)

  add_custom_target( rpm DEPENDS dist)
  add_custom_command(TARGET rpm
                    COMMAND sh -c "rpmbuild ${RPMDEST} -tb ${CPACK_SOURCE_PACKAGE_FILE_NAME}.tar.gz"
                    VERBATIM
                    DEPENDS dist)

  add_custom_target( srpm DEPENDS dist)
  add_custom_command(TARGET srpm
                    COMMAND sh -c "rpmbuild ${RPMDEST} -ts ${CPACK_SOURCE_PACKAGE_FILE_NAME}.tar.gz"
                    VERBATIM
                    DEPENDS dist)
endif(NOT WITHIN_IOSEA_NAMESPACE_REPO)

# INSTALL
install(FILES ${PROJECT_SOURCE_DIR}/libiosea-hash.pc DESTINATION lib)
install(FILES ${PROJECT_SOURCE_DIR}/include/iosea/hashlib.h DESTINATION include/iosea)
