Contributor's guide
===================

Contributing the code
----

The development of Aery32 Framework happens in GitHub, which is a web-based hosting service that use the Git revision control system. To contribute the code you have to have a GitHub account. After that you can `fork <http://help.github.com/fork-a-repo/>`_ Aery32 repository and start contribute by sending `pull request <http://help.github.com/send-pull-requests/>`_.

GitHub has a good beginners guide about `how to set up Git in Windows <http://help.github.com/win-set-up-git/>`_.

Sending a pull request (creating a patch)
''''

Fixes are sent as `pull requests <http://help.github.com/send-pull-requests/>`_. Before you start fixing the Aery32 library, create a new branch and code your patch in this new branch. If there is a reported issue you are fixing, name the branch according to the GitHub issue number, e.g. gh-02.

The benefit of this approach is that you can have plenty of fixes which are isolated from each others, especially from the master branch.

Coding standards:
''''

.. code-block:: c

    #include <stdbool.h>
    #include <avr32/io.h>

    typedef struct Foo Foo;
    typedef struct Bar Bar;

    struct Foo {
        Bar *bar;
    };

    struct Bar {
        Foo *foo;
    };

    /**
     * Initialize I/O pins
     *  
     * This is a docblock to describe the function. Do not use parameter
     * names in function prototypes.
     *
     * \param allinput If true all pins are init as inputs
     */  
    void init_io(bool);

    int
    main(void)
    {
        char *c;     /* char *c, not char* c */
        Foo foo;
        Bar bar;

        bar.foo = &foo;
        /* Do not use compound literals with structs in Aery32 library,
         * because avr32-g++ does not support those.
         *
         * bar = (Bar) {.foo = &foo}; // This is compound literal
         */

        init_io(false);
        for (;;) {      /* This is how infinite loops are written */
        }

        return 0;
    }

    void
    init_io(bool allinput)      /* Layout the functions like this */
    {
        int i;

        /* Use space between the syntactic keywords and opening parenthesis */
        for (i = 0; i < 2; i++) {
            AVR32_GPIO.port[i].gpers = 0xffffffff;

            /*
             * Excplicitly show the comparison with zero and always
             * use curly braces
             */
            if (allinput == 0) {
                AVR32_GPIO.port[i].oders = 0xffffffff;
            } else {
                AVR32_GPIO.port[i].oderc = 0xffffffff;
            }
        }
    }

Writing the documentation
----

The documentation is constructed by Sphinx. Sphinx is a Python documentation generator but works fine for C as well. To build html version of this documentation::

    cd aery32
    make refguide

The following commands assume you have Sphinx installed -- if not, see the installation instructions below. 

The source files of this documentation are located in ``docs/refguide/source/`` directory. From this folder you can find, for example, file ``index.rst``, which is the master document serving as a welcome page and "table of contents tree". To edit these source files just open the file in your favorite editor and be sure to edit in UTF-8 mode. To understand reSt syntax start from http://sphinx.pocoo.org/rest.html.

Installing Sphinx
''''

**In Windows**

*Case 1: I do have Python already installed*

If you do have Python installed already, then you likely have setuptools installed as well. In this case install Sphinx with easy_install. Fire your command prompt (Win+R cmd) and command::

    easy_install -U Sphinx

Otherwise follow steps below to install Python first and then Sphinx.

*Case 2: I don't have Python installed*

.. note::

    We do not install setuptools here and thus do not use easy_install to install Sphinx. However you will get it installed along Sphinx installer and it is recommended to use it later when installing other Python packages.

- Create temporary directory (e.g. myfoo) where to download the following things:

  - Python 2.7.x from http://python.org/download/
  - Sphinx 1.1.2 from http://pypi.python.org/pypi/Sphinx

- When the both download processes have been completed, you should have these two files:

  - ``python-2.7.2.msi`` or ``python-2.7.2.amd64.msi`` if you downloaded 64-bit version
  - ``Sphinx-1.1.2.tar.gz``
    
- First install Python by double clicking Python installer
- After successful installation of Python, untar ``Sphinx-1.1.2.tar.gz`` into temporary directory
  
  - The exctarction process creates the ``Sphinx-1.1.2`` directory, change to that directory and double click setup to install Sphinx
  - Once the Sphinx installation is complete, you will find sphinx-xxx executables in your Python Scripts subdirectory, ``C:\Python27\Scripts``. Be sure to add this directory to your PATH environment variable. As you can see, this directory includes now also easy_install executable, which you should use later to install other Python packages.

- You can now remove the temporary directory
