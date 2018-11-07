.. _App-config:

AppManager
=======================

This class allows to manage the services used by an application. We can easily manage inputs/inouts/ouputs and the
connections between them. It connects/disconnects the signals and slots when the service is
started/stopped and when an object is added/removed.

Manage services
-----------------
To manage a service, you should create the service using ``addService(type, autoStart, autoUpdate)``. You can also add
an existing service using ``addService(srv, autoStart, autoUpdate)``. The two booleans *autoStart* and *autoUpdate*
allows to start (and update) the service when all its required objects are present.

Use ``startServices()`` to start all the services that have all their objects and it also define that when you will add
a new object, the services will be started.

Manage objects
---------------


In order to specified the required objects for your service, you can define it in the constructor as follows :
``registerObject(key, access, autoConnect, optional)``.

.. code-block:: cpp

    MyService::MyService()
    {
        this->registerObject("objectKey", ::fwServices::IService::AccessType::INOUT, true, false);
    }

Then, you can associate an object with its identifier used in the AppManager using ``setObjectId()``:

.. code-block:: cpp

    service->setObjectId("objectKey", "objectId");

    ::fwData::Image::sptr image = ::fwData::Image::New();
    appManager->addObject(image, "objectId");


You can also register an object that is not define in the constructor (or change the autoConnect/optional attributes)
using ``registerObject(objecId, objectKey, access, autoConnect, optional)``.

.. code-block:: cpp

    service->registerObject("objectId", "objectKey", ::fwServices::IService::AccessType::INOUT, false, false);

    ::fwData::Image::sptr image = ::fwData::Image::New();
    appManager->addObject(image, "objectId");


Example:
-----------

.. code-block:: cpp

    m_appMgr = ::boost::make_unique< ::fwServices::AppManager >();

    // Initialize the manager
    m_appMgr->create();
    // Create and register a service
    // - readerService will be automatically started and updated
    // - mesherService will be automatically started but not updated
    auto readerService = m_appMgr->addService("::uiIO::editor::SIOSelector", true, true);
    auto mesherService = m_appMgr->addService("::opVTKMesh::SVTKMesher", true, false);

    // configure the services ...

    // Register the objects to associate with the services:
    // - readerService will generate an output image, it is registered as "loadedImage" in the application
    // - mesherService require an input image registered as "loadedImage" in the application
    // - mesherService will generate an output model series, it is registered as "generatedModel" in the application
    readerService->setObjectId("imageSeries", "loadedImage");
    mesherService->setObjectId("imageSeries", "loadedImage");
    mesherService->setObjectId("modelSeries", "generatedModel");

    // Start the reader service:
    // - readerService will be started because it does not require input or inout. It will also be updated.
    // - mesherService will not be started because it requires an input image.
    m_appMgr->startServices();

    // When readerService will be updated, it will generate the image required by the mesher service. As the image is
    // registered with the same identifier in the application, the mesherService will be automatically started.


You can access the objects managed by the configuration using ``addObject(obj, id)``, ``getObject(id)`` and
``removeObject(obj, id)``.
