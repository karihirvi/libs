project(lab_deps)
cmake_minimum_required(VERSION 2.8)
set(CMAKE_VERBOSE_MAKEFILE on)
include(ExternalProject)

if (NOT INSTALL_ROOT)
message("\
**************************
INSTALL_ROOT must be defined
INSTALL_ROOT = path to where to install the compiled software, e.g, /home/myname/root
**************************")
  message(FATAL_ERROR)
endif()
message("INSTALL_ROOT = ${INSTALL_ROOT}")

if (DEFINED unix AND NOT DEFINED JJ)
message("\
**************************
JJ must be defined
JJ = number of parallel build jobs, e.g, if JJ=8, then make -j8 is used
**************************")
  message(FATAL_ERROR)
else()
  message("JJ = ${JJ}")
endif()

#***********************************
# Downloads
#***********************************
# gtest
ExternalProject_Add(gtest-download
  PREFIX downloads/gtest-download
  URL https://googletest.googlecode.com/files/gtest-1.7.0.zip
  DOWNLOAD_NAME gtest-1.7.0.zip
  URL_HASH SHA1=f85f6d2481e2c6c4a18539e391aa4ea8ab0394af
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
#OpenBlas
ExternalProject_Add(openblas-download
  PREFIX downloads/openblas-download
  URL http://github.com/xianyi/OpenBLAS/archive/v0.2.14.tar.gz
  DOWNLOAD_NAME OpenBLAS-0.2.14.tar.gz
  URL_HASH SHA1=803ddccb727f2a94740136fb7cc8cef4a546ec98
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
# LAPACK
ExternalProject_Add(lapack-download
  PREFIX downloads/lapack-download
  URL http://www.netlib.org/lapack/lapack-3.5.0.tgz
  DOWNLOAD_NAME lapack-3.5.0.tgz
  URL_HASH SHA1=5870081889bf5d15fd977993daab29cf3c5ea970
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
# ARPACK-NG
ExternalProject_Add(arpack-download
  PREFIX downloads/arpack-download
  URL https://github.com/opencollab/arpack-ng/archive/3.2.0.tar.gz
  DOWNLOAD_NAME arpack-ng-3.2.0.tar.gz
  URL_HASH SHA1=128d04d82399c4174e1b32572845055aaaafef47
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)
# Armadillo
ExternalProject_Add(armadillo-download
  PREFIX downloads/armadillo-download
  URL http://sourceforge.net/projects/arma/files/armadillo-5.200.2.tar.gz
  DOWNLOAD_NAME armadillo-5.200.2.tar.gz
  URL_HASH SHA1=e7d8392dfcc7227c17bcdbe18ad88f2995115556
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
    URL_HASH SHA1=f85f6d2481e2c6c4a18539e391aa4ea8ab0394af
    BUILD_COMMAND make -j${JJ}    
    INSTALL_COMMAND ${CMAKE_COMMAND} -DINSTALL_ROOT=${INSTALL_ROOT} -DBUILD_DIR=${PROJECT_BINARY_DIR}/builds/gtest-build/src/gtest-build-build -P  ${CMAKE_CURRENT_LIST_DIR}/install_gtest.txt
#    INSTALL_COMMAND mkdir -p ${INSTALL_ROOT}/lib COMMAND cp libgtest.a libgtest_main.a ${INSTALL_ROOT}/lib COMMAND mkdir -p ${INSTALL_ROOT}/include COMMAND cp -r ../gtest-build/include/gtest ${INSTALL_ROOT}/include
  )
endif()

if (DEFINED WIN32)
  ExternalProject_Add(gtest-build
    PREFIX builds/gtest-build
    DEPENDS gtest-download
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=f85f6d2481e2c6c4a18539e391aa4ea8ab0394af
    CMAKE_GENERATOR "Visual Studio 12 2013 Win64"
    CMAKE_ARGS -Dgtest_force_shared_crt=ON
    BUILD_COMMAND msbuild ALL_BUILD.vcxproj /p:configuration=debug COMMAND msbuild ALL_BUILD.vcxproj /p:configuration=release
    INSTALL_COMMAND ${CMAKE_COMMAND} -DDEBUG=debug -DRELEASE=release -DINSTALL_ROOT=${INSTALL_ROOT} -DBUILD_DIR=${PROJECT_BINARY_DIR}/builds/gtest-build/src/gtest-build-build -P  ${CMAKE_CURRENT_LIST_DIR}/install_gtest.txt
  )
endif()

# OpenBLAS
ExternalProject_Get_Property(openblas-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(openblas-build
    PREFIX builds/openblas-build
    DEPENDS openblas-download
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=803ddccb727f2a94740136fb7cc8cef4a546ec98
    CONFIGURE_COMMAND ""
    BUILD_IN_SOURCE 1
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make PREFIX=${INSTALL_ROOT} install
  )
endif()

if (DEFINED WIN32)
  ExternalProject_Add(openblas-build
    PREFIX builds/openblas-build
    DEPENDS openblas-download
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=803ddccb727f2a94740136fb7cc8cef4a546ec98
    CMAKE_GENERATOR "MSYS Makefiles"
    CMAKE_ARGS -DCMAKE_GNUtoMS=ON
    CONFIGURE_COMMAND ""
    BUILD_IN_SOURCE 1
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make PREFIX=${INSTALL_ROOT} install
  )
endif()

# LAPACK
ExternalProject_Get_Property(lapack-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(lapack-build
    PREFIX builds/lapack-build
    DEPENDS lapack-download 
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=5870081889bf5d15fd977993daab29cf3c5ea970
    CMAKE_GENERATOR "MinGW Makefiles"
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DCMAKE_PREFIX_PATH=${INSTALL_ROOT} -Wno-dev
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
  )
endif()

if (DEFINED WIN32)
#  ExternalProject_Add(lapack-build
#    PREFIX builds/lapack-build
#    DEPENDS lapack-download 
#    URL ${download_dir}/${download_name}
#    URL_HASH SHA1=5870081889bf5d15fd977993daab29cf3c5ea970
#    CMAKE_GENERATOR "MSYS Makefiles"
#    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DCMAKE_PREFIX_PATH=${INSTALL_ROOT} -Wno-dev -DBUILD_SHARED_LIBS=ON -DCMAKE_GNUtoMS=ON  
#    BUILD_COMMAND make -j$(JJ)
#    INSTALL_COMMAND make install
#  )
  ExternalProject_Add(lapack-build
    PREFIX builds/lapack-build
    DEPENDS lapack-download 
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=5870081889bf5d15fd977993daab29cf3c5ea970
    CMAKE_GENERATOR "MinGW Makefiles"
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DCMAKE_PREFIX_PATH=${INSTALL_ROOT} -Wno-dev -DBUILD_SHARED_LIBS=ON -DCMAKE_GNUtoMS=ON  
    BUILD_COMMAND mingw32-make -j$(JJ)
    INSTALL_COMMAND mingw32-make install
  )
endif()

# ARPACK-NG
ExternalProject_Get_Property(arpack-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(arpack-build
    PREFIX builds/arpack-build
    DEPENDS arpack-download 
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=128d04d82399c4174e1b32572845055aaaafef47
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ./configure --prefix=${INSTALL_ROOT}
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
  )
endif()

if (DEFINED WIN32)
  ExternalProject_Add(arpack-build
    PREFIX builds/arpack-build
    DEPENDS arpack-download 
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=128d04d82399c4174e1b32572845055aaaafef47
    CMAKE_GENERATOR "MSYS Makefiles"
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ./configure --prefix=${INSTALL_ROOT} --host=x86_64-w64-mingw32 --with-blas=${INSTALL_ROOT}/lib/libblas.dll.a --with-lapack=${INSTALL_ROOT}/lib/liblapack.dll.a
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
    )
endif()

# Armadillo
ExternalProject_Get_Property(armadillo-download download_dir download_name)
if (DEFINED UNIX)
  ExternalProject_Add(armadillo-build
    PREFIX builds/armadillo-build
    DEPENDS armadillo-download openblas-build lapack-build arpack-build
    URL ${download_dir}/${download_name}
    URL_HASH SHA1=e7d8392dfcc7227c17bcdbe18ad88f2995115556
    BUILD_IN_SOURCE 1
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT} -DCMAKE_PREFIX_PATH=${INSTALL_ROOT}
    BUILD_COMMAND make -j${JJ}
    INSTALL_COMMAND make install
  )
endif()

if (DEFINED WIN32)
  message("TODO: eigen windows")
#  message(FATAL_ERROR)
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

if (DEFINED WIN32)
  message("TODO: eigen windows")
#  message(FATAL_ERROR)
endif()



