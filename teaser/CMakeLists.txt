project(teaser_source)
include(GNUInstallDirs)

# teaser_registration library
add_library(teaser_registration SHARED
        src/certification.cc
        src/registration.cc
        src/graph.cc
        )
target_link_libraries(teaser_registration
        PUBLIC Eigen3::Eigen
        PRIVATE pmc ${TEASERPP_BLAS_LAPACK_LIBS}
        )
target_include_directories(teaser_registration PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<BUILD_INTERFACE:${SPECTRA_INCLUDE_DIRS}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

find_package(OpenMP)
if(OpenMP_CXX_FOUND)
    target_link_libraries(teaser_registration PRIVATE OpenMP::OpenMP_CXX)
endif()

install(TARGETS teaser_registration
    EXPORT teaserpp-export
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
list(APPEND TEASERPP_EXPORTED_TARGETS teaser_registration pmc)
add_library(teaserpp::teaser_registration ALIAS teaser_registration)



# set exported targets in parent scope
set(TEASERPP_EXPORTED_TARGETS "${TEASERPP_EXPORTED_TARGETS}" PARENT_SCOPE)

# installation
install(EXPORT teaserpp-export 
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/teaserpp
    NAMESPACE teaserpp::
    FILE teaserppTargets.cmake
)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(DIRECTORY ${SPECTRA_INCLUDE_DIRS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
