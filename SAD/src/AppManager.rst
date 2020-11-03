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


You can also register an object that is not defined in the constructor (or change the autoConnect/optional attributes)
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

Launching multiple managers
------------------------------

If you want to dynamically launch an AppManager, you should inherit from this class. You can declare the required
inputs with strings. These strings will be replaced at the AppManager launch. You will need to call the
``getInputID("...")`` method to retrieve the replacing string.

The method "checkInputs" checks if all the required inputs are present and adds them to the manager.

You can find an example in the ExActivitiesQml sample.

.. code-block:: cpp

    static const std::string s_IMAGE_ID = "image";
    static const std::string s_MODEL_ID = "model";
    static const std::string s_VALIDATION_CHANNEL = "validationChannel";

    MyManager::MyManager() noexcept
    {
        this->requireInput(s_IMAGE_ID, InputType::OBJECT);
        this->requireInput(s_MODEL_ID, InputType::OBJECT);
        this->requireInput(s_VALIDATION_CHANNEL, InputType::CHANNEL);
    }

    MyManager::~MyManager() noexcept
    {
        this->destroy();
    }

    void MyManager::initialize()
    {
        this->create();

        if (this->checkInputs())
        {
            auto mesher = this->addService("::opVTKMesh::SVTKMesher", true, true);
            mesher->setObjectId("imageSeries", this->getInputID(s_IMAGE_ID));
            mesher->setObjectId("modelSeries", this->getInputID(s_MODEL_ID));

            ::fwServices::IService::ConfigType mesherConfig;
            mesherConfig.put("config.percentReduction", reduction);
            mesher->configure(mesherConfig);

            ::fwServices::helper::ProxyConnections connection(this->getInputID(s_VALIDATION_CHANNEL));
            connection.addSignalConnection(mesher->getID(), ::fwServices::IService::s_UPDATED_SIG);
            this->addProxyConnection(connection);

            this->startServices();
        }
        else
        {
            const std::string msg = "All the required inputs are not present, '" + this->getID() +
                                    "' will not be launched";
            ::fwGui::dialog::MessageDialog::showMessageDialog("Manager Initialization",
                                                              msg,
                                                              ::fwGui::dialog::IMessageDialog::CRITICAL);
        }
    }


The class launching the AppManager must replace the input keys by calling
``appManager->replaceInput("key", "value")``. These inputs can be provided by the Activity.
