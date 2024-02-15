# Guidelines for Linux Games on GOG

The following guide covers the most common aspects of making a Linux version of your game. Some are specific to popular game engines, while others are related to common mistakes made by developers not familiar with the platform. Almost all of them are easy to fix once you learn about them.

The guide is written with the desired “click and run” user experience in mind — this means that no additional steps are required from the user to make the game work.

Before submitting a Linux build to us for release, we strongly recommend that you test your build using a freshly installed Ubuntu 16.04 or 18.04 system (the GOG supported distributions). This will not only prevent the occurrence of many of the issues highlighted in this article, but will also ensure that your game is truly distribution platform agnostic.

Should you have any questions or require any assistance, don’t hesitate to contact us by [submitting a ticket](https://devportal.gog.com/support/contact) or sending an email to our [Product team](basic-game-assets.md#contact-us).

## 64- and 32-bit Architecture Support

### Problem

In contrast to Windows and macOS/OSX, 32-bit software does not function by default on Linux 64-bit systems. 64-bit systems are currently the vast majority (around 95%) of all Linux gaming desktop installations. This means that shipping only a 32-bit DRM-free build is shipping a broken build that doesn’t start at all.

The user can work around this issue by manually installing the compatibility libraries user-side using system tools, but this practice is suboptimal and leads to a poor user experience.

Furthermore, providing a 64-bit system user with a 32-bit build inadvertently makes the users susceptible to bugs and issues that are 32-bit specific, e.g. not being compatible with big drive partitions (see: [Large File Support](http://users.suse.com/~aj/linux_lfs.html)).

### Solution

The ideal solution is to build your game on a 64-bit Linux system (preferably Ubuntu 18.04). If your game uses the Unity engine, you can use the “*64-bit Linux*” (or “*Universal Linux*” if you require both 64- and 32-bit binaries) export in Unity Editor.

!!! Important
    Sometimes it is not possible to deliver a 64-bit build. Some tools, like Game Maker or Wine, allow only 32-bit Linux build and some games use a very specific 32-bit code or middleware. In these instances, please let us know the additional libraries (including 32-bit compatibility libraries) that your game requires to run on Linux and we will inform our users.

## Missing Dependencies

### Problem

Sometimes, games on Linux are shipped without the libraries that the game uses and are required to start the game. As a result, the shipped build is broken, because it doesn’t start at all.

The user can work around this issue by manually installing the required dependencies user-side using system tools, but this practice is suboptimal and leads to a poor user experience.

Windows games install redistributables at the time of installation, while Linux game builds are expected to contain all of the needed libraries needed to run the game on a freshly installed Linux system (e.g. Ubuntu).

### Solution

When transferring your build to us, please include the necessary libraries along with your game. Please either specify the [rpath](https://en.wikipedia.org/wiki/Rpath) with the location of these libraries, or ship a launch shell script that will point the binary to the location of these libraries (using the  `LD_LIBRARY_PATH` variable).

To ensure everything is working correctly, we recommend testing your game on a freshly installed Ubuntu 16.04 or 18.04 system — this will confirm if your game runs without installing anything user-side and without having the non-standard libraries present on users’ systems. Additionally, Valve’s Steam client includes its own set of libraries (known as *Steam runtime*), which provides the user with the required libraries for their games. Therefore, It is vital that you test on a cleanly installed Linux machine to ensure that your game includes the required dependencies across all distribution platforms.

!!! Important "Wwise Sound Plugin Issue"
    There is a known issue that affects Unity engine games using the Wwise sound plugin (*libAkSoundEngine.so*); this plugin has a hidden dependency on the SDL2 library and without it won’t run properly (often will only boot to a black screen).

## Missing DLCs

### Problem

A game using the [DLC Discovery](sdk-dlc-discovery.md) method ([`galaxy::api::IsDlcInstalled()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IApps.html#a46fbdec6ec2e1b6d1a1625ba157d3aa2)) from the GOG GALAXY SDK results in a build, in which DLCs are present for both Windows and macOS, but not for Linux (empty DLC installers).

### Solution

**The GOG GALAXY SDK is not available on Linux**, so the DLC Discovery method can’t be used on Linux operating systems. If needed, you can add the DLC and required files to the game project: the best way to do so is to create a dummy .txt file (one for each DLC) and connect it to the game bin (if the file is present, a user has an access to the DLC content), then add these files as DLC depots in [Build Creator](bc-quick-start.md).

Another way is to add code to the bin file that checks for the presence of the file named *goggame-<dlcID\>.info*, which is always automatically generated by our installer. With this method, you don’t need to add anything to the DLC depot.

## Locale Issues

### Problem

Some games encounter unexpected issues, such as glitched textures, games not loading, or broken menus, when they are ran on a Linux system that uses a language setting other than English, like French or Polish. This can be caused by language specific issues, such as the decimal separator varying between languages (e.g. a quarter is written 0.25 in English, but as 0,25 in Polish).

This issue typically affects games written in C# and those built within the Unity Engine.

This can be worked around by having the user force the most basic locale setting (see: [C locale](http://unix.stackexchange.com/a/87763)) before running the game from the terminal (using the following command: `export LC_ALL=C`). Encouraging users to do so is suboptimal and leads to a poor user experience.

### Solution

We recommend specifying locale-agnostic number formatting in your project OR including the export `LC_ALL=C` command, setting C locale, in the game launch script.

To ensure this is working correctly, please test your game on Linux systems using various locale.

!!! Important
    Errors caused by this issue are pretty random and sometimes difficult to debug/reproduce. Therefore, even if you do not encounter any issues when testing, we strongly recommend following the above solutions.

## Packaging and the Executable Bit

### Problem

Linux, like macOS, has a special way of marking the executable files using the file mode bits. These are preserved on Linux filesystems, but lost when a game is packaged on Windows. You must mark all the game executables, like binaries and shell scripts, manually (for example, by using the `chmod +x <binary>` command) before packaging the game (e.g. compressing the game into a zip archive). Otherwise, your game will not run directly after unpacking the game build.

### Solution

We recommend building and packaging your game on a Linux system to ensure that the executable file bit is preserved.

## More Information

If you wish to learn more about building games for Linux, you can check the following content created by Linux game porting professionals:

- Ethan Lee
    - [https://www.youtube.com/watch?v=B83CWUh0Log](https://www.youtube.com/watch?v=B83CWUh0Log)
    - [http://www.flibitijibibo.com/magfest2016/](http://www.flibitijibibo.com/magfest2016/)
    - [https://gist.github.com/flibitijibibo/b67910842ab95bb3decdf89d1502de88](https://gist.github.com/flibitijibibo/b67910842ab95bb3decdf89d1502de88)

- Aaron Melcher
    - [https://www.youtube.com/watch?v=WIbXifb8WGY](https://www.youtube.com/watch?v=WIbXifb8WGY)
    - [http://knockoutgames.co/talks/your_code_sucks_lightning.pdf](http://knockoutgames.co/talks/your_code_sucks_lightning.pdf)

