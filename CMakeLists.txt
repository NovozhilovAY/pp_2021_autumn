cmake_minimum_required( VERSION 3.12 )

message( STATUS "Parallel Programming Course" )

include( cmake/Git.cmake )

if(WIN32)
    option(gtest_force_shared_crt "" TRUE)
endif(WIN32)

include( "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Configure.cmake" )

############################### GTest ###############################
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/gtest")
include_directories("${CMAKE_SOURCE_DIR}/3rdparty/gtest/googletest/include")
if( WIN32 )
    include_directories("${CMAKE_SOURCE_DIR}/3rdparty/gtest-mpi/win")
else( WIN32 )
    include_directories("${CMAKE_SOURCE_DIR}/3rdparty/gtest-mpi/linux")
endif( WIN32 )

################################ MPI ################################
option(USE_MPI OFF)
if( USE_MPI )
    find_package( MPI )
    if( MPI_FOUND )
        include_directories( ${MPI_INCLUDE_PATH} )
    else( MPI_FOUND )
        set( USE_MPI OFF )
    endif( MPI_FOUND )
endif( USE_MPI )

############################### OpenMP ##############################
option(USE_OMP OFF)
if( USE_OMP OR USE_SEQ )
    find_package( OpenMP )
    if( OpenMP_FOUND )
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
    else( OpenMP_FOUND )
        set( USE_OMP OFF )
    endif( OpenMP_FOUND )
endif( USE_OMP OR USE_SEQ )

############################ std::thread ############################
option(USE_STD OFF)
if( USE_STD )
    find_package( Threads )
endif( USE_STD )

################################ TBB ################################
option(USE_TBB OFF)
if( USE_TBB )
    if( WIN32 )
        include( cmake/TBBGet.cmake )
        tbb_get( TBB_ROOT tbb_root RELEASE_TAG "v2020.3" CONFIG_DIR TBB_DIR )

        if(CMAKE_SIZEOF_VOID_P EQUAL 8)
            set(SYSTEM_BIT "intel64")
        elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
            set(SYSTEM_BIT "ia32")
        endif()

        file( GLOB TBB_PRE_LIB  "${tbb_root}/bin/${SYSTEM_BIT}/vc14/*.dll" )
        file( COPY ${TBB_PRE_LIB} DESTINATION "${CMAKE_BINARY_DIR}/bin")

        find_package( TBB )
    else( WIN32 )
        include( cmake/FindTBB.cmake )
    endif( WIN32 )

    if( TBB_FOUND )
        message(STATUS "Found TBB: TRUE (found version \"${TBB_VERSION_MAJOR}.${TBB_VERSION_MINOR}\") ")
        include_directories("${tbb_root}/include")
    else( TBB_FOUND )
        set( USE_TBB OFF )
    endif( TBB_FOUND )
endif( USE_TBB )

############################## CPPLINT ##############################
find_package(Python3 REQUIRED)
enable_testing()

option(USE_STYLE_CHECKER OFF)
if( USE_STYLE_CHECKER )
    message(STATUS "Enable style check: see google style")
    include(cmake/Cpplint.cmake)
    cpplint_add_subdirectory(modules)
endif( USE_STYLE_CHECKER )

############################## Modules ##############################
include_directories(3rdparty/unapproved)
add_subdirectory(modules)
