.. _tutoqml:

********************************************
[*TutoQml*] Create an application with Qml
********************************************

This page explain how to create an application using qml.

Launch the UI
===============

The application is declare in a bundle of type `APP` (see https://fw4spl.readthedocs.io/en/dev/HowTos/HowTo-BundleCreation.html).
The base of the application is located in the `Plugin` class.

To launch an qml interface, write your qml document in the *rc/* directory of your App. And then launch it from the
`Plugin::initialize()` method.

Example
------------

In our example, we will create an application named `TutoQml`. We will use the Qt example (https://doc.qt.io/qt-5.11/qmlfirststeps.html).
Create the file `ui.qml`:

.. code-block:: qml

    //import related modules
    import QtQuick 2.3
    import QtQuick.Controls 1.4
    import QtQuick.Window 2.2

    //window containing the application
    ApplicationWindow {

        //title of the application
        title: qsTr("Hello World")
        width: 640
        height: 480
        visible: true

        //menu containing two menu items
        menuBar: MenuBar {
            Menu {
                title: qsTr("&File")
                MenuItem {
                    text: qsTr("&Open")
                    onTriggered: console.log("Open action triggered");
                }
                MenuItem {
                    text: qsTr("&Quit")
                    onTriggered: Qt.quit();
                }
            }
        }

        //Content Area

        //a button in the middle of the content area
        Button {
            text: qsTr("Hello World")
            anchors.horizontalCenter: parent.horizontalCenter
            anchors.verticalCenter: parent.verticalCenter
            anchors.fill: parent
        }
    }


Then in the ``Plugin.cpp``, implement the `initialize()` method like:

.. code-block:: cpp
    void Plugin::initialize()
    {
        // get the QmlApplicationEngine
        SPTR(::fwQml::QmlEngine) engine = ::fwQml::QmlEngine::getDefault();

        // get the path of the qml file in the 'rc' directory
        auto path = ::fwRuntime::getBundleResourceFilePath("TutoQml", "ui.qml");

        // launch the qml component
        engine->loadMainComponent(path);
    }


Manage F4S servicesReg
============================

To manage your services in your application, you can use a `::fwServices::AppManager`. This class helps to create, start
and stop the services according to their associated the data. It also allows to manage the connections.

The easiest way is to add a class in your App that inherits from the AppManager and inherits from QObject to be
instantiated in Qml.

Example
-----------

Add the class `TutoQml::AppManager`.

**AppManager.hpp:**

.. code-block:: cpp

    #pragma once

    #include "TutoQml/config.hpp"

    #include <fwServices/AppManager.hpp>
    #include <fwServices/IService.hpp>

    #include <QObject>

    namespace TutoQml
    {

    /**
     * @brief   This class is started when the bundles is loaded.
     */
    class TUTOQML_CLASS_API AppManager : public QObject,
                                         public ::fwServices::AppManager
    {

    Q_OBJECT;
    public:
        /// Constructor.
        TUTOQML_API AppManager() noexcept;

        /// Destructor. Do nothing.
        TUTOQML_API ~AppManager() noexcept;

    public Q_SLOTS:

        // Initialize the manager
        void initialize();

        // Uninitialize the manager
        void uninitialize();

        // Open a file dialog to select the image to load
        void openImage();

        // Open a file dialog to select the file for the model to save, only if the model is meshed
        void saveModel();

        // Apply the mesher, only if the image is already loaded
        void applyMesher();

    private:

        ::fwServices::IService::sptr m_imageLoader;
        ::fwServices::IService::sptr m_mesher;
        ::fwServices::IService::sptr m_modelWriter;
    };

    } // namespace TutoQml


**AppManager.cpp:**

