Primary purpose of this cmake script is to automatically download and build 
mlpack (http://www.mlpack.org/) and it's dependencies.


Dependency hierarchy (Graphviz code, paste it to http://www.webgraphviz.com/)
digraph G {
  rankdir = BT
  Armadillo -> OpenBLAS
  Armadillo -> LAPACK
  LAPACK -> OpenBLAS
  Armadillo -> "ARPACK-NG"
  mlpack -> Boost
  mlpack -> Armadillo

  {rank = same; OpenBLAS; "ARPACK-NG"; Boost}
}
 
Libraries (TODO still) not related to mlpack:
- libpca

****** gtest for Windows (tested with win 7) *******
1) Open command line with "MSBuild Command Prompt for VS2015"
2) cd to build directory, e.g., "cd c:\Users\Kari\build". Remove CMakeCache.txt
   if it exists from a previous build with MSYS makefiles.
3) cmake -DINSTALL_ROOT=C:/Users/Kari/build/root -G "Visual Studio 14 2015 Win64" C:/Users/Kari/Documents/work/github/libs
4) Run "msbuild gtest-build.vcxproj"

****** mlpack for Windows (tested with win 7) *******
Prerequisites
1) Download MinGW from
https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/rubenvb/gcc-4.8-release/x86_64-w64-mingw32-gcc-4.8.0-win64_rubenvb.7z/download
2) Extract MinGW to C:\ which creates C:\mingw64 directory
3) Download MSYS (msys+7za+wget+svn+git+mercurial+cvs-rev13 from
https://sourceforge.net/projects/mingwbuilds/files/external-binary-packages/msys%2B7za%2Bwget%2Bsvn%2Bgit%2Bmercurial%2Bcvs-rev13.7z/download
4) Extract MSYS to C:\mingw64 which creates C:\mingw64\msys directory
5) Edit/create fstab in C:\mingw64\msys\etc directory and set up mingw mount as 
   described in "After Installing You Should" chapter here 
   http://www.mingw.org/wiki/Getting_Started 
   Two lines are sufficient, e.g., "C:\mingw64   /mingw" and a blank.

Before you build:
1) mlpack searches a BLAS library and links againts it. Unfortunately
   OpenBLAS (libopenblas.dll) is not in the list of BLAS implementations that
   mlpack searches for. To work around this, we'll create a symlink
   libblas.dll -> libopenblas.dll in the install bin directory. On Windows,
   however, symlinks are not that easy as discussed in the next point
2) By modifying local security policies you can allow a normal user to create 
   symlinks, but only if the user is not also an administrator (which is 
   usually the case if you're managing your own laptop, for example).
   Relevant references
    https://stackoverflow.com/questions/16465953/mklink-permission-on-windows-8
    https://social.msdn.microsoft.com/Forums/en-US/e967ab01-3136-4fda-9677-e5ecaaa2f694/configuring-symlink-support-in-win7?forum=os_fileservices
3) Considering 1 & 2, this script makes an assumption that a normal user 
   "builder" exists, it's not in the "Administrators" group and has full access
   to "%HOME%\build" directory, e.g., C:\Users\Kari\build. 
   Thus, do the following
   a) Create a "builder" normal user
   b) Create "%HOME%\build" directory and give yourself and "builder" full
      access to it. Also, consider disabling realtime virus scanning for 
      the build directory.
   c) Modify local security policies and add "builder" the right to create 
      symlinks as described in 2.
   With these assumptions INSTALL_COMMAND in the openblas-build should work and
   create the symlink.
   NOTE: the build will pause and prompt for "builder" password
4) Instead of 3, other option is to do things more manually.
   a) Remove the symlink creation from the openblas-build INSTALL_COMMAND
   b) Build OpenBLAS first
   c) Create the symlink manually by running cmd.exe with administrator 
      privileges
   d) Continue the build, see the dependency graph about which order to build.
  

Build (tested with cmake 3.3.1)
1) Open MSYS prompt by running "C:\mingw64\msys\msys.bat"
2) Cd to the build folder, e.g., "cd /c/Users/Kari/build/"
3) Run cmake and set INSTALL_ROOT to a directory where all libs are to be
   installed, e.g., 
   cmake -DINSTALL_ROOT=C:/Users/Kari/build/root -G "MSYS Makefiles" C:/Users/Kari/Documents/work/github/libs
   - Note to use / instead of \ in paths passed in defines (-D)
4) Build and install, e.g., "make mlpack-build". Running "make help" shows
   all the targets.

****** mlpack for Linux (TODO, not ready yet) ******
1) Create a build folder, e.g., ~/build, and cd there. 
2) Run cmake, e.g., cmake -DINSTALL_ROOT=/home/kari/work/mooselab/cpp/deps-root ../deps
3) run make


