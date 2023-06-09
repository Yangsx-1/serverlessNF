CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT(MICA)


SET(CMAKE_CXX_COMPILER g++)
SET(CMAKE_C_LINK_EXECUTABLE g++)



OPTION(DEBUG "Enable debugging" OFF)
OPTION(LTO "Use link time optimization" OFF)

# OPTION(AVX2 "Enable AVX2" ON)

OPTION(DPDK "Enable Intel DPDK" ON)



INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR}/src)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/build)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/build)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/build)



ADD_DEFINITIONS(-std=c++14)

ADD_DEFINITIONS(-Wall -Wextra)
ADD_DEFINITIONS(-Wzero-as-null-pointer-constant)
ADD_DEFINITIONS(-Wsign-conversion)
ADD_DEFINITIONS(-Wconversion)
ADD_DEFINITIONS(-Winline)

ADD_DEFINITIONS(-Wno-unused-function)

ADD_DEFINITIONS(-march=native)
ADD_DEFINITIONS(-pthread)

ADD_DEFINITIONS(-g)



IF(DEBUG)

  MESSAGE(WARNING "Debugging is ENABLED (to disable, run `cmake .. -DDEBUG=OFF`). Performance will be low.")
  ADD_DEFINITIONS(-Og)
  # ADD_DEFINITIONS(-O0)

ELSE(DEBUG)

  MESSAGE(STATUS "Debugging is disabled (to enable, run `cmake .. -DDEBUG=ON`)")
  ADD_DEFINITIONS(-DNDEBUG)
  ADD_DEFINITIONS(-O3)
  # ADD_DEFINITIONS(--param inline-unit-growth=200 --param large-function-growth=500)

ENDIF(DEBUG)


IF(LTO)

  MESSAGE(STATUS "Link time optimization is enabled (to disable, run `cmake .. -DLTO=OFF`)")
  ADD_DEFINITIONS(-flto)
  SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -flto")

ELSE(LTO)

  MESSAGE(WARNING "Link time optimization is DISABLED (to enable, run `cmake .. -DLTO=ON`). Performance will be low.")

ENDIF(LTO)


# IF(AVX2)

#   MESSAGE(STATUS "AVX2 is enabled (to disable, run `cmake .. -DAVX2=OFF`)")
#   ADD_DEFINITIONS(-mavx2)
#   SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -mavx2")

# ELSE(LTO)

#   MESSAGE(WARNING "AVX2 is DISABLED (to enable, run `cmake .. -DAVX2=ON`). Performance will be low.")

# ENDIF(AVX2)


IF(DPDK)

  MESSAGE(STATUS "Intel DPDK support is enabled (to disable, run `cmake .. -DDPDK=OFF`)")
  set(RTE_SDK ${CMAKE_SOURCE_DIR}/build/dpdk)
  add_definitions(-DUSE_DPDK)
  add_definitions(-isystem ${RTE_SDK}/build/include -include rte_config.h)
  link_directories(${LINK_DIRECTORIES} ${RTE_SDK}/build/lib)
  set(LIBRARIES ${LIBRARIES} -Wl,--whole-archive)
  set(LIBRARIES ${LIBRARIES} -Wl,--start-group
    rte_net      rte_ethdev       rte_kni        rte_mbuf      rte_distributor    rte_pmd_bond         rte_pmd_ring     rte_power
    rte_acl      rte_eal          rte_kvargs     rte_mempool   rte_mempool_ring   rte_pmd_e1000        rte_pmd_virtio   rte_ring
    rte_cfgfile  rte_hash         rte_lpm        rte_meter     rte_pmd_i40e       rte_pmd_vmxnet3_uio  rte_sched        rte_timer
    rte_cmdline  rte_ip_frag      rte_pipeline   rte_pmd_ixgbe rte_mempool_stack  rte_port             rte_table        rte_pmd_bnxt
    rte_pmd_ena  rte_pmd_nfp
    rte_pci      rte_bus_vdev     rte_pmd_vdev_netvsc          rte_stack          rte_cryptodev        rte_eventdev     rte_bus_pci
    numa
    rte_pmd_mlx5 ibverbs mlx5
    -Wl,--end-group)
  set(LIBRARIES ${LIBRARIES} rt dl pthread m)
  set(LIBRARIES ${LIBRARIES} -Wl,--no-whole-archive)
  set(LIBRARIES ${LIBRARIES} -gdwarf-4 -g3)
    #ADD THE 4 LINES FROM THE ORIGINAL VERSION
  set(LIBRARIES ${LIBRARIES} rte_kvargs rte_mbuf rte_ethdev rte_mempool rte_ring rte_eal)
  set(LIBRARIES ${LIBRARIES} rte_pmd_i40e rte_pmd_ixgbe rte_pmd_ring)
  set(LIBRARIES ${LIBRARIES} -Wl,--no-whole-archive)
  set(LIBRARIES ${LIBRARIES} rt dl)
  INCLUDE_DIRECTORIES(
     RTE_SDK/x86_64-native-linuxapp-gcc/include
     RTE_SDK/drivers/net/bnxt
     RTE_SDK/dpdk/drivers/net/ixgbe/base
     RTE_SDK/drivers/net/ixgbe
     RTE_SDK/drivers/net/i40e/base
     RTE_SDK/drivers/net/i40e
     RTE_SDK/drivers/net/e1000
     RTE_SDK/drivers/net/e1000/base
     RTE_SDK/drivers/net/mlx5
     #${CMAKE_CURRENT_SOURCE_DIR}/tools
     #${CMAKE_CURRENT_SOURCE_DIR}
)

