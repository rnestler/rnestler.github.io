Title: YouTube plugin for Kodi on the Raspberry Pi 3
Tags: Raspberry Pi, Linux, ArchLinux, ARM, Kodi
Status: draft
Language: en
Summary: Getting the YouTube plugin for Kodi to work on the Raspberry Pi 3

Last year I installed Kodi on my RaspberryPi 3 to use as a media system. While
it runs quite nicely to stream stuff from my NAS with remote controlling from
[Kore](https://f-droid.org/packages/org.xbmc.kore/), every now and then I
wanted to play something from YouTube, which didn't work.

Installing the YouTube plugin always failed with Kodi being unable to install a
dependency called `inputstream-adaptive`. Looking at
<https://kodi.tv/addons/omega/inputstream.adaptive/>` it seems, that the plugin
is only available for Windows, OSX and Android.

Looking at the ArchLinuxARM forums I found
<https://archlinuxarm.org/forum/viewtopic.php?f=60&t=13189&p=59774&hilit=kodi+inputstream#p59774>
which shows that people built the plugin themselves to get other plugins to
work.

# Installing from AUR

Luckily there is a PKGBUILD available on the AUR:
<https://aur.archlinux.org/packages/kodi-addon-inputstream-adaptive-git>. Since
I didn't want to bother with cross-compiling for the Pi I thought I'll just
compile it on the Pi itself: 

```bash
sudo pacman -S base-devel git
git clone https://aur.archlinux.org/kodi-addon-inputstream-adaptive-git.git
cd kodi-addon-inputstream-adaptive-git/
makepkg -s
```

# Trouble

The first error I ran into was that the build wants to use an unsupported flag
for the compiler:

```text
==> Starting build()...
-- The C compiler identification is GNU 12.1.0
-- The CXX compiler identification is GNU 12.1.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - failed
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - broken
CMake Error at /usr/share/cmake/Modules/CMakeTestCCompiler.cmake:67 (message):
  The C compiler

    "/usr/bin/cc"

  is not able to compile a simple test program.

  It fails with the following output:

    Change Dir: '/home/alarm/archpkg/kodi-addon-inputstream-adaptive-git/src/kodi-addon-inputstream-adaptive-git/kodi-addon-inputstream-adaptive-git-build/CMakeFiles/CMakeScratch/TryCompile-SzNkS5'

    Run Build Command(s): /usr/bin/cmake -E env VERBOSE=1 /usr/bin/make -f Makefile cmTC_def07/fast
    /usr/bin/make  -f CMakeFiles/cmTC_def07.dir/build.make CMakeFiles/cmTC_def07.dir/build
    make[1]: Entering directory '/home/alarm/archpkg/kodi-addon-inputstream-adaptive-git/src/kodi-addon-inputstream-adaptive-git/kodi-addon-inputstream-adaptive-git-build/CMakeFiles/CMakeScratch/TryCompile-SzNkS5'
    Building C object CMakeFiles/cmTC_def07.dir/testCCompiler.c.o
    /usr/bin/cc   -march=armv7-a -mfloat-abi=hard -mfpu=neon -O2 -pipe -fstack-protector-strong -fno-plt -fexceptions         -Wp,-D_FORTIFY_SOURCE=3 -Wformat -Werror=format-security         -fstack-clash-protection         -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer  -o CMakeFiles/cmTC_def07.dir/testCCompiler.c.o -c /home/alarm/archpkg/kodi-addon-inputstream-adaptive-git/src/kodi-addon-inputstream-adaptive-git/kodi-addon-inputstream-adaptive-git-build/CMakeFiles/CMakeScratch/TryCompile-SzNkS5/testCCompiler.c
    cc: error: unrecognized command-line option '-mno-omit-leaf-frame-pointer'; did you mean '-fno-omit-frame-pointer'?
    make[1]: *** [CMakeFiles/cmTC_def07.dir/build.make:78: CMakeFiles/cmTC_def07.dir/testCCompiler.c.o] Error 1
    make[1]: Leaving directory '/home/alarm/archpkg/kodi-addon-inputstream-adaptive-git/src/kodi-addon-inputstream-adaptive-git/kodi-addon-inputstream-adaptive-git-build/CMakeFiles/CMakeScratch/TryCompile-SzNkS5'
    make: *** [Makefile:127: cmTC_def07/fast] Error 2


  CMake will not be able to correctly generate this project.
Call Stack (most recent call first):
  CMakeLists.txt:5 (project)


-- Configuring incomplete, errors occurred!
==> ERROR: A failure occurred in build().
    Aborting...
```

The culprit was `cc: error: unrecognized command-line option '-mno-omit-leaf-frame-pointer'; did you mean '-fno-omit-frame-pointer'`.


It turns out that it was due to a wrong default configuration in
`/etc/makepkg.conf`. This issue was already reported twice in the forums, so I
didn't bother to report it as well:

 * <https://archlinuxarm.org/forum/viewtopic.php?f=15&t=16837&p=72357>
 * <https://archlinuxarm.org/forum/viewtopic.php?f=57&t=16830&p=72344>

Removing `-mno-omit-leaf-frame-pointer` from `/etc/makepkg.conf` got me a bit
further: 

```text
CMake Error at /usr/share/cmake/Modules/FindPackageHandleStandardArgs.cmake:230 (message):
  Could NOT find RAPIDJSON (missing: RAPIDJSON_INCLUDES)
Call Stack (most recent call first):
  /usr/share/cmake/Modules/FindPackageHandleStandardArgs.cmake:600 (_FPHSA_FAILURE_MESSAGE)
  FindRAPIDJSON.cmake:19 (find_package_handle_standard_args)
  CMakeLists.txt:18 (find_package)
```

This was just a missing dependency in the PKGBUILD, which I could just install
with `sudo pacman -S rapidjson`.

# Success

After the two bumps the build finished without further problems and I could
install the addon:

```bash
sudo pacman -U kodi-addon-inputstream-adaptive-git-21.4.4.Omega.r9.g109c0e7a-1-armv7h.pkg.tar.xz
sudo systemctl restart kodi.service
```

![Installing the YouTube addon]({static}/images/kodi/kodi-youtube-addon-install.png){width=100%}
