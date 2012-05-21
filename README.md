[Aery32 Rerefence Guide](http://aery32.readthedocs.org/)
======================

The documentation is written in reStructuredText format to be constructed by Sphinx. Sphinx is a Python documentation generator but works fine for C as well. To build local html version of this documentation

    git clone git://github.com/aery32/aery32-refguide.git
    cd aery32-refguide
    make html

The following commands assume you have Sphinx installed -- if not, see the installation instructions below. 

The source files of this documentation are located in ``source/`` directory. From that folder you can find, for example, file ``index.rst``, which is the master document serving as a welcome page and "table of contents tree". To edit these source files just open the file in your favorite editor and be sure to edit in UTF-8 mode. To understand reSt syntax start from http://sphinx.pocoo.org/rest.html.

Installing Sphinx (in Windows 7)
--------------------------------

*Do you have Python installed?*

**- Yes, I have!**

Then you likely have setuptools installed as well. In this case install Sphinx with easy_install. Fire your command prompt (Win+R cmd) and command

    easy_install -U Sphinx

Otherwise follow steps below to install Python first and then Sphinx.

**- No, I do not have Python**

Note: We do not install setuptools here and thus do not use easy_install to install Sphinx. However you will get it installed along Sphinx installer and it is recommended to use it later when installing other Python packages.

- Create temporary directory where to download the following things:

  - Python 2.7.x from http://python.org/download/
  - Sphinx 1.1.2 from http://pypi.python.org/pypi/Sphinx

- When the both download processes have been completed, you should have these two files:

  - ``python-2.7.2.msi`` or ``python-2.7.2.amd64.msi`` if you downloaded 64-bit version
  - ``Sphinx-1.1.2.tar.gz``
    
- First install Python by double clicking Python installer
- After successful installation of Python, untar ``Sphinx-1.1.2.tar.gz`` into temporary directory
  
  - The exctarction process creates the ``Sphinx-1.1.2`` directory, change to that directory and double click setup to install Sphinx
  - Once the Sphinx installation is complete, you will find sphinx-xxx executables in your Python Scripts subdirectory, ``C:\Python27\Scripts``. Be sure to add this directory to your PATH environment variable. As you can see, this directory includes now also easy_install executable, which you should use later to install other Python packages.

- You can now remove the temporary download directory