ELSE(DPDK)

  MESSAGE(STATUS "Intel DPDK support is disabled (to enable, run `cmake .. -DDPDK=ON`)")

ENDIF(DPDK)


IF(SQLITE)

  MESSAGE(STATUS "Sqlite backend support is enabled (to disable, run `cmake .. -DSQLITE=OFF`)")
  add_definitions(-DUSE_SQLITE)
  set(LIBRARIES ${LIBRARIES} sqlite3)

ELSE(SQLITE)

  MESSAGE(STATUS "Sqlite backend support is disabled (to enable, run `cmake .. -DSQLITE=ON`)")

ENDIF(SQLITE)



SET(SOURCES ${SOURCES} src/mica/alloc/hugetlbfs_shm.cc)
SET(SOURCES ${SOURCES} src/mica/directory/directory_client.cc)
SET(SOURCES ${SOURCES} src/mica/directory/etcdcpp/etcd_mod.cc)
SET(SOURCES ${SOURCES} src/mica/util/config.cc)
SET(SOURCES ${SOURCES} src/mica/util/cityhash/city_mod.cc)
SET(SOURCES ${SOURCES} src/mica/util/siphash/siphash24.c)
SET_SOURCE_FILES_PROPERTIES(src/mica/util/siphash/siphash24.c PROPERTIES LANGUAGE CXX)
SET(SOURCES ${SOURCES} src/mica/util/lcore.cc)
SET(SOURCES ${SOURCES} src/mica/util/stopwatch.cc)
SET(SOURCES ${SOURCES} src/mica/util/zipf.cc)

SET(LIBRARIES ${LIBRARIES} rt numa pthread)
SET(LIBRARIES ${LIBRARIES} curl)
# SET(LIBRARIES ${LIBRARIES} boost_coroutine boost_system)



