# The tests make use libnvToolsExt.a, which is bundled with CUDA TOOLKIT
# but the library is not exported in a FindCUDA.cmake CUDA_<name>_LIBRARY variable
# Depending on the platform, this could be in ${CUDA_TOOLKIT_ROOT_DIR}/{lib,lib64,lib32}
# so we assume it is side-by-side with curand, and use that directory as a hint
# for the find_library call.
get_filename_component(CUDA_LIB_PATH ${CUDA_curand_LIBRARY} DIRECTORY)
find_library(CUDA_nvToolsExt_LIBRARY nvToolsExt HINTS ${CUDA_TOOLKIT_ROOT_DIR} ${CUDA_LIB_PATH})
if(NOT CUDA_nvToolsExt_LIBRARY)
  message(FATAL_ERROR "Can't find nvToolsExt")
endif()

list(APPEND CUDA_LIBRARIES ${CUDA_CUDA_LIBRARY} ${CUDA_curand_LIBRARY} ${CUDA_nvToolsExt_LIBRARY})

# mpi:
# find_package(MPI)
# cuda_add_executable(mpi_test mpi/mpi_test.cu include/test_utilities.h)
# target_include_directories(mpi_test PUBLIC include)
# target_link_libraries(mpi_test ${PROJECT_NAME} ${CUDA_LIBRARIES} ${MPI_CXX_LIBRARIES})
# install(TARGETS mpi_test DESTINATION bin)

# single:
set(single_srcs
  all_gather_scan
  all_gather_test
  all_reduce_scan
  all_reduce_test
  broadcast_scan
  broadcast_test
  reduce_scan
  reduce_scatter_scan
  reduce_scatter_test
  reduce_test
)

foreach(name ${single_srcs})
  cuda_add_executable(${name} single/${name}.cu include/test_utilities.h)
  target_include_directories(${name} PUBLIC include)
  target_link_libraries(${name} ${PROJECT_NAME} ${CUDA_LIBRARIES})
endforeach()
