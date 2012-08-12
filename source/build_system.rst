The build System
================

Aery32 Framework comes with a powerful Makefile that provides a convenient way to compile the project. It also has targets for chip programming. At this point the Makefile attempts to use batchisp in Windows and dfu-programmer in Linux, so make sure you have those installed.

To compile the project just command::

    make

To clean the project folder from binaries call::

    make clean

and to recompile all the files::

    make re

The above command recompiles only the files from the project root. It does not recompile the Aery32 library, because that would be ridiculous. If you also want to recompile the Aery32 library use ``make reall``.

When you are ready to upload the program into the board type::

    make program

If you also want to start the program immediately type::

    make program start

or in shorter format::

    make programs

Additional targets are ``make debug`` that compiles the project with the debug statements. Use this when you need to do in-system debugging. There's also target for quality assurance check. That's ``make qa`` which compiles the project with more pedantic compiler options. It's good to use this every now and then. Particularly when there are problems in your program. This target can also tell you, if your inline functions are not inlined for some reason.

.. note::

	By default the project is compiled with -O2 optimization. If you run into troubles and your program behaves unpredictly on the chip, first try some other level of optimization

	.. code-block:: shell

		make reall COPT="-O0 -fdata-sections -ffunction-sections"

How to add new source files to the project
------------------------------------------

New sources files (.c, .cpp and .h) can be added straight into the project's root and the build system will take care of those. You can take it easy so far. However, if you like to separate your source code into folders, for example, into subdirectory called ``my/`` then you have to edit the Makefile slightly. After creating the directory, open the Makefile and find the line::

    SOURCES=$(wildcard *.cpp) $(wildcard *.c)

Edit this line so that it looks like this::

    SOURCES=$(wildcard *.cpp) $(wildcard *.c) $(wildcard my/*.c) $(wildcard my/*.cpp)