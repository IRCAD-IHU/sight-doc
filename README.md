Description
===========

This repository contains the documentation of Sight. The main repository is available [here](https://git.ircad.fr/Sight/sight).

Building the documentation with Linux/MacOS
===========================================

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
First, you need to install [Python](https://www.python.org/downloads/)

- Add Python to your PATH
```
SET PATH=%PATH%;C:\Python
```
- Add Python Script
```
SET PATH=%PATH%;C:\Python\Scripts
```
- Use pip to install Sphinx
```
pip install sphinx
```
- Use pip to install Sphinx rtd-theme
```
pip install sphinx_rtd_theme
```

Once sphinx is installed, launch the following command at the root of your local copy to generate html documentation:
```
sphinx-build . _build\html
```