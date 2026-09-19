# install_pi_pico_sdk_and_debugger
## This is a repo where i will add a script or possible more, to facilitate the installation of pi pico sdk and debugger for arch users, or maybe it will be just a tutorial, depends on the time i have at hand



So, here it goes...

### 1) Install gdb, because gdb-multiarch is somehow not cool anymore in arch linux land, it is needed to install gdb from here: https://gitlab.archlinux.org/archlinux/packaging/packages/gdb
1.1 ) Download it from here: [GDB](https://gitlab.archlinux.org/archlinux/packaging/packages/gdb)</br><br>
1.2 ) Go [here](https://gitlab.archlinux.org/archlinux/packaging/packages/gdb/-/commit/baa8b09ae93345ed66c028e6d1479e0ac1cc8311) and then look at the .SRCINFO and PKGBUILD files and take those modifications to the package downloaded at **1.1)** to enable multiarch.</br><br>
1.3 ) For the love of all that is good and nice, don't use FISH console, cause for who knows what reason it will muck (yes it is muck not *uck) up your build process, cause it apparently has an issue with "=" da equals sign.</br>
      Just do yourself a favor and sudo pacman -Syyu gnome-terminal   I know, i know, you actually have to remember what you type, but hey, life is hard :)</br><br>
1.4 ) When all of the above are peachy, just makepkg -sci    and wait for a thousand eons.</br><br>
1.5 ) GDB with multiarch support is now installed, test it, a good start would be this vid: https://www.youtube.com/watch?v=MTkDTjdDP3c</br><br>

### 2) Install OpenOCD
2.1 ) This is a good place to start: https://pip-assets.raspberrypi.com/categories/610-raspberry-pi-pico/documents/RP-008276-DS-2-getting-started-with-pico.pdf section A.6. Debug with OpenOCD</br><br>
2.1.1) Also it is good to check out how openocd is put together: https://deepwiki.com/raspberrypi/pico-sdk-tools/4.1-openocd-(debug-and-programming)</br><br>
2.2 ) git clone https://github.com/raspberrypi/openocd.git --branch sdk-2.2.0</br><br>
2.3 ) cd openocd</br><br>
2.4 ) git submodule init</br><br>
2.5 ) git submodule update</br><br>
2.6 ) ./bootstrap</br><br>
2.7 ) ./configure --disable-werror --enable-internal-jimtcl</br><br>
2.8 ) Get openocd rules from here: https://github.com/raspberrypi/openocd/blob/sdk-2.0.0/contrib/60-openocd.rules  and copy the file to /etc/udev/rules.d/</br><br>
2.9 ) create a group called plugdev: sudo groupadd plugdev</br><br>
2.10 ) check if it is properly created: sudo tail /etc/group</br><br>
2.11 ) add yourself to the group: sudo usermod -a -G plugdev $USER</br><br>
2.12 ) instead of the dialout group for serial communication, on arch like systems, add yourself to the uucp group: sudo usemod -a -G uucp $USER</br><br>
2.13 ) reload the udev rules: sudo udevadm control --reload</br><br>

### 3) Install VS Code, cause at least this will go easy:
3.1 ) Get it from here: https://aur.archlinux.org/packages/visual-studio-code-bin</br><br>
3.2 ) git clone https://aur.archlinux.org/visual-studio-code-bin.git</br><br>
3.3 ) cd visual-studio-code-bin</br><br>
3.4 ) makepkg -sci</br><br>
3.5 ) When it is finally instaled, run it and from the Extensions tab install Raspberry Pi Pico</br><br>
3.6 ) Accept everything it wants to install, EVERYTHINGG...</br><br>
3.7 ) From my small brain understanding, the picotool is installed with the VS Code extension, however it needs rules, download the rules file from [here](https://github.com/raspberrypi/picotool/tree/master/udev)</br><br>
3.8 ) copy the file to /etc/udev/rules.d/</br><br>
3.9 ) reload the udev rules: sudo udevadm control --reload</br><br>

### 4) Restart and hope that you can actually build a blink example and flash it to your device in bootsel mode (press the B button and plug it in to the USB) then hit run in the lower right corner of VS Code

### 5) Getting ready for the debugger:
5.1 ) Go here: https://github.com/raspberrypi/debugprobe/releases/tag/debugprobe-v2.3.1</br><br>
5.2 ) Download debugprobe_on_pico.uf2 and flash it on a pi pico using the bootsel mode and copying the uf2 file on the rpi "drive"</br><br>
5.3 ) In the /usr/local/share/openocd/scripts/interface/ you will find a file called CMSIS-DAP.cfg  open it in root mode (so that we can edit it)</br><br>
5.4 ) Finding out the iSerial for the CMSIS-DAP file:</br>
5.4.1 ) Go here: https://seagin.me/setting-up-rasberry-pi-debug-probe-linux/ and check out at the bottom.</br><br>

### 6) testing out if we can connect to the debug probe with openocd:
6.0 ) check out the wiring for the pi pico and the probe you just flashed from the getting started book i linked above in section A.2 page 17.</br><br>
6.1 ) go to the project folder where you previously built the blink example, in my case cd ~/Development/PiPico/blink/build    if you're unsure, look at the terminal output in VS Code, you should see the path where the elf and the uf2 files are created</br><br>
6.2 ) sudo openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "adapter speed 5000" -c "program blink.elf verify reset exit"</br><br>
6.3 ) now try the above without sudo and hope that all is fine and dandy :)</br><br>

### 7) This will be part of previous chapter, but i have to finger out how this actually works:
7.1 ) open two terminals, and in both of them navigate to the blink, or whatever program you build and have the elf and uf2 files there !!!!!!!!!!!!!</br><br>
7.2 ) on one of them connect to the pico board via openocd: sudo openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "adapter speed 5000"</br><br>
7.3 ) while the other terminal window has the open connection via openocd, here we will connect using gdb: gdb blink.elf</br><br>
7.4 ) in the opened gdb application then type (to connect to openocd) the three lines below:</br><br>
```
target extended-remote localhost:3333
monitor reset init
continue
```
## To-do:
1) make this connect via visual studio code and document the process</br>
1.1) Well it turns out that the VS Code SDK is configured in a way that the picotool, openocd and maybe gdb are actually contained within the sdk folder that it creates when installing the plugin to VSCode. It works, to nothing to tweak here, as of yet :)</br><br>

2) make a stand-alone project, without VS Code, using the sdk which is mostly libraries for the pico, the picotool, the openocd and the gdb, to see how this works under the hood.</br><br>

Some Info on what to study up:
- dude builds a debugger (in rust, but the concepts are valid for C): https://www.timdbg.com/posts/
- another dude made this thing using pi pico and C (yay): https://qcentlabs.com/posts/swd_banger/
- PI PICO Getting Started Guide: https://pip-assets.raspberrypi.com/categories/610-raspberry-pi-pico/documents/RP-008276-DS-2-getting-started-with-pico.pdf
- PI PICO C/C++ SDK Documentation (800+ pages): https://pip-assets.raspberrypi.com/categories/609-microcontroller-boards/documents/RP-009085-KB-4-raspberry-pi-pico-c-sdk.pdf





