Description
===========

This repository contains the documentation of FW4SPL. The main repository is available [here](https://github.com/fw4spl-org/fw4spl).

Building the documentation with Linux
=====================================

In order to build this documentation, you will need to install Sphinx (especially the sphinx-build command).
The documentation for installation is available [here](http://www.sphinx-doc.org/en/stable/install.html).

Once sphinx is installed, launch the following command at the root of your local copy to generate html documentation:
```
make html
```

Other generation backends can be listed with the `make` command.

Building the documentation with Windows
=======================================

In order to build this documentation, you will need to install Sphinx (especially the sphinx-build command).
First, you need to install `Python <https://www.python.org/downloads/>`_

	- Add Python to your PATH 		: SET PATH=%PATH%;C:\Python27
	- Add Python Script 			: SET PATH=%PATH%;C:\Python27\Scripts
	- Use pip to install Sphinx 	: pip install sphinx

Once sphinx is installed, launch the following command at the root of your local copy to generate html documentation:
```
sphinx-build . _build\html
```