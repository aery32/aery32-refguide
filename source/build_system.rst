The build system
================

Aery32 Framework comes with a powerful Makefile that provides a convenient way
to compile the project. It also has targets for chip programming.

To compile the project just command::

    make

When the compilation process is done, binaries will appear under the project's
root (``aery32.hex`` and ``aery32.elf``). These two files are used in chip
programming or program uploading, or chip flashing. Whatever you like to call
it. In addition to the binaries, assembly listing and file mapping files, have
been created. ``aery32.lst`` and ``aery32.map``, respectively.

The program size is also showed at the end of the compile, like this::

    Program size:
       text    data     bss     dec     hex filename
       3700    1340   64196   69236   10e74 aery32.elf
          0    5040       0    5040    13b0 aery32.hex
    SDRAM usage: 5512 bytes, 8.41064 %

Here the program size has been given in separate sections and the static
RAM usage has been calculated.

``.text`` correspond the FLASH usage and ``.data + .bss`` indicates the
RAM allocation. ``.bss`` section is the place where the initialized
data is copied at runtime during the startup.

Dynamic RAM usage of stack and heap sections cannot be calculated before hand
so make sure that there are always reasonable amount of RAM available for heap.
Default stack size is 4 kB and you just have to know if it's enough. You
can increase the stack size from linker script if needed.

.. note::

    By default the project is compiled with -O2 optimization. If you run into
    troubles and your program behaves unpredictly on the chip, first try some
    other level of optimization

    .. code-block:: none

        make reall COPT="-O0 -fdata-sections -ffunction-sections"

Chip programming
----------------

To program the chip with the compiled binary type::

    make program

At this point the Makefile attempts to use batchisp in Windows and
dfu-programmer in Linux, so make sure you have those installed. If you also
want to start the program immediately type::

    make program start

or in shorter format::

    make programs

If you want to clean the project folder from the binaries call::

    make clean

To recompile all the project files call::

    make re

The above command recompiles only the files from the project root. It does not recompile the Aery32 library, because that would be ridiculous. If you also want to recompile the Aery32 library use ``make reall``. There's also ``cleanall`` that cleans the Aery32 folder in addition to the project's root.

How to add new source files to the project
------------------------------------------

New sources files (.c, .cpp and .h) can be added straight into the project's root and the build system will take care of those. If you like to separate your source code into folders, for example, into subdirectory called ``my/`` then you have to edit the Makefile slightly. After creating the directory, open the Makefile and find the line::

    SOURCES=$(wildcard *.cpp) $(wildcard *.c)

Edit this line so that it looks like this::

    SOURCES=$(wildcard *.cpp) $(wildcard *.c) $(wildcard my/*.c) $(wildcard my/*.cpp)

Compile with debug statements
-----------------------------

There are two additional make targets that helps you in debugging, ``make debug`` and ``make qa``. The most important is::

    make debug

This compiles the project with the debug statements. Use this when you need to do in-system debugging.

For quality assurance check use::

    make qa

This target compiles the project with more pedantic compiler options. It's good to use this every now and then. Particularly when there are problems in your program. This target can also tell you, if your inline functions are not inlined for some reason.