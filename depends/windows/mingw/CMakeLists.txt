cmake_minimum_required(VERSION 3.5)
project(mingw)

# Currently unused (our mirrors not have newest)
if(0)
  foreach(repo msys mingw32 mingw64)
    if(${repo} STREQUAL msys)
      file(WRITE ${CMAKE_CURRENT_SOURCE_DIR}/etc/pacman.d/mirrorlist.${repo} "Server = http://mirrors.kodi.tv/build-deps/win32/msys2/repos/${repo}2/$arch\n")
    else()
      file(WRITE ${CMAKE_CURRENT_SOURCE_DIR}/etc/pacman.d/mirrorlist.${repo} "Server = http://mirrors.kodi.tv/build-deps/win32/msys2/repos/${repo}\n")
    endif()
  endforeach()
endif()

if($ENV{PROCESSOR_ARCHITECTURE} MATCHES "AMD64" OR
   $ENV{PROCESSOR_ARCHITECTURE} MATCHES "IA64") 
   set(HOST mingw-w64-x86_64)
   set(LUAHOST mingw-w64-x86_64)
   set(MINGW_PATH "mingw64")
elseif($ENV{PROCESSOR_ARCHITECTURE} MATCHES "ARM64")
   set(HOST mingw-w64-cross-mingwarm64)
   set(LUAHOST mingw-w64-x86_64)
   set(MINGW_PATH "mingw64")
else()
   set(HOST mingw-w64-i686)
   set(LUAHOST mingw-w64-i686)
   set(MINGW_PATH "mingw32")
endif()

execute_process(COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/usr/bin/bash.exe -lc
    " " ERROR_QUIET)
execute_process(COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/usr/bin/bash.exe -lc
    "rm /var/lib/pacman/sync/*")    
execute_process(COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/usr/bin/bash.exe -lc
    "pacman --noconfirm -Syuu")
execute_process(COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/usr/bin/bash.exe -lc
    "pacman --noconfirm -Syuu \
                           mingw-w64-cross-binutils \
                           ${HOST}-gcc ${LUAHOST}-lua \
                           ${HOST}-headers \
                           make \
                           autoconf \
                           automake \
                           libtool \
                           pkg-config \
                           nasm \
                           yasm \
                           patch")
if($ENV{PROCESSOR_ARCHITECTURE} MATCHES "ARM64")
   # mingw-w64-cross-mingwarm64 does not create gcc anywhere in the path
   execute_process(COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/usr/bin/bash.exe -lc
    "which gcc || find /opt/bin -type f -name 'aarch64-w64-mingw32-*' -exec /bin/bash -c 'ln -s {} $(echo /opt/bin/{} | sed -e ''s#/opt/bin/aarch64-w64-mingw32-##g'')' \\;")
   execute_process(COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/usr/bin/bash.exe -lc
   "which cc || ln -s /opt/bin/gcc /opt/bin/cc")
endif()

file(GLOB_RECURSE shared_libs ${CMAKE_CURRENT_SOURCE_DIR}/${MINGW_PATH}/*.dll.a)
if(shared_libs)
  file(REMOVE ${shared_libs})
endif()

configure_file(MinGWConfig.cmake.in MinGWConfig.cmake @ONLY)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/MinGWConfig.cmake DESTINATION ${CMAKE_INSTALL_PREFIX})
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/caller.sh DESTINATION ${CMAKE_INSTALL_PREFIX})
