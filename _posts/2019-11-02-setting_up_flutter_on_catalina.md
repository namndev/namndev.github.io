---
layout: post
title: Setting up Flutter on macOS Catalina
image: https://miro.medium.com/max/10000/1*A1TtHIM4tL0ejuQOjVSZUw.png
tags: [Flutter, Mobile]
comments: true
---

Flutter has good [installation documentation](https://flutter.dev/docs/get-started/install), but to install it on macOS Catalina you need to fly above some further obstacles. I’m writing some instructions here for reference and to help out any other potential lost programming souls.

## Install the Flutter SDK

- Download the latest stable release from [here](https://flutter.dev/docs/get-started/install/macos).
- Unzip and move the flutter folder under `/Users/<your_user>/developement` (create one if you don’t have one and don’t change this in the future or you’ll have to update the path again).
- Update your path:

```bash
1. Determine the default shell on your Mac by following the instructions here. Basically, if you created a new user account in macOS Catalina, zsh (Z shell) will be your default shell. In all other cases, bash will still be your default shell.
        
2. Launch the Terminal.
        
3. Type vim .bash_profile (or vim .zprofile if zsh is your default shell) to open the vim editor.

4. Type i to enter INSERT mode (or esc to exit INSERT mode).

5. Type export PATH="$PATH:[YOUR_PATH]/flutter/bin" replacing [YOUR_PATH] with the path to the folder where you moved the flutter folder earlier ex. export PATH="$PATH:/Users/<your_user>/development/flutter/bin"

6. Type esc, then :wq! to save and exit.

7. Quit the Terminal and open it again to refresh.

8. Type echo $PATH to check that the path was correctly added.
        
9. Type which flutter to verify the flutter command is available.
        
10. Type flutter --version to check the Flutter version.
```

- If running the flutter command fails and you get an `xcrun error`, try typing `xcode-select --install` to update the Command Line Tools and then restart the Terminal again.
- Now you might get an error saying: ***“dart” can’t be opened because Apple cannot check it for malicious software***. The only option I found to remedy this is to globally disable Gatekeeper by typing `sudo spctl --master-disable`
- Now the flutter command should be working, so we can type `flutter doctor` to check for missing dependencies.

## Install Xcode
To develop apps for iOS, we need to install Xcode.

- Install the latest version from the App Store on your Mac.
- Configure the command-line tools by running `sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer` and `sudo xcodebuild-runFirstLaunch`
- Run `sudo xcodebuild -license`	 to accept the license agreement.
- Later on, you might get an error saying: ***“idevice_id” cannot be opened because the developer cannot be verified***. Or the equivalent message for ***“ideviceinfo”, “idevicesyslog”*** or ***“iproxy”***. You can type the lines below into your Terminal to silence the warnings, replacing `<your_user>` like before.

```bash
For "idevice_id" error:
sudo xattr -d com.apple.quarantine /Users/<your_user>/development/flutter/bin/cache/artifacts/libimobiledevice/idevice_id

For "ideviceinfo" error:
sudo xattr -d com.apple.quarantine /Users/<your_user>/development/flutter/bin/cache/artifacts/libimobiledevice/ideviceinfo

For "idevicesyslog" error:
sudo xattr -d com.apple.quarantine /Users/<your_user>/development/flutter/bin/cache/artifacts/libimobiledevice/idevicesyslog

For "iproxy" error:
sudo xattr -d com.apple.quarantine /Users/<your_user>/development/flutter/bin/cache/artifacts/usbmuxd/iproxy
```

If there is another message that is not in this list, you can search for its path in `flutter/bin/cache/artifacts` and type the equivalent message to the above in your Terminal.

## Test the iOS Simulator

- Open by typing in Terminal `open -a Simulator`
- To create a sample app from the Terminal:

```bash
1. cd to the desired directory where you want your project to be.
2. type flutter create my_app
3. cd my_app
4. With the Simulator open, type flutter run
```
You should see an app running on your Simulator now where you can tap a button to increase the counter.

## Install Android Studio
- Download the latest version from [here](https://developer.android.com/studio).
- Run the installation wizard with all the default settings.
- When trying to install Intel HAXM, Apple will show a warning. We need to go to `System Preferences -> Security & Privacy`, click on the lock symbol and enter your password to make changes. There should be a message under ‘Allow apps downloaded from’, click on Allow and wait a bit for the installation to finish. This is necessary to enable [VM acceleration](https://developer.android.com/studio/run/emulator-acceleration). We can check whether HAXM is successfully installed by typing in the Terminal: `kextstat | grep intel` and we should get a message containing `com.intel.kext.intelhaxm` if successful.
- Finally, don’t forget to copy Android Studio into your Applications folder.

## Test the Android Emulator

***Create a Virtual Device***

- Open Android Studio and tap on the `Configure` button at the bottom.
- Go to `AVD Manager -> Create Virtual Device`.
- Choose a device (ex. Pixel 2).
- Download one of the recommended system images (ex. Android 10.0 (Q)) and tap `Next`.
- Under Emulated Performance, select `Hardware — GLES 2.0` (or leave it on `Automatic` if it can’t be selected). It seems to work fine with hardware acceleration on my Mac as I’m getting the message ***‘Using hardware rendering with device Android SDK built for x86.’*** when I’m typing `flutter run`.
- Tap on ***Finish*** and then on the ***play button*** to launch the AVD in the Emulator.

### Installing legacy Java on macOS Catalina
And for the final big hurdle, we need to install a legacy version of Java, in quite a hacky way.

- If you try to run the sample app by typing ***`flutter run`***, you might get a popup that you need to install Java for Mac. It also points you to the Java website where you happily download the latest version of Java ***(don’t do it!)*** and install it ***(don’t do it!)***. But this doesn’t make the popup go away, because you need a ***legacy version of Java for Mac***. [This version is 2017–001 and you can get it from Apple](https://support.apple.com/kb/DL1572?locale=en_GB).
- If you, like me, naively installed the latest version of Java, you can [uninstall it by following these instructions](https://www.java.com/en/download/help/mac_uninstall_java.xml).
- Now it gets even trickier. Even if you uninstalled all existing versions of Java, you might still get an error message when you’re trying to install the legacy version saying: ***“Java for macOS 2017–001 can’t be installed on this disk. A newer version of this package is already installed.”***. To get around this, you can [implement this nice hack that I found here](https://www.harrisgeospatial.com/Support/Self-Help-Tools/Help-Articles/Help-Articles-Detail/ArtMID/10220/ArticleID/23780/Mac-OS-Catalina-1015-ENVIIDL-and-Legacy-Java-6-Dependencies), or follow the instructions below.

```bash
1. Open Launchpad -> Other -> Script Editor

2. Select New Document and copy the following text:
set theDMG to choose file with prompt "Please select javaforosx.dmg:" of type {"dmg"}
do shell script "hdiutil mount " & quoted form of POSIX path of theDMG
do shell script "pkgutil --expand /Volumes/Java\\ for\\ macOS\\ 2017-001/JavaForOSX.pkg ~/tmp"
do shell script "hdiutil unmount /Volumes/Java\\ for\\ macOS\\ 2017-001/"
do shell script "sed -i '' 's/return false/return true/g' ~/tmp/Distribution"
do shell script "pkgutil --flatten ~/tmp ~/Desktop/ModifiedJava6Install.pkg"
do shell script "rm -rf ~/tmp"
display dialog "Modified ModifiedJava6Install.pkg saved on desktop" buttons {"Ok"}

3. Select Script -> Compile and then Script -> Run.

4. You will get a popup prompting you to select the javaforosx.pkg file. This is the file in JavaForOSX.dmg that you downloaded from Apple earlier (after you mount it).
            
5. Running the script will create a ModifiedJava6Install.pkg on your desktop. Run this ModifiedJava6Install.pkg to install the legacy Java version.
```

## Test the Emulator
- Go to the directory where we saved ***my_app*** before and type ***`flutter run`***
- The sample app should run on the Android emulator now, where you can again tap on a button to increase the counter!
- Finally, type ***`flutter doctor --android-licenses`*** to accept the Android licenses.

## Dr. Flutter
We should now have the sample app running on both iOS Simulator and Android Emulator!

Type once again ***`flutter doctor`*** to see if there are any issues left. There shouldn’t be any, except for the missing devices warning if you don’t have the Simulator or Emulator open.

You can type ***`flutter devices`*** to see all currently connected devices. If the Simulator or Emulator are not open, they won’t show up here.

You can also type ***`flutter emulators`*** to see all available simulators/emulators. We should see 2 now, the Pixel 2 and the iOS Simulator.

## Support for web
We can also add support for the web by following [these instructions](https://flutter.dev/docs/get-started/web), although it’s still in early support and only on the master branch (not recommended for production).

## Install the Flutter and Dart plugins for Android Studio

Finally, in order to develop with Flutter we need to install the Flutter and Dart plugins for Android Studio.

- Open Android Studio and go to `Configure -> Plugins`.
- Search for the `Flutter` plugin and install it. This will also install the Dart plugin as it is a dependency.
- ***Restart*** the IDE when installation is done.
- We can now do the [***Test Drive***](https://flutter.dev/docs/get-started/test-drive) by building our first Flutter app!

## Upgrade Flutter
To [upgrade Flutter](https://flutter.dev/docs/development/tools/sdk/upgrading), simply run `flutter upgrade`.

You can also learn more about Flutter’s release channels [here](https://github.com/flutter/flutter/wiki/Flutter-build-release-channels).

> Thanks!
