.. _App-config:

App-config
=======================

Dynamic program with factories
------------------------------

As shown in the :ref:`Object-Service concept example<Object-Service_example>`, it is easy to change an application's
behavior by simply changing the appropriate data and services. For example changing an image visualization application
to a 3D model visualization application.
Unfortunately, this is limited to applications based on one service and one data,
and thus it would be impossible to apply to applications containing multiple services and objects.

To overcome this, the Sight architecture provides a dynamic management of configurations
to allow the use of multiple objects and services.

The xml configuration for an application is defined with the extension ``::fwServices::registry::AppConfig``.


Dynamic program with application configuration
----------------------------------------------

In the ``fwServices`` library, an application configuration parser
allows to parse XML files and creates and manages objects, services and
communications.

.. code-block:: c++

    // The parser
    void main (int argc , char * argv [])
    {
        string xmlAppConfigPath = argv [1];

        ::fwServices::AppConfigManager::sptr acm
                = ::fwServices::AppConfigManager::New();

        acm->setConfig(xmlAppConfigPath);
        acm->create(); // Creates objects and services from config.
        acm->start(); // Starts services specified in config.
        acm->update(); // Updates services specified in config.

        acm->stop(); // Stops services specified in config.
        acm->destroy(); // Destroy all services and then data.
    }


The following part corresponds to the configuration XML file of the previous :ref:`Object-Service_example`.

.. code-block:: xml

    <object uid="image" type ="::fwData::MyData" />

    <service uid="frame" type="DefaultFrame">
        <!-- service configuration -->
    </service>

    <service uid="view" type="MyCustomImageView">
        <in key="object" uid="image" />
        <!-- service configuration -->
    </service>

    <service uid="reader" type="MyCustomImageReader">
        <in key="object" uid="image" />
        <!-- service configuration -->
    </service>

    <!-- view listen now image modification -->
    <connect>
        <signal>image/objectModified</signal>
        <slot>view/receive</slot>
    </connect>

    <start uid="frame" />
    <start uid="view"/>
    <start uid="reader"/>

    <!-- Read the image on filesystem and notify
         the view to refresh is content -->
    <update uid ="reader"/>


This simple example shows how it is possible to build an application with several objects and services
using only a program and its configurations files.


Example
--------

.. code-block:: xml

    <extension implements="::fwServices::registry::AppConfig">
        <id>myAppConfigId</id>
        <parameters>
            <param name="appName" default="my Application" />
            <param name="appIconPath" />
        </parameters>
        <desc>Image Viewer</desc>
        <config>

            <object uid="myImage" type="::fwData::Image" />

            <!--
                Description service of the GUI:
                The ::gui::frame::SDefaultFrame service automatically positions the various
                containers in the application main window.
                Here, it declares a container for the 3D rendering service.
            -->
            <service uid="myFrame" type="::gui::frame::SDefaultFrame">
                <gui>
                    <frame>
                        <name>${appName}</name>
                        <icon>${appIconPath}</icon>
                        <minSize width="800" height="600" />
                    </frame>
                </gui>
                <registry>
                    <!-- Associate the container for the rendering service. -->
                    <view sid="myRendering" />
                </registry>
            </service>

            <!--
                Reading service:
                The <file> tag defines the path of the image to load. Here, it is a relative
                path from the repository in which you launch the application.
            -->
            <service uid="myReaderPathFile" type="::ioVTK::SImageReader">
               <inout key="target" uid="myImage" />
               <file>./TutoData/patient1.vtk</file>
            </service>

            <!--
                Visualization service of a 3D medical image:
                This service will render the 3D image.
            -->
            <service uid="myRendering" type="::vtkSimpleNegato::SRenderer">
               <in key="image" uid="myImage" />
            </service>

            <!--
                Definition of the starting order of the different services:
                The frame defines the 3D scene container, so it must be started first.
                The services will be stopped the reverse order compared to the starting one.
            -->
            <start uid="myFrame" />
            <start uid="myReaderPathFile" />
            <start uid="myRendering" />

            <!--
                Definition of the service to update:
                The reading service load the data on the update.
                The render update must be called after the reading of the image.
            -->
            <update uid="myReaderPathFile" />
            <update uid="myRendering" />

        </config>
    </extension>

Parameters
~~~~~~~~~~~

id
****
The id is the configuration identifier, and is thus unique to each configuration.

parameters (optional)
***********************
The parameters is a list of the parameters used by the configuration.

- param:
    defines the parameter

    - name:
        parameter name, used as ``${paramName}`` in the configuration. It will be replaced by the string
        defined by the service, activity or application that launches the configuration.

    - default (optional):
        default value for the parameter, it is used if the value is not given by the config launcher.

