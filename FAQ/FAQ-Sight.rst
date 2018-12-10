What is sight?
===============

The framework Sight (Surgical Image Guidance and Health-care Toolkit) is an open-source
framework, developed by IRCAD (research institute against cancer and disease).
The purpose of Sight is to ease the creation of applications in the medical imaging field.
Therefore it provides features like digital image
processing in 2D and 3D, visualization or simulation of medical interactions.

Building an application with Sight only requires to write one or several XML files.
Its functionalities can be also extended by writing new components in C++,
which is the coding language of the framework.

What does sight mean?
======================

Sight means Surgical Image Guidance and Health-care Toolkit.

What are the features of sight?
=======================================

Sight is based on a component architecture composed of C++ libraries.
The three main concepts of the architecture, explained in the following sections, are:

-  object-service concept
-  component approach
-  signal-slot communication

The framework is multi-platform and runs under Windows, Linux, and MacOS.
Building an application with Sight only requires to write one or several XML files.
Its functionalities can be also extended by writing new components in C++,
which is the coding language of the framework.


Which platforms does sight run on?
===================================

This framework can run under Windows, Linux and MacOS.

Where can I find applications developed with sight ?
======================================================

Some tutorials are provided with the framework and you can also build VRRender,
a free visualization software or ARCalibration, a user-friendly camera calibration tool.

Which prerequisites do I need to develop new services and bundles ?
=====================================================================

You must have a good knowledge in C++. Concerning the configuration files, they are written in XML.

What are the BinPkgs?
======================

The BinPkgs (binary packages) contain all the extern libraries needed by sight.
For each BinPkg, a CMakeLists provides the OS specific instructions to build it.
They can be downloaded on https://git.ircad.fr/sight/sight-deps.

Is it difficult to compile an application with sight?
======================================================

No, it isn't. You just have to compile all the bundles and libraries used by the application.
Please follow the :ref:`installation instructions<Installation>` for your platform.

Why does sight provide a launcher?
===================================

The launcher is used to create the entry point of the application.
It parses the profile and configuration xml file to build it.

How can I debug my program ?
=============================

First, you can watch the log of the application. Under Windows platform,
log messages are saved on filesystem in **SLM.log** file, in the working directory.
On other systems, they are displayed in the terminal.

You can increase or decrease the log level of a sub-project in the CMake configuration,
by setting the **advanced** variable **SPYLOG_LEVEL_<PROJECT>**
(thus you to press 't' in *ccmake* or click of the 'advanced' checkbox in *cmake-gui* to show it).

The allowed values are : ['trace', 'debug', 'info', 'error', 'fatal', 'warning', 'disable'].
The value 'trace' gives the maximum of log, whereas 'disable' disables log.
The default value is 'error', which is enough in most cases.
'warning' can be useful in some cases.
Lower levels are really designed to be activated punctually when debugging a specific piece of code.

.. note::
    Printing many log messages ( by activating trace on all sub-projects for ex. ) can be very time consuming for the application.


Secondly, you can of course compile your application in Debug mode (set **CMAKE_BUILD_TYPE** to "Debug" )
and then debug it using **gdb** (Linux/Mac), **QtCreator** (Linux/Mac), **Visual Studio** (Windows).

Thirdly, you can manage the program complexity by reducing the number of activated components (in profile.xml)
and the number of created services (in config.xml) to better localize errors.

Fourthly, verify that your profile.xml / plugin.xml and each bundle plugin.xml are well-formed,
by using xmllint (command line tool provided by libxml2).

I have an assertion/fatal message when I launch my program, any idea to correct the problem ?
===================================================================================================

First, you can read the output message :) and try to solve the problem.
In many cases, there are two kind of problems. The program fails to :

- create the service given in configuration. In this case, four reasons are possibles :
    - the name of service implementation in *config.xml* contains mistakes
    - the bundle that contains this service is not activated in the profile
    - the bundle plugin.xml, that contains this service,
      does not declare the service or the declaration contains mistakes.
    - the service is not registered in the Service Factory (forget of macro *fwServicesRegisterMacro(...)* in file .cpp)
- manage the configuration of service. In this case, the implementation code
  in .cpp file ( generally configuring() method of service ) does not correspond
  to description code in config.xml ( Missing arguments, or not well-formed, or mistakes string parameters ).

Do I need to convert my data object to a ::fwData::Object ?
==================================================================================================

Do you need to share this data between services ?

    - If the answer is no, then you don't need to wrap your data.
    - Otherwise, you need to have an object that inherits of ::fwData::Object.

In this latter case, do you need to share this object between different services which use different third-party libraries, i.e. for ::fwData::Image, itk::Image vs vtkImage ?

    - If the answer is yes, then you need create a new object like fwData::Image and a wrapping with fwData::Image<=>itk::Image and fwData::Image<=>vtkImage.
    - Otherwise, you can just encapsulated an itk::Image in fwData::Image and create an accessor on it. ( however, this choice implies that all applications that use fwData::Image need ITK library for running. )

.. _campPath:

What is a camp path ?
======================

A **camp path** (also called sesh@ path) is a path  used to browse an object (and sub-object)
using the introspection (see fwDataCamp and :ref:`Serialization`).
The path begins with a '@' or a '!'.

- ``@`` : the returned string is the fwID of the sub-object defined by the path.
- ``!`` : the returned string is the value of the sub-object,
  it works only on String, Integer, Float and  Boolean object.

Sadly, we do not have yet a document giving the paths for all existing data.
To know how an object can be accessed with a sesh@ path, you can
have a look at the corresponding fwDataCamp implementation of the object.
For instance, the file *fwDataCamp/Image.cpp* shows :

.. code:: c++

    fwCampImplementDataMacro((fwData)(Image))
    {
        builder
        .tag("object_version", "2")
        .tag("lib_name", "fwData")
        .base< ::fwData::Object>()
        .property("size", &::fwData::Image::m_size)
        .property("type", &::fwData::Image::m_type)
        .property("spacing", &::fwData::Image::m_spacing)
        .property("origin", &::fwData::Image::m_origin)
        .property("array", &::fwData::Image::m_dataArray)
        .property("nb_components", &::fwData::Image::m_numberOfComponents)
        .property("window_center", &::fwData::Image::m_windowCenter)
        .property("window_width", &::fwData::Image::m_windowWidth)
        ;
    }

Which means that each property is a reachable by a **camp path**.
This is notably used by services in the ``ctrlCamp`` bundle, like ``SExtractObjObj`` or ``SCopy``.
For instance the height of the image can be retrieved using:

.. code:: xml

     @size.1

Other examples:
----------------

To get the image contained in a ``::fwData::Composite`` with the key ``myImage``

.. code:: xml

     @values.myImage

To get the first reconstruction of a ModelSeries contained in a ``::fwData::Composite`` with the key ``myModel``

.. code:: xml

     @values.myModel.reconstruction_db.0
