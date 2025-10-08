# Quality Assurance

Once you performed the actions described in *[Build Delivery](build-delivery.md)* (that is, you uploaded your Release Candidate build to the Developer Portal, it passed your internal tests, and you informed your Product Manager), the game will undergo a series of tests performed by our QA team.

- Aside from checking the game to make sure there are no major bugs or crashes, we will also check the game for any DRM, as all games on GOG have to be playable offline and without the Galaxy client
- If the build that you sent is not a Release Candidate (i.e. it is a preliminary or a beta version, not the one that is going to be released on GOG), basic checks will be performed — more thorough tests will be carried out on the Release Candidate build
- If any issues are found, you will be informed about them by your Product Manager; QA comment and logs will usually be attached to the message

# MOST COMMON ISSUES:
- Mentions/links to other stores - builds uploaded to gog (of games, demos, dlcs etc.) **SHOULDN’T** contain **ANY** mentions or links to other stores (like Steam or Epic); official websites/official social media links to Developer's Websites are, however, very much welcome  
- **DRM-free is a MUST, copy protection is a NO** - doesn’t matter if Your game implements sdk or is a vanilla version - it has to be DRM free; meaning: a game is shipped without technical restrictions on how it is installed, copied, or activated - this most often means, single player aspects of Your game have to work for users who don’t use GOG Galaxy (or other Game Clients like Steam), users who decide to log out from GOG Galaxy or don't have GOG Galaxy installed & users who disconnect from the Internet: [No DRM, No Problem](gog-and-steam.md)  
- If your game requires **additional dependencies (redists)** in order to work, please remember to add them to build: [Dependencies on Windows](bc-dependencies.md)  
- **Unreal Engine games (& games with launchers)** are a little bit different - please see “File Task in Unreal Engine Games” section: [File Tasks - GOG Developer Docs](bc-file-tasks.md)  
- Adding **Cloud Saves** to Your Game - Cloud Saves are enabled by us during Tests. Cloud Saves are possible only for titles that: store saves in a separate subfolder (we do not sync loose files nor saves stored in registry) & size of total savefiles quota don’t exceed 200mb  
- Most common issues with **DLCs: adding a DLC to a project** - [DLC - GOG Developer Tools](bc-dlc.md)  
- Most common issues with **DLCs - Goodies: Bonus goodies** (like OSTs, Artbooks) which are to be added separatelly and not as a part of game build, should be added as Extras, and not as installable DLCs - [DLCs, Extras & Demos - GOG Developer Docs](dlc-and-extras.md)  
- **Languages**: You have various possibilities with languages but please keep in mind, that if your game handles language change in-game you should choose „no separate language builds” as your option - [Languages - GOG Developer Docs](bc-languages.md)  
- **Systems requirements**: in a perfect scenario all new releases should be compatible with 2 latest LTS Operation System versions (so for example as of 2024: Windows 10&11 / MacOS 14&15 / Ubuntu 22.04 LTS&24.04 LTS)  
- Apart from GOG Galaxy builds (for Windows & MacOS), we offer additional **Offline Installers**: [Offline Installers - GOG Developer Docs](offline-installers.md) Please keep in mind that those installers won’t accept special characters: ~ # % * in in Your game’s name, so if possible, please refrain from using them  
 
# LINUX & MACOS SPECIFIC ISSUES:  
- **DLCs on Linux**: since there is no GOG Galaxy on this OS - please see “**DLCs in Builds for Linux**” section - [DLC Discovery - GOG Developer Docs](sdk-dlc-discovery.md)  
- **Libraries on Linux**: In case the game requires uncommon, 32bit or simply older versions of libraries, all of these should be included with the build  
- **Start.sh in Linux games**: Please make sure that the start.sh is linked to the correct binary file. There is no single guideline here, since there are substantial differences in how different engines handle running the game.  
- **Games requiring notarization on MacOS**: Even though we recommend adding MacOS games as App Bundles [Our Recommendations for macOS Game Structure - GOG Developer Docs](bc-recommendations-for-mac.md) for games requiring notarization this method might not work (as a result game might not launch correctly for end user); for such games, when uploading a build, please choose non default option - installation as **NORMAL FOLDER** - instead of App Bundle: [Installation Directory - GOG Developer Docs](bc-installation-dir.md)
- **Missing OpenSSL libraries in Linux games**: Please make sure to ship your Linux build of the game with all necessary OpenSSL libraries. Recently, we've noticed that many builds of Game Maker games are missing libcrypto.so.1.0.0. If your game uses OpenSSL libraries then please make sure to put them in a build of your game!

