*******************************************************************
How to create a module, a lib, an executable or an application ?
*******************************************************************

In sight, the modules, libraries, applications and executables are folders containing:

- [required] two files to generate the *CMake* target: ``CMakeLists.txt``
  and ``Properties.cmake`` (see :ref:`HowCMake`).
- [optional] *include* and *src* folder to contain the header and source files.
- [optional] *rc* folder to contain resources and XML configuration files
- [optional] *test* folder to contain the unit tests

.. _moduleCreation:

How to create a module ?
==========================

In sight, you will encounter two types of modules:

- the modules containing only XML configurations
- the modules containing services or other cpp code

It is possible to contain at the same time configurations and services (or C++ code), but for the sake of clarity and
reusability we recommend to separate the two.

.. _configModule:

XML configurations modules
--------------------------

These modules does not contain C++ code, they only contain XML files and the required *CMake* files.
In the module folder, there is only the *CMake* files and the *rc* folder.

CMake files
~~~~~~~~~~~~

The CMakeLists.txt contains only ``fwLoadProperties()`` to load the Properties.cmake

The Properties.cmake defines the modules needed to launch the configuration (ie. the module of all the services present
in the configurations).

Example:

.. code-block:: cmake

    set( NAME dataManagerConfig )
    set( VERSION 0.1 )
    set( TYPE MODULE )
    set( DEPENDENCIES  ) # no dependency

    set( REQUIREMENTS # required module
        gui
        guiQt
        uiMedDataQt
        uiReconstructionQt
        ctrlSelection
        media
    )

Configurations
~~~~~~~~~~~~~~~

A module could contain several configurations, they are in the ``plugin.xml`` file in the *rc* folder.

.. code-block:: xml

    <plugin id="dataManagerConfig" version="@PROJECT_VERSION@" >

        <requirement id="dataReg" />
        <requirement id="servicesReg" />

        <!-- ... extensions ... -->

    </plugin>

The ``@PROJECT_VERSION@`` will be automatically replaced by the version defined in the Properties.cmake.

The ``<requirement>`` tags contain the modules that must be started before to start your module (see https://sight.pages.ircad.fr/sight/group__requirement.html).

Then the extensions are defined. There are different types of extensions, the most common are:

-  ``::fwServices::registry::AppConfig`` to define configurations for applications (see :ref:`tuto01`)
-  ``::fwActivities::registry::Activities`` to define activities
-  ``:fwServices::registry::ServiceConfig`` to define configurations of services (mostly used to configure readers/writers)
- ``::fwServices::registry::ServiceFactory`` to define services

.. TODO add links to documentation for the extensions

.. note::

    To separate the configuration in several files, you can use ``<xi:include href="..." />``

.. _serviceModule:

Service modules
----------------

You don't need to create the ``plugin.xml`` file for the module that contains only services,
it will be automatically generated.
A ``CMake`` script parses the services macro and doxygen
to generate the ``::fwServices::registry::ServiceFactory`` extension
(see :ref:`serviceCreation` and :ref:`serviceNotFound`)

The module contains the service header files in the `include` folder and the `source` files in the `src` folder.
It must also contain a ``Plugin`` class used to register the module.

The ``Plugin.hpp`` in the *include* folder should look like:

.. code-block:: cpp

    #pragma once

    #include <fwRuntime/Plugin.hpp>

    namespace myModule
    {

    class MYMODULE_CLASS_API Plugin : public ::fwRuntime::Plugin
    {

    public:

        /// Plugin destructor
        ~Plugin() noexcept;

        /// This method is used by runtime to start the module.
        void start();

        /// This method is used by runtime to stop the module.
        void stop() noexcept;

        /// This method is used by runtime to initialize the module.
        void initialize();

        /// This method is used by runtime to uninitialize the module.
        void uninitialize() noexcept;

    };

    } // namespace myModule


The ``Plugin.cpp`` in the *src* folder should be like:

.. code-block:: cpp

    #include <fwRuntime/utils/GenericExecutableFactoryRegistrar.hpp>

    #include "myModule/Plugin.hpp"

    namespace myModule
    {

    //-----------------------------------------------------------------------------

    static ::fwRuntime::utils::GenericExecutableFactoryRegistrar<Plugin> registrar("::myModule::Plugin");

    //-----------------------------------------------------------------------------

    Plugin::~Plugin() noexcept
    {
    }

    //-----------------------------------------------------------------------------

    void Plugin::start()
    {
    }

    //-----------------------------------------------------------------------------

    void Plugin::stop() noexcept
    {
    }

    //-----------------------------------------------------------------------------

    void Plugin::initialize()
    {
    }

    //-----------------------------------------------------------------------------

    void Plugin::uninitialize() noexcept
    {
    }

    //-----------------------------------------------------------------------------

    } // namespace myModule


.. warning::

    The ``registrar("::myModule::Plugin");`` is the most important line, it allows to register the module to be used in a XML based application.

    **Don't forget to register the module with the correct namespace with '::'.**

The methods ``start()`` and ``stop`` must be implemented but are usually empty. They are called when the application is
started and stopped. The ``initialize()`` method is executed after the *start* of all the modules and ``uninitialize()`` before the *stop*.
