---
layout: post
title: "Setting up Kinect for Linux"
categories: Tutorial
author: "Muhammad Taha"
---

Windows is what I usually use. It's what I grew up with. It's what I know best. Nevertheless, it hasn't adapted very well to ARM. There have been push to bring Windows 10 to ARM from Microsoft since 2017 but 
those haven't been very successful. Hence, it's really hard to get Windows to run on a microcomputer like Raspberry Pi. So, going with Linux was the obvious choice. 

I wanted to interact with my Pi using hands-free gestures so I went with a Kinect for tracking my hands (which I later realized is succeeded by PoseNet). 

Here's how you would go about setting it up and running on Raspberry Pi OS:

## Hook it up
Plug the device in then run `lsusb -v -d 045e:02ae | grep -e "Bus\|iSerial"` or simply `lsusb -v` as `sudo` and if you see something like this:
```bash
iManufacturer           2 Microsoft
iProduct                1 Xbox NUI Camera
iSerial                 3 XXXXXXXXXXXXXXXX
```
Then Linux recognizes your Kinect.

## Installing dependencies
Install the following dependencies before attempting to install `libfreenect`:
```bash
sudo apt-get install git-core cmake freeglut3-dev pkg-config build-essential libxmu-dev libxi-dev libusb-1.0-0-dev
```

## Installing Libfreenect
Clone the current branch from github like so:
`git clone git://github.com/OpenKinect/libfreenect.git`
Then install using the following commands:
```bash
cd libfreenect
mkdir build
cd build
cmake -L ..
make
sudo make install
sudo ldconfig /usr/local/lib64/
```

## Setting up permissions
```bash
sudo adduser $USER video
sudo adduser $USER plugdev
```

## Kinect rules
Use nano to create the following file: `sudo nano /etc/udev/rules.d/51-kinect.rules` then fill it with the following text:
```
# ATTR{product}=="Xbox NUI Motor"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02b0", MODE="0666"
# ATTR{product}=="Xbox NUI Audio"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02ad", MODE="0666"
# ATTR{product}=="Xbox NUI Camera"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02ae", MODE="0666"
# ATTR{product}=="Xbox NUI Motor"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02c2", MODE="0666"
# ATTR{product}=="Xbox NUI Motor"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02be", MODE="0666"
# ATTR{product}=="Xbox NUI Motor"
SUBSYSTEM=="usb", ATTR{idVendor}=="045e", ATTR{idProduct}=="02bf", MODE="0666"
```

## Running Kinect `gl-view`
Restart your computer after creating the `.rules` file then attempt:
`freenect-glview`.

## Dealing with errors
If `freenect-glview` fails, then try it with sudo. 
If it throws the following error:
> Kinect camera test
> Number of devices found: 1
> Found sibling device [same parent]
> Failed to set the LED of K4W or 1473 device: -1
> Found sibling device [same parent]
> Could not open device: -1
> Could not open device
Then try `freenect-micview` or `freenect-camtest`. If either of those fail then you're probably missing an `audios.bin` file.

You can get it by using python (check your environment to see if you have 2 or 3 installed). Use the python version that you mainly use.
I'm taking Python 3 here as an example.
```
sudo python3 libfreenect/src/fwfetcher.py
```
Wait a bit for it to download. Then find the directory where audios.bin is downloaded (probably in `libfreenect` or `libfreenect/src`) and copy it to `/usr/local/shared/libfreenect`.
Then restart. 

Finally running `freenect-micview` should work and a screen should pop up showing a visualization of sound waves. Exit gracefully from the program by either pressing `q` or `ctrl+c`.
Do not press `ctrl+z` as that would not unlock the process and you'd have to restart your system.

Another important thing to note it if you ever get the `could not open device` error. Try the following things:
* Unplug and plug Kinect back in
* Try it with `sudo`
* Run `freenect-micview`

That should hopefully show a depth-view and rgb cam side-by-side.

