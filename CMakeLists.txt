cmake_minimum_required(VERSION 2.8)
set(CMAKE_VERBOSE_MAKEFILE on)
include(ExternalProject)
include(ProcessorCount)

if (NOT INSTALL_ROOT)
message("\
**************************
INSTALL_ROOT must be defined
INSTALL_ROOT = path to where to install the compiled software, e.g, /home/myname/root
**************************")
  message(FATAL_ERROR)
endif()
ProcessorCount(JJ)
message("INSTALL_ROOT = ${INSTALL_ROOT}")
message("Parallel build with ${JJ} cores")

#***********************************
# Downloads
#***********************************
# gtest
ExternalProject_Add(gtest-download
  PREFIX downloads/gtest-download
  URL https://github.com/google/googletest/archive/release-1.8.0.zip
  DOWNLOAD_NAME gtest-1.8.0.zip
  URL_HASH SHA1=667f873ab7a4d246062565fad32fb6d8e203ee73
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
#OpenBlas
ExternalProject_Add(openblas-download
  PREFIX downloads/openblas-download
  URL https://github.com/xianyi/OpenBLAS/archive/v0.2.19.tar.gz
  DOWNLOAD_NAME OpenBLAS-0.2.19.tar.gz
  URL_HASH SHA1=df4ceb25d2529748ad0f2715e62fefb348477edd
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
# LAPACK
ExternalProject_Add(lapack-download
  PREFIX downloads/lapack-download
  URL http://www.netlib.org/lapack/lapack-3.6.1.tgz
  DOWNLOAD_NAME lapack-3.6.1.tgz
  URL_HASH SHA1=7220a9bc9c766cb513007f313b804d88f110c228
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
# ARPACK-NG
ExternalProject_Add(arpackng-download
  PREFIX downloads/arpackng-download
  URL https://github.com/opencollab/arpack-ng/archive/3.4.0.tar.gz
  DOWNLOAD_NAME arpack-ng-3.4.0.tar.gz
  URL_HASH SHA1=c6ea9ea1712c27ec45fbf33c3f064fdba7e9c89b
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
# Armadillo
ExternalProject_Add(armadillo-download
  PREFIX downloads/armadillo-download
  URL http://sourceforge.net/projects/arma/files/armadillo-7.400.2.tar.xz
  DOWNLOAD_NAME armadillo-7.400.2.tar.xz
  URL_HASH SHA1=99c60e7c973539d4b84b24401c81f428ded3b8c9
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
# libpca
ExternalProject_Add(libpca-download
  PREFIX downloads/libpca-download
  URL http://sourceforge.net/projects/libpca/files/libpca-1.3.3.tar.gz
  DOWNLOAD_NAME libpca-1.3.3.tar.gz
  URL_HASH SHA1=ead870a2a2c6ca4b62c180d782a051ee476fa030
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
# boost
ExternalProject_Add(boost-download
  PREFIX downloads/boost-download
  URL https://sourceforge.net/projects/boost/files/boost/1.61.0/boost_1_61_0.tar.gz/download
  DOWNLOAD_NAME boost_1_61_0.tar.gz
  URL_HASH SHA1=2d2b80771f584f327827193fcf3abf836edcdfaa
  INSTALL_COMMAND ""
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
)

# mlpack
ExternalProject_Add(mlpack-download
  PREFIX downloads/mlpack-download
  URL http://www.mlpack.org/files/mlpack-2.0.3.tar.gz
  DOWNLOAD_NAME mlpack-2.0.3.tar.gz
  URL_HASH SHA1=3e9e7d4ae7d6ac70f97262d2bbedafada039d33c
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)

#***********************************
# Builds
#***********************************
# Gtest
ExternalProject_Get_Property(gtest-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(gtest-build
    PREFIX builds/gtest-build
    DEPENDS gtest-download
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=667f873ab7a4d246062565fad32fb6d8e203ee73
    BUILD_COMMAND make -j${JJ}    
    INSTALL_COMMAND ${CMAKE_COMMAND} -DINSTALL_ROOT=${INSTALL_ROOT} -DBUILD_DIR=${PROJECT_BINARY_DIR}/builds/gtest-build/src/gtest-build-build -P  ${CMAKE_CURRENT_LIST_DIR}/install_gtest.txt
  )
endif()

if (DEFINED WIN32)
  ExternalProject_Add(gtest-build
    PREFIX builds/gtest-build
    DEPENDS gtest-download
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=667f873ab7a4d246062565fad32fb6d8e203ee73
    CMAKE_GENERATOR "Visual Studio 14 2015 Win64"
    CMAKE_ARGS -Dgtest_force_shared_crt=ON -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT}
    BUILD_COMMAND msbuild ALL_BUILD.vcxproj /p:configuration=release
    INSTALL_COMMAND msbuild INSTALL.vcxproj /p:configuration=release    
  )
endif()

# OpenBLAS
ExternalProject_Get_Property(openblas-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(openblas-build
    PREFIX builds/openblas-build
    DEPENDS openblas-download
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=df4ceb25d2529748ad0f2715e62fefb348477edd
    CONFIGURE_COMMAND ""
    BUILD_IN_SOURCE 1
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make PREFIX=${INSTALL_ROOT} install
  )
endif()

if (DEFINED WIN32)
  string(REPLACE "/" "\\" INSTALL_ROOT_WIN ${INSTALL_ROOT})
  ExternalProject_Add(openblas-build
    PREFIX builds/openblas-build
    DEPENDS openblas-download
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=df4ceb25d2529748ad0f2715e62fefb348477edd
    CMAKE_GENERATOR "MSYS Makefiles"
    CMAKE_ARGS -DCMAKE_GNUtoMS=ON
    CONFIGURE_COMMAND ""
    # must be built in source
    BUILD_IN_SOURCE 1
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make PREFIX=${INSTALL_ROOT} install COMMAND runas /user:builder "cmd /C \"cd ${INSTALL_ROOT_WIN}\\bin & mklink libblas.dll libopenblas.dll\""
    # Symlink alternative
    # INSTALL_COMMAND make PREFIX=${INSTALL_ROOT}
  )
endif()

# LAPACK
ExternalProject_Get_Property(lapack-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(lapack-build
    PREFIX builds/lapack-build
    DEPENDS lapack-download 
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=7220a9bc9c766cb513007f313b804d88f110c228
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DCMAKE_PREFIX_PATH=${INSTALL_ROOT} -Wno-dev
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
  )
endif()

if (DEFINED WIN32)
  ExternalProject_Add(lapack-build
    PREFIX builds/lapack-build
    DEPENDS lapack-download openblas-build
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=7220a9bc9c766cb513007f313b804d88f110c228
    CMAKE_GENERATOR "MSYS Makefiles"
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DCMAKE_PREFIX_PATH=${INSTALL_ROOT} -Wno-dev -DBUILD_SHARED_LIBS=ON -DCMAKE_GNUtoMS=ON -DUSE_OPTIMIZED_BLAS=ON -DBLAS_LIBRARIES=${INSTALL_ROOT}/bin/libopenblas.dll
    BUILD_COMMAND mingw32-make -j$(JJ)
    INSTALL_COMMAND mingw32-make install
  )
endif()

# ARPACK-NG
ExternalProject_Get_Property(arpackng-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(arpackng-build
    PREFIX builds/arpackng-build
    DEPENDS arpackng-download 
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=c6ea9ea1712c27ec45fbf33c3f064fdba7e9c89b
    CMAKE_ARGS -DBUILD_SHARED_LIBS=ON
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
  )
endif()

if (DEFINED WIN32)
  ExternalProject_Add(arpackng-build
    PREFIX builds/arpackng-build
    #DEPENDS arpackng-download openblas-build lapack-build
    DEPENDS arpackng-download
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=c6ea9ea1712c27ec45fbf33c3f064fdba7e9c89b
    CMAKE_GENERATOR "MSYS Makefiles"
    # must be built in source
    BUILD_IN_SOURCE 1
    CMAKE_ARGS -DBUILD_SHARED_LIBS=ON -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DCMAKE_PREFIX_PATH=${INSTALL_ROOT} -DCMAKE_GNUtoMS=ON -DBLAS_LIBRARIES=${INSTALL_ROOT}/bin/libopenblas.dll -DLAPACK_LIBRARIES=${INSTALL_ROOT}/bin/liblapack.dll
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND ${CMAKE_COMMAND} -DINSTALL_ROOT=${INSTALL_ROOT} -DBUILD_DIR=${PROJECT_BINARY_DIR}/builds/arpackng-build/src/arpackng-build -P  ${CMAKE_CURRENT_LIST_DIR}/install_arpackng.txt
    
    )
endif()

# Armadillo
ExternalProject_Get_Property(armadillo-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(armadillo-build
    PREFIX builds/armadillo-build
    DEPENDS armadillo-download openblas-build lapack-build arpack-build
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=99c60e7c973539d4b84b24401c81f428ded3b8c9
    BUILD_IN_SOURCE 1
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DCMAKE_PREFIX_PATH=${INSTALL_ROOT}
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
  )
endif()

if (DEFINED WIN32)
  ExternalProject_Add(armadillo-build
    PREFIX builds/armadillo-build
    DEPENDS armadillo-download openblas-build lapack-build arpackng-build
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=99c60e7c973539d4b84b24401c81f428ded3b8c9
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DCMAKE_PREFIX_PATH=${INSTALL_ROOT} -DCMAKE_MODULE_PATH="${INSTALL_ROOT}/bin ${INSTALL_ROOT}/lib"
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
  )
endif()

# libpca
ExternalProject_Get_Property(libpca-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(libpca-build
    PREFIX builds/libpca-build
    DEPENDS libpca-download armadillo-build
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=ead870a2a2c6ca4b62c180d782a051ee476fa030
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ./configure --prefix=${INSTALL_ROOT} --with-armadillo-libdir=${INSTALL_ROOT}/lib --with-armadillo-incdir=${INSTALL_ROOT}/include
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
  )
endif()

# Boost
ExternalProject_Get_Property(boost-download download_dir download_name)
set (BUILD_CMD_BASE ./bjam threading=multi -j${JJ} variant=release --layout=tagged)
set (INSTALL_CMD_BASE ./bjam -j${JJ} variant=release --layout=tagged)
if (DEFINED WIN32)
  ExternalProject_Add(boost-build
    PREFIX builds/boost-build
    DEPENDS boost-download
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=2d2b80771f584f327827193fcf3abf836edcdfaa
    CONFIGURE_COMMAND ./bootstrap.sh --with-libraries=program_options,math,test,serialization
    # must be built in source
    BUILD_IN_SOURCE 1
    BUILD_COMMAND ${BUILD_CMD_BASE} stage    
    INSTALL_COMMAND ${INSTALL_CMD_BASE} --prefix=${INSTALL_ROOT} install
  )
endif()

ExternalProject_Get_Property(mlpack-download download_dir download_name)
if (DEFINED WIN32)
  ExternalProject_Add(mlpack-build
    PREFIX builds/mlpack-build
    DEPENDS mlpack-download armadillo-build boost-build
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=3e9e7d4ae7d6ac70f97262d2bbedafada039d33c
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DBOOST_ROOT=${INSTALL_ROOT} -DARMADILLO_INCLUDE_DIR=${INSTALL_ROOT}/include -DARMADILLO_LIBRARY=${INSTALL_ROOT}/bin/libarmadillo.dll -DCMAKE_MODULE_PATH="${INSTALL_ROOT}/bin ${INSTALL_ROOT}/lib" -Wno-dev -DCMAKE_GNUtoMS=ON -DBUILD_CLI_EXECUTABLES=OFF -DBUILD_TESTS=OFF 
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
  )
endif()


