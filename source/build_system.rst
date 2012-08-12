The build System
================

Aery32 Framework comes with a powerful Makefile that provides a convenient way to compile the project and upload the compiled application into the board by using USB DFU Bootloader. To compile the project just command::

    make

or::
    
    make all

To clean the project folder from binaries call::

    make clean

and to recompile all the files::

    make re

When you are ready to upload the program into the board type::

    make program

If you also want to start the program immediately type::

    make program start

or in shorter format::

    make programs

How to add new source files to the project
------------------------------------------

New sources files (.c, .cpp and .h) can be added straight into the project's root and the build system will take those account. If you like to separate your source code into folders, for example, into ``my/`` subdirectory that is placed in the project's root, you have to edit the Makefile slightly. After creating the directory, open the Makefile and find the line::

    SOURCES=$(wildcard *.cpp) $(wildcard *.c)

Edit this line so that it looks like this::

    SOURCES=$(wildcard *.cpp) $(wildcard *.c) $(wildcard my/*.c) $(wildcard my/*.cpp)