# Building Cataclysm-DDA on Mac OS X

To build Cataclysm on Mac you'll have to get XCode with command line tools (or just download them separately from https://developer.apple.com/downloads/) and Homebrew package manager.



## Step 1: Install SDL2

SDL2, SDL2\_image, and SDL2\_ttf are needed for the tiles build.

### Option 1: manual download and install

* [**SDL2 framework**](http://www.libsdl.org/download-1.2.php)
    * Direct download: http://www.libsdl.org/release/SDL2-2.0.3.dmg

* [**SDL2\_image framework**](http://www.libsdl.org/projects/SDL_image/)
    * Direct download: http://www.libsdl.org/projects/SDL_image/release/SDL2_image-2.0.0.dmg

* [**SDL2\_ttf framework**](http://www.libsdl.org/projects/SDL_ttf/)
    * Direct download: http://www.libsdl.org/projects/SDL_ttf/release/SDL2_ttf-2.0.12.dmg

Copy `SDL2.framework`, `SDL2_image.framework`, and `SDL2_ttf.framework`
to `/Library/Frameworks` or `/Users/name/Library/Frameworks`.

### Option 2: Homebrew package manager

Alternately, shared libraries (libSDL2, libSDL2\_image, libSDL2\_ttf) can be used
instead of frameworks. Install with a package manager (Fink, MacPorts,
Homebrew, pkgsrc) or build and install from source.

For Homebrew (can also install lua):

    brew install sdl2 sdl2_image sdl2_ttf



## Step 2: Install ncurses and gettext

ncurses and gettext are needed for localization.
Install with a package manager, or build from source and install.
**NOTE: ncurses needs wide character support enabled.**

For Homebrew:

    brew tap homebrew/dupes
    brew install gettext ncurses
    brew link --force gettext ncurses

**After you build Cataclysm** you might want to to unlink gettext and ncurses with: 

    brew unlink gettext ncurses 
    
Reason: if you build other software, these versions might conflict with what the other software expects.  



## Step 3: Compile

### Sample compilation commands for current versions of OS X

One of the following commands will likely work for you. Tweak flags to suit your needs.

build a release version, use `SDL` + graphical tiles, don't use `gettext`:

    $ make NATIVE=osx OSX_MIN=10.6 RELEASE=1 TILES=1 LOCALIZE=0
    
build a release version, use `SDL` + graphical tiles, link to libraries in the OS X `Frameworks` folders, don't use `gettext`:

    $ make NATIVE=osx OSX_MIN=10.6 RELEASE=1 TILES=1 FRAMEWORK=1 LOCALIZE=0

build a release version, use `ncurses`, don't use `gettext`:

    $ make NATIVE=osx OSX_MIN=10.6 RELEASE=1 LOCALIZE=0

build a debug version, use `SDL` with just the ASCII tiles, link to libraries in the OS X `Frameworks` folders, don't use `gettext`:

    $ make NATIVE=osx OSX_MIN=10.6 SDL=1 FRAMEWORK=1 LOCALIZE=0 

### Make Options

Description of the options used above. Tweak until things work. More notes are in the `Makefile`.

* `FRAMEWORK=1` attempt to link to libraries under the OS X `Frameworks` folders; omit to use the usual libsdl, libsdl\_image, libsdl\_ttf (e.g. leave out when you `brew install` the packages).
* `LOCALIZED=0` disable localization (to get around possible `gettext` errors if it is not setup correctly); omit to use `gettext`. 
* `NATIVE=osx` build for OS X.
* `OSX_MIN=version` sets `-mmacosx-version-min=` (for OS X > 10.5 set it to 10.6 or higher); omit for 10.5.
* `RELEASE=1` build an optimized 'release' version; omit for debug build.
* `SDL=1` build the SDL version with graphical ASCII; omit to use `ncurses` interface.
* `TILES=1` build the SDL version with graphical tiles (and graphical ASCII). Setting this includes `SDL=1`, no need to set that flag too.



## Step 4: Run

    $ ./cataclysm

or

    $ ./cataclysm-tiles



## Step 5 (optional): Application packaging

If you just want to build from source and run the game, this step is not for you.

Create new folder and name it `Cataclysm.app`.

Put compiled binaries (`./cataclysm-tiles` and/or `./cataclysm`) with `./gfx/` and `./data/` folders inside `/Cataclysm.app/Contents/Resources/`.

To bundle SDL libs copy `SDL2.framework`, `SDL2_image.framework`, and `SDL2_ttf.framework` to `/Cataclysm.app/Contents/Resources/libs/` or shared libs homebrew installed from `/usr/local/Cellar/sdl2*/version/lib/lib*`.

Create folder `/Cataclysm.app/Contents/MacOS` and file ./Cataclysm within it with this content:

```bash
#!/bin/sh
PWD=`dirname "${0}"`
OSREV=`uname -r | cut -d. -f1`
if [ "$OSREV" -ge 11 ] ; then
   export DYLD_LIBRARY_PATH=${PWD}/../Resources/libs
   export DYLD_FRAMEWORK_PATH=${PWD}/../Resources/libs
else
   export DYLD_FALLBACK_LIBRARY_PATH=${PWD}/../Resources/libs
   export DYLD_FALLBACK_FRAMEWORK_PATH=${PWD}/../Resources/libs
fi
cd "${PWD}/../Resources/"; ./cataclysm-tiles
```

### Creating a DMG

* Create an new folder named Cataclysm
* Move your Cataclysm.app into it
* Start Disk Utility
* File / New -> Disk Image From Folder
* Select the Cataclysm folder you created above.

Done!


## Troubleshooting

### ISSUE: crash on startup due to libint.8.dylib aborting

Basically if you're compiling on Mountain Lion or above, it won't be possible to run successfully on older OS X versions due to libint.8 / pthreads version issue.

See below (quoted form https://wiki.gnome.org/GTK+/OSX/Building)

"There's another issue with building on Lion or Mountain Lion using either "native" or the 10.7 SDK: Apple has updated the pthreads implementation to provide recursive locking. This would be good except that Gettext's libintl uses this and if the pthreads implementation doesn't provide it it fabricates its own. Since the Lion pthreads does provide it, libintl links the provided function and then crashes when you try to run it against an older version of the library. The simplest solution is to specify the 10.6 SDK when building on Lion, but that won't work on Mountain Lion, which doesn't include it. See below for how to install and use XCode 3 on Lion and later for building applications compatible with earlier versions of OSX."

Workaround: install XCode 3 like that article describes, or disable localization support in Cataclysm so gettext/libint are not dependencies. Or else simply don't support OS X versions below 10.7.

### ISSUE: Colours don't show up correctly.

Open Terminal's preferences, turn on "Use bright colors for bold text" in "Preferences -> Settings -> Text"

