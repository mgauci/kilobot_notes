Using Eclipse and WinAVR to Program Kilobots
============================================
### Author: Melvin Gauci
melvin.gauci@gmail.com

This document describes how to set up the Eclipse IDE to program Kilobots using the WinAVR toolchain on Windows.

**NOTE:** If you're new to the Kilobot platform, you're highly recommended to you use the [online Kilobot editor](https://www.kilobotics.com/editor), instead of going through the process below. This process is only worth the trouble for people who'll be working with the Kilobot platform extensively and would like to use a more comprehensive IDE.

1. Installing WinAVR
--------------------

1.1. [Download](http://winavr.sourceforge.net/) and install WinAVR. During installation, accept the option to install Programmer’s Notepad.

1.2. If you’re running Windows 8.1 (or Vista, apparently!) you’ll need to replace a DLL in the WinAVR installation directory (a bug that's described [here](http://www.avrfreaks.net/forum/windows-81-compilation-error)). [Download](http://www.madwizard.org/download/electronics/msys-1.0-vista64.zip) and extract this DLL, and copy it to `/utils/bin` under your WinAVR installation directory. When prompted if you want to replace the current DLL, accept.

2. Building Kilolib
-------------------

2.1. [Download](https://github.com/acornejo/kilolib) Kilolib and extract the zipped file.

> You now need to build Kilolib, and it’s easiest to do this using the Programmer’s Notepad IDE. There's no need to create a project; you just need to point the IDE to the Kilolib directory, and it will execute the Makefile that's in there.

2.2. In Programmer’s Notepad, click `File -> Open`, navigate to the Kilolib directory and open the file `blank.c`. 

2.3. Click `Tools > Make All`.

> If all goes well, you should get: `Process Exit Code: 0`. The built files are now found under `/build` in your Kilolib directory. The static library is the file `kilolib.a`.

3. Installing Eclipse and Plugins
---------------------------------
> First, you need to have Eclipse installed with C/C++ support (the CDT plugin).

3.1. If you don’t plan to use Eclipse for Java development, you can directly download and install the [Eclipse IDE for C/C++ Developers](http://eclipse.org/downloads/packages/eclipse-ide-cc-developers/lunasr2) version. In this case, skip the next step.

3.2. Alternatively, if you already have Eclipse installed, but not the CDT plugin, install it by following the instructions [here](http://eclipse.org/cdt/downloads.php).

> Next, you need to install the AVR Eclipse plugin. Note that [this does NOT include an AVR toolchain](http://avr-eclipse.sourceforge.net/wiki/index.php/The_AVR_GCC_Toolchain) (compiler, linker, etc.), which is why you installed WinAVR.

3.3. Install the AVR Eclipse plugin by following the instructions [here](http://avr-eclipse.sourceforge.net/wiki/index.php/Plugin_Download).
 
4. Setting Up an Eclipse Project
--------------------------------
> Now for the slightly gruelling part. You'll set up an Eclipse project from scratch with the correct settings to build Kilobot programs. Once you’ve finished, you should probably (read: certainly) make a copy of this ‘clean’ project, so you don’t have to repeat this procedure every time you want to start a new project.

### Creating the Project

4.1. Open Eclipse and select a workspace (e.g. create a directory on your desktop for handiness).

4.2. Click `File > New > C Project`. In the dialog, give your project a name (here we'll assume it's `Kilobot`) and under `Project Type`, select `AVR Cross Target Application > Empty Project`. Click `Next`.

4.3. You probably don’t need both Debug and Release configurations. Uncheck `Debug` and click `Next`.

4.4. In the next dialog (`AVR Target Hardware Properties`), under `MCU Type` select `ATmega328P`, and set the `MCU Frequency` to `8000000` Hz (8 MHz). Click `Finish`.

4.5. Create a new source file in your project by right clicking on the project's name `> New > Source File`. Give this new source file a name, e.g. `main.c`. Copy and paste into this file the code from `blank.c` found in the Kilolib directory. You’ll use this code to make sure everything is working (later, you’ll write your own programs in this file).

### Adding Header and Library Files

> You now need to provide Eclipse with paths to the Kilolib header (.h) and library (.a) files. Although you could set these paths to any directory on your machine, I recommend copying them into your Eclipse project directory. They're small enough, and this way your project is self-contained.

4.6. In your Eclipse project directory, create two new directories, and call them `include` and `lib`. (to create a new directory, right click on the project name `> New > Folder`). 

4.7. Copy *all* the .h files from the Kilolib directory into the `include` directory. 

4.8. Copy the kilolib.a file from /build in the Kilolib directory into the new `lib` directory. 

4.9. *Very importantly*, rename the kilolib.a file you just copied into the `lib` directory to `libkilolib.a` (i.e. prepend it with "lib"). This is because the linker expects all library files to start with "lib".

We’ll now tell Eclipse where to look for the include and library files.

7. Right click on your project, and go to Properties. Go to C/C++ Build -> Settings. Under “Tool Settings” (first tab on the left, should be activated by default) go to “Directories” under AVR Compiler. Under “Include Paths (-I)” add a new path, and paste this:
"${workspace_loc:/${ProjName}/include}"
This is a relative path, telling the compiler to look for the /include directory in your project directory, which in turn is in the workspace directory. Alternatively instead of pasting this, you can click the “Workspace” button in the dialog that pops up when you add a new path, and navigate to the /include directory in your project.

8. Now go to “Libraries” under AVR C Linker. Under “Libraries Path (-L)” add a new path and paste: 
"${workspace_loc:/${ProjName}/lib}"
(or navigate to /lib by clicking “Workspace” as described above).

9. Under “Libraries (-l)” add a library and type: “kilolib”. The linker will automatically look for a file called “libkilolib.a”, which is why we renamed this before.

### Fixing a Bug

One last step: If you try compiling now and you use any functions from the math library (math.h), you’ll get an error. This is due to a bug - apparently Eclipse attempts to use the C++ instead of the C library. This is described here: http://forum.arduino.cc/index.php?topic=40215.0

10. The first solution posted there works. It’s copied here for convenience:

Try modifying the linker "Command line pattern" in (C/C++ Build -> Settings -> AVR C++ Linker) to look like this:

${COMMAND}  -lc -lm ${FLAGS} ${OUTPUT_FLAG}${OUTPUT_PREFIX}${OUTPUT}  ${INPUTS}  -lc

This should convince the linker to get math functions from libc and not libgcc. Apparently there is some bad code in the libgcc math functions that assumes all inter-library jumps are reachable with an RJMP when in fact the linker can put functions anywhere in .text that it feels fit.

The above workaround works for me.

5. Building Your Code and Uploading it to a Kilobot Robot
---------------------------------------------------------

11. Close the settings. You’re now good to go. Build the project by pressing Ctrl++B (or going to Project -> Build All). A subdirectory “Release” will be created in your project, and the object file will be found here (“Kilobot.hex” if you named your project “Kilobot”).

12. You can now upload this hex file to your Kilobot robot using the KiloGUI app.

6. Closing Notes
----------------

If you want you can now delete the Kilolib folder from your PC as we have copied the necessary header + library files into the Eclipse project directory.

If you really want to you can also unisntall the Programmer’s Notepad but you cannot uninstall the WInAVR toolchain. This is the toolchain that Eclipse is using under the hood.

Renaming a project.