desc (optional)
****************
The description of the application.


Object
~~~~~~~~
the <object> tags define the objects of the AppConfig.

- uid (optional):
    Unique identifier of the object (::fwTools::fwID). If it is not defined, it will be automatically generated.
- type:
    Object type (ex: ``::fwData::Image``, ``::fwData::Composite``)
- src (optional, "new" by default)
     possible values: "new", "ref", "deferred"

     - **"new"** : defines that the object should be created
     - **"ref"** : defines that the object already exists in the application.
       The uid must be the same as the first declaration of this object (with "new").
     - **"deferred"** : defines that the object will be created later (by a service).

Specific object configuration
******************************

- matrix (optional):
    It works only for ``::fwData::TransformationMatrix3D`` objects. It defines the value of the matrix.

.. code-block:: xml

    <object uid="matrix" type="::fwData::TransformationMatrix3D">
        <matrix>
        <![CDATA[
            1  0  0  0
            0  1  0  0
            0  0  1  0
            0  0  0  1
        ]]>
        </matrix>
    </object>

- value (optional):
    Only these objects contain this tag : ``::fwData::Boolean``, ``::fwData::Integer``, ``::fwData::Float`` and ``::fwData::String``. It allows to define the value of the object.

.. code-block:: xml

    <object type="::fwData::Integer">
        <value>42</value>
    </object>

- colors (optional):
    Only ``::fwData::TransferFunction`` contains this tag. It allows to fill the transfer function values.

.. code-block:: xml

    <object type="::fwData::TransferFunction">
        <colors>
            <step color="#ff0000ff" value="1" />
            <step color="#ffff00ff" value="500" />
            <step color="#00ff00ff" value="1000" />
            <step color="#00ffffff" value="1500" />
            <step color="#0000ffff" value="2000" />
            <step color="#000000ff" value="4000" />
        </colors>
    </object>

- item (optional):
    It defines a sub-object of a composite or a field of any other object.

    - **key:** key of the object

    - **object:** the 'item' tag can only contain 'object' tags that represents the sub-object

.. code-block:: xml

    <item key="myImage">
        <object uid="myImageUid" type="::fwData::Image" />
    </item>

Service
~~~~~~~~~
The <service> tags represent a service working on the object(s).
Services list the data the use and how they access them.
Some services needs a specific configuration, it is usually described in the doxygen.

- uid (optional):
    Unique identifier of the service. If it is not defined, it will be automatically generated.
- impl:
    Service implementation type (ex: ``::ioVTK::SImageReader``)
- type (optional):
    Service type (ex: ``::io::IReader``)
- autoConnect (optional, "no" by default):
    Defines if the service receives the signals of the working object
- worker (optional):
    Allows to run the service in another worker (see :ref:`Multithreading`)

.. code-block:: xml

    <service uid="mesher" type="::opMesh::SMesher" worker="myWorker">
        <in key="image" uid="imageId" />
        <out key="mesh" uid="meshId" />
    </service>

- in:
    input object, it is const and cannot be modified
- inout:
    input object that can be modified
- out:
    output object, it must be created by a service and registered with the 'setOutput(key, obj)' method.
    The output object must be declared as "deferred" in the \<object\> declaration.

    - **key** : object key used to retrieve the object into the service
    - **uid** : unique identifier of the object declared in the <object> tag
    - **optional** : (optional, default "no", values: "yes" or "no") If "yes",
      the service can be started even if the object is not present.
      By definition, the output objects are always optional.


.. code-block:: cpp

    ::fwData::Image::csptr image = this->getInput< ::fwData::Image >("image");
    ::fwData::Mesh::sptr mesh = ::fwData::Mesh::New();
    // mesher .....
    this->setOutput("mesh", mesh);

Connection
~~~~~~~~~~~
- connect (optional):
     allows to connect one or more signal(s) to one or more slot(s). The signals and slots must be compatible.

    - channel (optional):
        name of the channel use for the connections.

.. code-block:: xml

    <connect channel="myChannel">
        <signal>object_uid/signal_name</signal>
        <slot>service_uid/slot_name</slot>
    </connect>


Start-up
~~~~~~~~~~
- start:
    defines the service to start when the AppConfig is launched. The services will be automatically stopped in the
    reverse order when the AppConfig is stopped.

.. code-block:: xml

    <start uid="service_uid" />

**The service using "deferred" object as input will be automatically started when the object is created.**


- update:
    defines the service to update when the AppConfig is launched.

.. code-block:: xml

    <update uid="service_uid" />
