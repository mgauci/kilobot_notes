Using Eclipse and WinAVR to Program Kilobots
============================================
Author: Melvin Gauci
Created: 05 May 2015
Last modified: 05 May 2015

We’ll set ourselves up to use Eclipse as our IDE, which will build our code using the WinAVR toolchain.
Installing the WinAVR Toolchain
1. Download and install WinAVR from http://winavr.sourceforge.net/. Make sure you accept the option to also install Programmer’s Notepad during installation.

2. If you’re running Windows 8.1 (or Vista, apparently!) you’ll need to replace a DLL or things won’t build. This bug is described here: http://www.avrfreaks.net/forum/windows-81-compilation-error

“I found solution.
Copy this file:
http://www.madwizard.org/download/electronics/msys-1.0-vista64.zip
to utils\bin directory (WinAVR)”

The file is also available in the same directory as this document, for convenience.
Compiling Kilolib
1. Download and extract Kilolib from Alex Cornejo’s Github repo: https://github.com/acornejo/kilolib (also available as a zipped folder in the same directory as this document).

We now need to build Kilolib. It’s easiest to do this from Programmer’s Notepad IDE. We don’t need to create a project; we just need to point it to the Kilolib root directory, and it will execute the Makefile supplied with the library using the WinAVR toolchain.

2. Open Programmer’s Notepad, File -> Open, navigate to the Kilolib directory and open the file “blank.c”. Click on Tools > Make All.

3. If all goes well, you should get: “Process Exit Code: 0”. The build files are now found under /build in your Kilolib directory. The library itself is the file kilolib.a.

Installing Eclipse & Plugins
We now need to set up an Eclipse project to compile with WinAVR and link to the Kilolib library.

First, you need to have Eclipse with C/C++ support installed. Eclipse can be downloaded from here: http://eclipse.org/downloads/

1a. If you don’t plan to use Eclipse for Java development, you can directly download the C/C++ version (Eclipse IDE for C/C++ Developers) and you can skip the next step of installing the CDT plugin.

1b. Alternatively, if you already have Eclipse installed, but not the CDT plugin, install it by following the instructions here: http://eclipse.org/cdt/downloads.php

Next, we need to install the AVR Eclipse plugin. Note that this does NOT include an AVR toolchain (compiler, linker, etc.), which is why we installed the WinAVR toolchain before (see here: http://avr-eclipse.sourceforge.net/wiki/index.php/The_AVR_GCC_Toolchain).

2. Install the AVR Eclipse plugin by following the instructions here: http://avr-eclipse.sourceforge.net/wiki/index.php/Plugin_Download
Setting Up an Eclipse Project
We’ll now set up an Eclipse project from scratch to compile code for the Kilobot. Once you’ve finished, you probably want to make a copy of this ‘clean’ project and keep it in a safe place, so you don’t have to repeat this procedure every time you want to start a new project.

1. Open Eclipse, and select a workspace (e.g. create a folder on your Desktop for handiness).

2. Click File > New > C Project. In the dialog, give your project a name (e.g. Kilobot) and under Project Type, select AVR Cross Target Application > Empty Project. Click Next.

3. You probably don’t need both Debug and Release configurations. Uncheck Debug and click Next.

4. In the next dialog (AVR Target Hardware Properties), select ATmega328P as the MCU Type and set the MCU Frequency to 8000000 Hz (8 MHz). Click Finish.

5. Create a new main file for your project by right clicking on its name, New > Source File, and give it a name, e.g. main.c. Copy and paste into this file the code from blank.c found in the Kilolib folder. We’ll use this code to make sure everything is working (later, you’ll write your own programs in this file).

We now need to provide with directories for the include files (.h) and library file (.a) of the Kilolib library. Although these could be anywhere on your machine, I recommend making a copy of them in your Eclipse project directory. They are small enough, and this way, your project is self-contained.

6. In your Eclipse project, create two new directories, include and lib (right click on the Project Name > New > Folder). Copy ALL the .h files from the Kilolib directory into the new “include” directory. Copy the kilolib.a file from /build in the Kilolib directory into the new lib directory. Very importantly, rename the kilolib.a file you just copied into /lib in your Eclipse project directory to libkilolib.a (i.e. prepend it with “lib”). This is because the linker expects all library files to start with lib.

We’ll now tell Eclipse where to look for the include and library files.

7. Right click on your project, and go to Properties. Go to C/C++ Build -> Settings. Under “Tool Settings” (first tab on the left, should be activated by default) go to “Directories” under AVR Compiler. Under “Include Paths (-I)” add a new path, and paste this:
"${workspace_loc:/${ProjName}/include}"
This is a relative path, telling the compiler to look for the /include directory in your project directory, which in turn is in the workspace directory. Alternatively instead of pasting this, you can click the “Workspace” button in the dialog that pops up when you add a new path, and navigate to the /include directory in your project.

8. Now go to “Libraries” under AVR C Linker. Under “Libraries Path (-L)” add a new path and paste: 
"${workspace_loc:/${ProjName}/lib}"
(or navigate to /lib by clicking “Workspace” as described above).

9. Under “Libraries (-l)” add a library and type: “kilolib”. The linker will automatically look for a file called “libkilolib.a”, which is why we renamed this before.

One last step: If you try compiling now and you use any functions from the math library (math.h), you’ll get an error. This is due to a bug - apparently Eclipse attempts to use the C++ instead of the C library. This is described here: http://forum.arduino.cc/index.php?topic=40215.0

10. The first solution posted there works. It’s copied here for convenience:

Try modifying the linker "Command line pattern" in (C/C++ Build -> Settings -> AVR C++ Linker) to look like this:

${COMMAND}  -lc -lm ${FLAGS} ${OUTPUT_FLAG}${OUTPUT_PREFIX}${OUTPUT}  ${INPUTS}  -lc

This should convince the linker to get math functions from libc and not libgcc. Apparently there is some bad code in the libgcc math functions that assumes all inter-library jumps are reachable with an RJMP when in fact the linker can put functions anywhere in .text that it feels fit.

The above workaround works for me.

11. Close the settings. You’re now good to go. Build the project by pressing Ctrl++B (or going to Project -> Build All). A subdirectory “Release” will be created in your project, and the object file will be found here (“Kilobot.hex” if you named your project “Kilobot”).

12. You can now upload this hex file to your Kilobot robot using the KiloGUI app.
Closing Notes
If you want you can now delete the Kilolib folder from your PC as we have copied the necessary header + library files into the Eclipse project directory.

If you really want to you can also unisntall the Programmer’s Notepad but you cannot uninstall the WInAVR toolchain. This is the toolchain that Eclipse is using under the hood.