.. code-block:: cpp

    #include "TutoQml/AppManager.hpp"

    namespace TutoQml
    {

    static const std::string s_IMAGE_SERIES_ID = "imageSeries";
    static const std::string s_MODELSERIES_ID  = "modelSeries";

    //------------------------------------------------------------------------------

    AppManager::AppManager() noexcept
    {
    }

    //------------------------------------------------------------------------------

    AppManager::~AppManager() noexcept
    {
    }

    //------------------------------------------------------------------------------

    void AppManager::initialize()
    {
        this->create();

        // create the services
        m_imageLoader = this->addService("::uiIO::editor::SIOSelector", "", true);
        m_mesher      = this->addService("::opVTKMesh::SVTKMesher", "", true);
        m_modelWriter = this->addService("::uiIO::editor::SIOSelector", "", true);

        // associate the object to the services
        m_imageLoader->setObjectId("data", s_IMAGE_SERIES_ID);
        m_mesher->setObjectId("imageSeries", s_IMAGE_SERIES_ID);
        m_mesher->setObjectId("modelSeries", s_MODELSERIES_ID);
        m_modelWriter->setObjectId("data", s_MODELSERIES_ID);

        // configure the services
        ::fwServices::IService::ConfigType imageSeriesReaderConfig;
        imageSeriesReaderConfig.put("type.<xmlattr>.mode", "reader");
        imageSeriesReaderConfig.put("type.<xmlattr>.class", "::fwMedData::ImageSeries");
        m_imageLoader->configure(imageSeriesReaderConfig);

        ::fwServices::IService::ConfigType mesherConfig;
        mesherConfig.put("config.percentReduction", 50);
        m_mesher->configure(mesherConfig);

        ::fwServices::IService::ConfigType modelSeriesWriterConfig;
        modelSeriesWriterConfig.put("type.<xmlattr>.mode", "writer");
        m_modelWriter->configure(modelSeriesWriterConfig);

        // Start the services if all their data are present
        this->startServices();
    }

    //------------------------------------------------------------------------------

    void AppManager::uninitialize()
    {
        // stop the started services and unregister all the services
        this->stopAndUnregisterServices();
    }

    //------------------------------------------------------------------------------

    void AppManager::openImage()
    {
        m_imageLoader->update();
    }

    //------------------------------------------------------------------------------

    void AppManager::saveModel()
    {
        if (m_modelWriter->isStarted())
        {
            m_modelWriter->update();
        }
    }

    //------------------------------------------------------------------------------

    void AppManager::applyMesher()
    {
        if (m_mesher->isStarted())
        {
            m_mesher->update();
        }
    }

    //------------------------------------------------------------------------------

    } // namespace TutoQml


The AppManager must be registered as a qml type in order to instantiate it in qml. This is done in `Plugin::start()`:

.. code-block:: cpp

    void Plugin::start()
    {
        qmlRegisterType<AppManager>("TutoQml", 1, 0, "AppManager");
    }


We instantiate the AppManager in the qml ui and call the different slots in the qml.

.. code-block:: qml

    //import related modules
    import QtQuick 2.3
    import QtQuick.Controls 1.2
    import QtQuick.Window 2.2
    // Import TutoQml module
    import TutoQml 1.0

    //window containing the application
    ApplicationWindow {

        //title of the application
        title: qsTr("Hello World")
        width: 640
        height: 480
        visible: true

        // (un)initialize the app manager
        Component.onCompleted: appManager.initialize()
        onClosing: appManager.uninitialize();

        // Instantiate the AppManager
        AppManager {
            id: appManager

            // Now we can call its different slots from the qml objects
        }

        //menu containing three menu items
        menuBar: MenuBar {
            Menu {
                title: qsTr("&File")
                MenuItem {
                    text: qsTr("&Open image")
                    onTriggered: appManager.openImage() // call 'openImage' slot of the appManager
                }
                MenuItem {
                    text: qsTr("&Save model")
                    onTriggered: appManager.saveModel() // call 'saveModel' slot of the appManager
                }
                MenuItem {
                    text: qsTr("&Exit")
                    onTriggered: Qt.quit();
                }
            }
        }

        //Content Area

        //a button in the middle of the content area
        Button {
            text: qsTr("Apply mesher")
            anchors.horizontalCenter: parent.horizontalCenter
            anchors.verticalCenter: parent.verticalCenter
            anchors.fill: parent
            onClicked: appManager.applyMesher() // call 'applyMesher' slot of the appManager
        }
    }


VTK scene
============

Now, we will explain how to display our objects with a VTK scene (::fwRenderVTK::SRender) into a qml interface. We
render the scene into an off-screen frame buffer and then render it into a Qml widget. We use the
``::fwVTKQml::FrameBufferItem`` to render the scene.

Example
---------------

Add the `FrameBufferItem` in the qml interface:

.. code-block:: qml

    //import related modules
    import QtQuick 2.3
    import QtQuick.Controls 1.2
    import QtQuick.Layouts 1.0
    import QtQuick.Window 2.2
    // Import TutoQml module
    import TutoQml 1.0
    // Import fwVTKQml module to use the FrameBuffer
    import fwVTKQml 1.0

    //window containing the application
    ApplicationWindow {

        //title of the application
        title: qsTr("Hello World")
        width: 640
        height: 480
        visible: true

        // initialize the app manager
        Component.onCompleted: appManager.initialize()
        onClosing: appManager.uninitialize();

        // Instantiate the AppManager
        AppManager {
            id: appManager
            // @disable-check M16
            frameBuffer: scene3D // set the frameBuffer to the appManager in order to use it in the scene service
        }

        //menu containing two menu items
        menuBar: MenuBar {
            //...
        }

        //Content Area

        ColumnLayout {
            spacing: 0
            anchors.fill: parent

            Rectangle {
                id: rectangle
                color: "#000000"
                Layout.fillHeight: true
                Layout.fillWidth: true

                FrameBuffer {
                    id: scene3D
                    // @disable-check M16 (disable error on qtcreator to use the designer)
                    onReady: appManager.createVtkScene() // manage the vtk scene services
                    onWidthChanged: initialize()
                    onHeightChanged: initialize()
                }
            }

            //a button in the bottom of the content area
            Button {
                text: qsTr("Apply mesher")
                Layout.fillWidth: true
                onClicked: appManager.applyMesher()
            }
        }
    }


