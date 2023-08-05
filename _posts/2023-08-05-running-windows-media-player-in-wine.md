---
layout: post
title: Running Windows Media Player in Wine
cover-img: assets/img/posts/2023-08-05-running-windows-media-player-in-wine/banner.jpg
cover-img-credit: ''
thumbnail-img: assets/img/posts/2023-08-05-running-windows-media-player-in-wine/thumbnail.jpg
share-img: assets/img/posts/2023-08-05-running-windows-media-player-in-wine/banner.jpg
tags: [windows, media]
---

Windows Media Player has actually always been my preferred choice for managing my music library. It's simple, it has a decent amount of functionality, it can handle most formats that I use regularly, and it looks pretty nice. I'd like to be able to use it on my modern Kubuntu desktop computer, but it seems there are a number of pitfalls when getting it to work with Wine. This page documents my experimentation and research for this.

## WMP10 Walkthrough

Strangely, the most useful resource I found on this was [this YouTube video](https://www.youtube.com/watch?v=0tQHrahbKBs) detailing how to get Windows Media Player 10 to work with Wine. To avoid having to scrub through the rather tedious video, the process is as follows:

1. Install [PlayOnLinux](https://www.playonlinux.com/), which although targeted towards getting Windows games running on Linux, is a useful Wine prefix manager for your system. You can set up specific versions of wine under specific prefixes, which is exactly what we want for the super-custom WMP environment.
2. Go to `Tools -> Manage Wine Versions`, and under the x86 versions, install `3.0.4`. This specific version is, for some reason, the only one that supports WMP.
3. Go to `Configure` and click `New` in the bottom left. Create a new 32-bit virtual drive using Wine `3.0.4`. Name it whatever you like (I'll assume from here on that it's called `WMP`).
4. Once created, select the virtual drive in the configuration window, go to the `Wine` tab and choose `Configure Wine`.
5. In the Wine configuration window, go to the `Applications` tab and for `Default Settings`, set the Windows version to be `Windows XP`. Next, choose `Add Application`, type `wmplayer.exe` into the name box in the file chooser, click `Open`, and set the Windows version for this to be `Windows XP` as well. Both of these steps are required, as the default setting will need to be modified later as part of the install process. Keep the configuration window open - we'll return to it in a second.
6. Download [jscript.dll](http://www.thehandofagony.com/alex/dll/jscript.dll), [urlmon.dll](http://www.thehandofagony.com/alex/dll/URLMON.DLL), [devenum.dll](https://www.dlldump.com/dllfiles/D/devenum.dll) and [quartz.dll](https://www.dll-files.com/download/d23e4614004ccee845ef3cff5a0d5546/quartz.dll.html?c=b3ZnV3N0dUNHSTZ6dUZvdTJqSjBhZz09). Rename all of these DLLs to be lowercase if they are not already. Go to the Wine prefix for the virtual drive you set up earlier (usually `~/PlayOnLinux's virtual drives/WMP`), navigate to `drive_c/windows/system32`, and paste the four aforementioned DLLs into this folder, overwriting any existing DLLs.
7. Go back to the Wine configuration window that was opened in step 5. In the `Applications` tab, select `Default Settings`. Then go to the `Libraries` tab, and in the `New override for library` dropdown box, type `urlmon` and then click `Add`. This adds the `urlmon` DLL override for all applications.
8. Switch to the `Applications` tab, select `wmplayer.exe`, and then switch back to the `Libraries` tab again. This time, type `quartz` into the dropdown and click `Add`, and then type `devenum` and click `Add`. This adds overrides for the `quartz` and `devenum` DLLs specifically for Windows Media Player.
9. Assuming `wmplayer.exe` is still selected in the `Applications` tab, go to the `Graphics` tab and deselect the tick box for `Allow the window manager to control the windows`. This means that Windows Media Player itself can take control for full-screen playback.
10. Click `Apply` in the Wine configuration window, but don't close it - we'll come back to it again later.
11. Back in the PlayOnLinux configuration window that was opened in step 3, under the `Wine` tab, click `Command prompt`. In the command prompt that opens, execute `regsvr32 jscript.dll`. This should bring up a success message reporting that the DLL was successfully registered. Close the command prompt.
12. In the PlayOnLinux configuration window, go to the `Miscellaneous` tab and click `Run a Windows executable (.exe) in this virtual drive`. This is where we install Windows Media Player itself. In the file browser that appears, select your WMP installer executable, and this should bring up the install wizard. Click through the stages and allow it to complete. *TODO: It'd be useful if we linked to a WMP10 installer here, but I can't seem to find the one I used...*
13. Once installed, Windows Media Player will appear in a very broken-looking state. To close it, you can click on the Windows logo in the top left, and then choose `File -> Exit`. The next few steps will fix the player.
14. In the Wine configuration window that you left open in step 10, go to the `Applications` tab and choose `Default Settings`. Change the Windows version in the dropdown to `Windows 2000`.
15. Download the [Windows Media Encoder 9](https://download.cnet.com/Windows-Media-Encoder/3001-2212_4-14887.html) installer. In the PlayOnLinux configuration window, under the `Miscellaneous` tab, click `Run a Windows executable (.exe) in this virtual drive` and select this installer executable. Click through the installer and allow it to complete.
16. Next, in the PlayOnLinux configuration window, go to the `Install Components` tab and install `Microsoft Core Fonts`, `wmpcodecs`, and `wmp10`. Note that some downloads don't seem to work - the video says it's OK, and just to continue regardless and allow whichever installers you download successfully to run. It might be worth seeing whether we can obtain other copies of the installers that are downloaded here - from the video, PlayOnLinux is looking for [http://itc.edu.stockholm.se/shareware/arkiv/PC/WMEncoder.exe](http://itc.edu.stockholm.se/shareware/arkiv/PC/WMEncoder.exe). PlayOnLinux itself looks for [https://web.archive.org/web/20121003223319/http://download.microsoft.com/download/8/1/f/81f9402f-efdd-439d-b2a4-089563199d47/WMEncoder.exe](https://web.archive.org/web/20121003223319/http://download.microsoft.com/download/8/1/f/81f9402f-efdd-439d-b2a4-089563199d47/WMEncoder.exe) when I try to install this component, and then says the files do not match.
17. Finally, create a shortcut by going to the `General` tab in the PlayOnLinux configuration window, clicking `Make a new shortcut from this virtual drive`, and choosing `wmplayer.exe`.

## Attempts At Getting Windows Media Player 11 To Work

The process for WMP11 starts off the same, but at step 12 the installer will get stuck as it is not able to verify that the install of Windows is genuine. If the `Validate` button in the installer greys out when clicked and never becomes active again, cancel and re-run the installer: at this point it will say that the copy of Windows is not genuine.

To fix this, I followed the instructions presented [here](https://askubuntu.com/a/84188/209850): select `Default settings` in the Wine configuration window's `Applications` tab, and set the Windows version to be `Windows 2003`. Then at the EULA screen in the installer, **before** accepting the EULA, switch the Windows version back to `Windows XP`. Once it reaches its end, the installer will say that it failed to install WMP, but according to the aforementioned instructions, this doesn't matter.

Unfortunately, I never got WMP11 to run acceptably - it always looked graphically mutilated and did not respond to input. Another avenue to explore might be to check newer Wine versions and see if it works there.
