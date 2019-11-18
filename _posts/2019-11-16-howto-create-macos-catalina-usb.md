---
layout: post
title: "How to Create a macOS Catalina USB Installer"
#description: ""
#date: 2019-07-04 00:39:00 +0800
tags: mac
---

1. Insert a USB drive larger than 8GB into your Macbook Pro.
2. Open App Store and download macOS Catalina.
3. After download completes, the installer application will launch. Quit this application. Do not select continue.
4. Check if this folder exists:
```
/Applications/Install\ macOS\ Catalina.app/Contents/SharedSupport
```
If the folder exists and has files in it, skip steps 5 to 7 below and proceed to step 8.
<br/>
If this folder does not exist, proceed to step 5.
5. Create the SharedSupport folder
```
mkdir /Applications/Install\ macOS\ Catalina.app/Contents/SharedSupport
```
6. Go to Applications and launch the Install macOS Catalina. You will need to do this because the download did not download all files into your mac. Proceed with the install until it asks you to restart. IMPORTANT: When you get the prompt to restart the computer, quit the installer or press Command-q. Do not restart your computer!
7. Locate this folder and copy all files under this directory into /Applications/Install\ macOS\ Catalina.app/Contents/SharedSupport/
```
cp /macOS\ Install\ Data\* /Applications/Install\ macOS\ Catalina.app/Contents/SharedSupport/
```
8. Open a terminal and run the createinstallmedia command as shown below:
<br/>
<br/>
```
sudo /Applications/Install\ macOS\ Catalina.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled
```
where /Volumes/Untitled is the path to your USB drive.
7. Remove the folder: /macOS\ Install\ Data\