Then, we need to implement the slot `createVtkScene` in the AppManager to create the scene services and associate the
FrameBuffer.

**AppManager.hpp:**

Add a FrameBuffer property to set it in qml

.. code-block:: cpp

    Q_PROPERTY(FrameBufferItem* frameBuffer MEMBER m_frameBuffer)


**AppManager.cpp:**

.. code-block:: cpp

    void AppManager::createVtkScene()
    {
        if (!m_vtkSceneCreated)
        {
            // generic scene
            auto renderSrv = this->addService< ::fwRenderVTK::SRender >("::fwRenderVTK::SRender", "", true);
            m_imageAdaptor       = this->addService("::visuVTKAdaptor::SImageSeries", "", true);
            m_modelSeriesAdaptor = this->addService("::visuVTKAdaptor::SModelSeries", "", true);

            m_imageAdaptor->setObjectId("imageSeries", s_IMAGE_SERIES_ID);
            m_modelSeriesAdaptor->setObjectId("model", s_MODELSERIES_ID);

            // create and register the render service
            ::fwServices::IService::ConfigType renderConfig;
            ::fwServices::IService::ConfigType pickerConfig;
            pickerConfig.add("<xmlattr>.vtkclass", "fwVtkCellPicker");
            pickerConfig.add("<xmlattr>.id", "picker");
            renderConfig.add_child("scene.picker", pickerConfig);
            renderConfig.add("scene.renderer.<xmlattr>.id", "default");
            renderSrv->setConfiguration(renderConfig);
            renderSrv->useContainer(false);
            renderSrv->displayAdaptor(m_modelSeriesAdaptor->getID());
            renderSrv->displayAdaptor(m_imageAdaptor->getID());

            // set the interactor and the frame buffer
            auto interactorManager = ::fwRenderVTK::factory::New< ::fwVTKQml::VtkRenderWindowInteractorManager >();
            SLM_ASSERT("Frame Buffer is not yet defined", m_frameBuffer);
            interactorManager->setFrameBuffer(m_frameBuffer);
            renderSrv->setInteractorManager(interactorManager);
            renderSrv->configure();

            // configure the image adaptor
            ::fwServices::IService::ConfigType imageAdaptorConfig;
            imageAdaptorConfig.add("config.<xmlattr>.renderer", "default");
            imageAdaptorConfig.add("config.<xmlattr>.picker", "picker");
            imageAdaptorConfig.add("config.<xmlattr>.mode", "3d");
            imageAdaptorConfig.add("config.<xmlattr>.slice", "3");
            imageAdaptorConfig.add("config.<xmlattr>.sliceIndex", "axial");
            m_imageAdaptor->configure(imageAdaptorConfig);

            // configure the model adaptor
            ::fwServices::IService::ConfigType modelSeriesAdaptorConfig;
            modelSeriesAdaptorConfig.add("config.<xmlattr>.renderer", "default");
            modelSeriesAdaptorConfig.add("config.<xmlattr>.picker", "");
            m_modelSeriesAdaptor->configure(modelSeriesAdaptorConfig);

            // start the scene service
            this->startService(renderSrv);
            m_vtkSceneCreated = true;
        }
    }


Use editors in Qml
=====================

