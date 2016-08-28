Downloads and builds dependencies

Windows prerequisites
- Visual Studio (compiled with VS 2013 Pro)
- MinGW (compiled with x86_64-5.1.0-posix-seh-rt_v4-rev0 from http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/5.1.0/threads-posix/seh/)
- MSYS (compiled with msys+7za+wget+svn+git+mercurial+cvs-rev13 from http://sourceforge.net/projects/mingwbuilds/files/external-binary-packages/), extract it to MinGW directory
- Allow normal users to create symbolic links. Launch secpol.msc via Start -> Run and navigate Local Policies -> User Rights Assignment and select Create symbolic links. Then add you or the whole Users group to the list.

Windows (tested with cmake 3.0.1)
1) Open Visual Studio native tools command prompt, e.g, "VS2013 x64 Native Tools Command Prompt"
2) Create a build folder and cd there. For example, create a deps-build folder on the same level where deps is.
3) Run cmake, e.g., cmake -DINSTALL_ROOT=C:/Users/Kari/Documents/work/mooselab/cpp/deps-root -DJJ=8 -G "MSYS Makefiles" ../deps
   - Note to use / instead of \ in paths
4) Make sure MinGW + MSYS tools are in your path, e.g., make. 
   For example, prepend your %PATH% by a path to MinGW bin directory, e.g., set PATH=C:\Program Files\mingw-w64\x86_64-5.1.0-posix-seh-rt_v4-rev0\mingw64\bin;%PATH%
5) Run msbuild, e.g., msbuild ALL_BUILD.vcxproj
   If you need to force rebuild: msbuild ALL_BUILD.vcxproj /t:rebuild   
   
Linux:
1) Create a build folder and cd there. For example, create a deps-build folder on the same level where deps is.
2) Run cmake, e.g., cmake -JJ=8 -DINSTALL_ROOT=/home/kari/work/mooselab/cpp/deps-root ../deps
3) run make