IF(LTO)

  #ADD_EXECUTABLE(test_table src/mica/test/test_table.cc ${SOURCES})
  #TARGET_LINK_LIBRARIES(test_table ${LIBRARIES})

  ADD_EXECUTABLE(test_processor src/mica/test/test_processor.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_processor ${LIBRARIES})

  #ADD_EXECUTABLE(test_load src/mica/test/test_load.cc ${SOURCES})
  #TARGET_LINK_LIBRARIES(test_load ${LIBRARIES})

  ADD_EXECUTABLE(test_zipf src/mica/test/test_zipf.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_zipf ${LIBRARIES})

  ADD_EXECUTABLE(test_prefetch src/mica/test/test_prefetch.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_prefetch ${LIBRARIES})

  ADD_EXECUTABLE(test_rand src/mica/test/test_rand.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_rand ${LIBRARIES})

  ADD_EXECUTABLE(test_hash src/mica/test/test_hash.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_hash ${LIBRARIES})

  ADD_EXECUTABLE(test_atomics src/mica/test/test_atomics.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_atomics ${LIBRARIES})

  ADD_EXECUTABLE(test_tsc_sync src/mica/test/test_tsc_sync.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_tsc_sync ${LIBRARIES})

  ADD_EXECUTABLE(microbench src/mica/test/microbench.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(microbench ${LIBRARIES})

  ADD_EXECUTABLE(cdf src/mica/test/cdf.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(cdf ${LIBRARIES})

  IF(DPDK)

    ADD_EXECUTABLE(server src/mica/test/server.cc ${SOURCES})
    TARGET_LINK_LIBRARIES(server ${LIBRARIES})

    ADD_EXECUTABLE(netbench src/mica/test/netbench.cc ${SOURCES})
    TARGET_LINK_LIBRARIES(netbench ${LIBRARIES})

    IF(SQLITE)

      ADD_EXECUTABLE(server_sqlite src/mica/test/server_sqlite.cc ${SOURCES})
      TARGET_LINK_LIBRARIES(server_sqlite ${LIBRARIES})

    ENDIF(SQLITE)

  ENDIF(DPDK)

ELSE(LTO)

  ADD_LIBRARY(common ${SOURCES})

  ADD_EXECUTABLE(test_logResizing src/mica/test/test_logResizing.cc)
  TARGET_LINK_LIBRARIES(test_logResizing common ${LIBRARIES})

  ADD_EXECUTABLE(test_clean_table src/mica/test/test_clean_table.cc)
  TARGET_LINK_LIBRARIES(test_clean_table common ${LIBRARIES})

  ADD_EXECUTABLE(test_memory_adjustment src/mica/test/test_memory_adjustment.cc)
  TARGET_LINK_LIBRARIES(test_memory_adjustment common ${LIBRARIES})

  #ADD_EXECUTABLE(test_table src/mica/test/test_table.cc)
  #TARGET_LINK_LIBRARIES(test_table common ${LIBRARIES})

  ADD_EXECUTABLE(test_processor src/mica/test/test_processor.cc)
  TARGET_LINK_LIBRARIES(test_processor common ${LIBRARIES})

  #ADD_EXECUTABLE(test_load src/mica/test/test_load.cc)
  #TARGET_LINK_LIBRARIES(test_load common ${LIBRARIES})

  ADD_EXECUTABLE(test_zipf src/mica/test/test_zipf.cc)
  TARGET_LINK_LIBRARIES(test_zipf common ${LIBRARIES})

  ADD_EXECUTABLE(test_prefetch src/mica/test/test_prefetch.cc)
  TARGET_LINK_LIBRARIES(test_prefetch common ${LIBRARIES})

  ADD_EXECUTABLE(test_rand src/mica/test/test_rand.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_rand ${LIBRARIES})

  ADD_EXECUTABLE(test_hash src/mica/test/test_hash.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_hash ${LIBRARIES})

  ADD_EXECUTABLE(test_atomics src/mica/test/test_atomics.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_atomics ${LIBRARIES})

  ADD_EXECUTABLE(test_tsc_sync src/mica/test/test_tsc_sync.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_tsc_sync ${LIBRARIES})

  ADD_EXECUTABLE(microbench src/mica/test/microbench.cc)
  TARGET_LINK_LIBRARIES(microbench common ${LIBRARIES})

  ADD_EXECUTABLE(cdf src/mica/test/cdf.cc)
  TARGET_LINK_LIBRARIES(cdf common ${LIBRARIES})

  IF(DPDK)

    ADD_EXECUTABLE(server src/mica/test/server.cc)
    TARGET_LINK_LIBRARIES(server common ${LIBRARIES})

    ADD_EXECUTABLE(netbench src/mica/test/netbench.cc)
    TARGET_LINK_LIBRARIES(netbench common ${LIBRARIES})

    IF(SQLITE)

      ADD_EXECUTABLE(server_sqlite src/mica/test/server_sqlite.cc)
      TARGET_LINK_LIBRARIES(server_sqlite common ${LIBRARIES})

    ENDIF(SQLITE)

  ENDIF(DPDK)

ENDIF(LTO)