To make the connection between qml and our cpp data, we created the `::fwQml::IQmlEditor` service type. This class should
be inherited (like the `::fwGui::editor::IEditor` and be associated to a qml file.

This editor should be declared as qml type in the `Plugin::start()` of the bundle like:

.. code-block:: cpp

    void Plugin::start()
    {
        qmlRegisterType<MyEditor>("muyBundle", versionMajor, versionMinor, "MyEditor");
    }


To be used as a services, the AppManager must be notified that the service is created. We usually add a signal in the qml
file to notify the service creation like:

.. code-block:: qml

    // qml interface associated to the new editor
    Item {
        id: editorView
        enabled: false

        // signal to notify the service creation
        signal serviceCreated(var srv)

        Component.onCompleted: {
                // the signal is emitted when the qml component is created
                serviceCreated(myEditor)
        }

        MyEditor {
            id: myEditor

            // @disable-check M16
            onStarted: { // enabled the view when the editor is started
                editorView.enabled = true
            }
        }

        // ....
    }


Our editor will be instantiated, but it cannot be started because it is not registered by the AppManager and it requires
data. We disabled it by default and wait until the service is started to enabled it.

In our main qml file, we need to forward the signal to the AppManager.

.. code-block:: qml

    import myBundle 1.0

    // ...

    ApplicationWindow {

        // ...

        AppManager {
            id: appManager
            // ...
        }

        MyEditor {
            id: myEditor

            onServiceCreated: {
                // call onServiceCreated with the service instance and an identifier.
                // The identifier is only required if the same editor is used multipes times.
                appManager.onServiceCreated(srv, "myEditor1")
            }
            // ...
        }
    }


Wee need to be sure that the bundle's editors are registered before to use it, so we need to add the *requirement* in
the `plugin.xml`

.. code-block:: xml

    <plugin id="MyAppQml" class="::MyAppQml::Plugin"  version="@PROJECT_VERSION@" >

        <requirement id="servicesReg" />
        <!-- Add the qml bundle requirement. -->
        <requirement id="uiReconstructionQml" />

        <library name="TutoQml" />

    </plugin>


In the AppManager, we implement the slot `onServiceCreated(const QVariant& obj, const QString& id)`:

.. code-block:: cpp

    void AppManager::onServiceCreated(const QVariant& obj, const QString& id)
    {
        // check that the service is a IQmlEditor
        ::fwQml::IQmlEditor::sptr srv(obj.value< ::fwQml::IQmlEditor* >());
        if (srv)
        {
            // check if it is the desired editor
            if (srv->isA("::muyBundle::MyEditor") && id == "myEditor1")
            {
                // eventually associate the objects
                srv->setObjectId("obj", s_OBJ_ID);

                // register the new service in the AppManager
                this->addService(srv, true);
            }
            // ...
        }
    }


Example
------------

In our example, we will use the ``uiReconstructionQml`` bundle containing two qml files (``organMaterialEditorqml`` and
``representationEditor.qml``) in the *rc/* directory and the classes ``SOrganMaterialEditor`` and ``SRepresentationEditor``.

These two editors allows to change the color and the representation of a Reconstruction.

First, we add the two editors in our main qml file:

.. code-block:: qml

    import uiReconstructionQml 1.0

    // ...
    ApplicationWindow {
    // ...
        ColumnLayout {
            spacing: 0
            Layout.fillHeight: true
            Layout.preferredWidth: 80

            OrganMaterialEditor {
                id: organMaterialEditor
                Layout.fillWidth: true
                Layout.preferredHeight: 50

                onServiceCreated: {
                    appManager.onServiceCreated(srv, "organMaterialEditor")
                }
            }

            RepresentationEditor {
                id: representationEditor
                Layout.fillWidth: true
                Layout.fillHeight: true

                onServiceCreated: {
                    appManager.onServiceCreated(srv, "representationEditor")
                }
            }
        }
    }


Then, we implement the method ``onServiceCreated()`` in the AppManager to register the service and its required object.
This editor required a ``Reconstruction``, we will use the first ``Reconstruction`` from the generated ``ModelSeries``.

.. code-block:: cpp

    void AppManager::onServiceCreated(const QVariant& obj, const QString& id)
    {
        Q_UNUSED(id); // we don't use the id here because only one service of each type is used.

        // check that the service is a IQmlEditor
        ::fwQml::IQmlEditor::sptr srv(obj.value< ::fwQml::IQmlEditor* >());
        if (srv)
        {
            // check if it is the SOrganMaterialEditor
            if (srv->isA("::uiReconstructionQml::SOrganMaterialEditor"))
            {
                // register the new service in the AppManager, it will be automatically started when the reconstruction is
                // added
                this->setObjectId( "reconstruction", s_RECONSTRUCTION_ID);
                this->addService(srv, true);
            }
            // check if it is the SRepresentationEditor
            else if (srv->isA("::uiReconstructionQml::SRepresentationEditor"))
            {
                // register the new service in the AppManager, it will be automatically started when the reconstruction is
                // added
                this->setObjectId( "reconstruction", s_RECONSTRUCTION_ID);
                this->addService(srv, true);
            }
        }
    }


To register the ``Reconstruction``, we retrieve the ModelSeries when it is meshed and get the first Reconstruction to
add it in the AppManager.

.. code-block:: cpp

    void AppManager::applyMesher()
    {
        if (m_mesher->isStarted())
        {
            // wait until the mesher finished
            m_mesher->update().wait();

            // get the generated model series
            ::fwMedData::ModelSeries::sptr model = m_mesher->getOutput< ::fwMedData::ModelSeries >("modelSeries");

            // get the reconstruction and add it into the managed data
            if (model && model->getReconstructionDB().size() > 0)
            {
                ::fwData::Reconstruction::sptr rec = model->getReconstructionDB().front();
                this->addObject(rec, s_RECONSTRUCTION_ID);
            }
        }
    }
