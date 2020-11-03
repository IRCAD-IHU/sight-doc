.. _TutorialsTuto04SignalSlot:

***********************************************
[*Tuto04SignalSlot*] Signal-slot communication
***********************************************

The fourth tutorial explains the communication mechanism with signals and slots.

.. figure:: ../media/tuto04SignalSlot.png
    :scale: 25
    :align: center

=============
Prerequisites
=============

Before reading this tutorial, you should have seen :
 * :ref:`TutorialsTuto03DataService`
 * :ref:`SADSigSlot`

=========
Structure
=========

------------------
Properties.cmake
------------------

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto04SignalSlot )
    set( VERSION 0.2 )
    set( TYPE APP )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        fwlauncher              # Needed to build the launcher
        appXml                  # XML configurations

        guiQt                   # Start the module, load qt implementation of gui

        # Objects declaration
        fwData
        servicesReg             # fwService

        # UI declaration/Actions
        gui
        style

        # Reader
        ioVTK

        # Services
        uiIO
        visuBasic
    )

    moduleParam(guiQt
        PARAM_LIST
            resource
            stylesheet
        PARAM_VALUES
            style-0.1/flatdark.rcc
            style-0.1/flatdark.qss
    ) # Allow dark theme via guiQt

    moduleParam(
            appXml
        PARAM_LIST
            config
        PARAM_VALUES
            Tuto04SignalSlot_AppCfg
    ) # Main application's configuration to launch

.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``: the file used to launch the application.

----------
plugin.xml
----------

This file is in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <plugin id="Tuto04SignalSlot" version="@PROJECT_VERSION@" >

        <requirement id="servicesReg" />
        <requirement id="guiQt" />

        <extension implements="::fwServices::registry::AppConfig" >
            <id>Tuto04SignalSlot_AppCfg</id>
            <config>

                <!-- ******************************* Objects declaration ****************************** -->

                <!-- The main data object is ::fwData::Mesh. -->
                <object uid="mesh" type="::fwData::Mesh" />

                <!-- ******************************* UI declaration *********************************** -->

                <service uid="mainFrame" type="::gui::frame::SDefaultFrame" >
                    <gui>
                        <frame>
                            <name>Tuto04SignalSlot</name>
                            <icon>Tuto04SignalSlot-@PROJECT_VERSION@/tuto.ico</icon>
                            <minSize width="1280" height="720" />
                        </frame>
                        <menuBar />
                    </gui>
                    <registry>
                        <menuBar sid="menuBarView" start="yes" />
                        <view sid="containerView" start="yes" />
                    </registry>
                </service>

                <service uid="menuBarView" type="::gui::aspect::SDefaultMenuBar" >
                    <gui>
                        <layout>
                            <menu name="File" />
                        </layout>
                    </gui>
                    <registry>
                        <menu sid="menuFileView" start="yes" />
                    </registry>
                </service>

                <service uid="menuFileView" type="::gui::aspect::SDefaultMenu" >
                    <gui>
                        <layout>
                            <menuItem name="Open mesh" shortcut="Ctrl+O" />
                            <separator />
                            <menuItem name="Quit" specialAction="QUIT" shortcut="Ctrl+Q" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="openMeshAct" start="yes" />
                        <menuItem sid="quitAct" start="yes" />
                    </registry>
                </service>

                <!--
                    Default view service:
                    This service defines the view layout. The type '::fwGui::CardinalLayoutManager' represents a main
                    central view and other views at the 'right', 'left', 'bottom' or 'top'.
                    Here the application contains a central view at the right.

                    Each <view> declared into the <layout> tag, must have its associated <view> into the <registry> tag.
                    A minimum window height and a width are given to the two non-central views.
                -->
                <service uid="containerView" type="::gui::view::SDefaultView" >
                    <gui>
                        <layout type="::fwGui::CardinalLayoutManager" >
                            <view align="center" />
                            <view caption="Move cameras 1,2" align="right" minWidth="600" minHeight="100" />
                            <view caption="Move camera 3" align="right" minWidth="600" minHeight="100" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="rendering1Srv" start="yes" />
                        <view sid="rendering2Srv" start="yes" />
                        <view sid="rendering3Srv" start="yes" />
                    </registry>
                </service>

                <!-- ******************************* Actions ****************************************** -->

                <service uid="openMeshAct" type="::gui::action::SStarter" >
                    <start uid="meshReaderSrv" />
                </service>

                <service uid="quitAct" type="::gui::action::SQuit" />

                <!-- ******************************* Services ***************************************** -->

                <service uid="meshReaderSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="mesh" />
                    <type mode="reader" /><!-- mode is optional (by default it is "reader") -->
                </service>

                <!--
                    Visualization services:
                    We have three rendering service representing a 3D scene displaying the loaded mesh. The scene are
                    shown in the windows defines in 'view' service.
                -->
                <service uid="rendering1Srv" type="::visuBasic::SMesh" >
                    <in key="mesh" uid="mesh" autoConnect="yes" />
                </service>

                <service uid="rendering2Srv" type="::visuBasic::SMesh" >
                    <in key="mesh" uid="mesh" autoConnect="yes" />
                </service>

                <service uid="rendering3Srv" type="::visuBasic::SMesh" >
                    <in key="mesh" uid="mesh" autoConnect="yes" />
                </service>

                <!-- ******************************* Connections ***************************************** -->

                <!--
                    Each 3D scene owns a 3D camera that can be moved by clicking in the scene.
                    - When the camera move, a signal 'camUpdated' is emitted with the new camera information (position,
                    focal, view up).
                    - To update the camera without clicking, you could call the slot 'updateCamPosition'

                    Here, we connect some rendering services signal 'camUpdated' to the others service slot
                    'updateCamPosition', so the cameras are synchronized between scenes.
                -->
                <connect>
                    <signal>rendering1Srv/camUpdated</signal>
                    <slot>rendering2Srv/updateCamPosition</slot>
                    <slot>rendering3Srv/updateCamPosition</slot>
                </connect>

                <connect>
                    <signal>rendering2Srv/camUpdated</signal>
                    <slot>rendering1Srv/updateCamPosition</slot>
                </connect>

                <!-- ******************************* Start services ***************************************** -->

                <start uid="mainFrame" />

            </config>
        </extension>

    </plugin>

You can also group the signals and all the slots together.

.. code-block:: xml

    <connect>
        <signal>rendering1Srv/camUpdated</signal>
        <signal>rendering2Srv/camUpdated</signal>
        <signal>rendering3Srv/camUpdated</signal>

        <slot>rendering1Srv/updateCamPosition</slot>
        <slot>rendering2Srv/updateCamPosition</slot>
        <slot>rendering3Srv/updateCamPosition</slot>
    </connect>

.. tip::
    You can remove a connection to see that a camera in the scene is no longer synchronized.

=========================
Signal and slot creation
=========================

---------------
*SRenderer.hpp*
---------------

.. code-block:: cpp

    // {...}

    class VISUBASIC_CLASS_API SMesh : public ::fwGui::IGuiContainerSrv
    {

    public:

        // {...}

        VISUBASIC_API static const ::fwCom::Slots::SlotKeyType s_UPDATE_CAM_POSITION_SLOT;

        VISUBASIC_API static const ::fwCom::Signals::SignalKeyType s_CAM_UPDATED_SIG;

        typedef ::fwCom::Signal< void (::fwData::TransformationMatrix3D::sptr) > CamUpdatedSignalType;

        // {...}

    private:

        // {...}

        /**
         * @brief Proposals to connect service slots to associated object signals.
         * @return A map of each proposed connection.
         * @note This is actually useless since the sub-service already listens to the data,
         * but this prevents a warning in fwServices from being raised.
         *
         * Connect ::fwData::Mesh::s_MODIFIED_SIG to s_UPDATE_SLOT
         */
        VISUBASIC_API KeyConnectionsMap getAutoConnections() const override;

        // {...}

    private:

        /// SLOT: receives new camera transform and update the camera.
        void updateCamPosition(::fwData::TransformationMatrix3D::sptr _transform);

        /// SLOT: receives new camera transform from the camera service and trigger the signal.
        void updateCamTransform();

        // {...}

        /// Contains the signal emitted when camera position is updated.
        CamUpdatedSignalType::sptr m_sigCamUpdated;

        // {...}

    };

---------------
*SRenderer.cpp*
---------------

.. code-block:: cpp

    const ::fwCom::Slots::SlotKeyType SMesh::s_UPDATE_CAM_POSITION_SLOT  = "updateCamPosition";
    static const ::fwCom::Slots::SlotKeyType s_UPDATE_CAM_TRANSFORM_SLOT = "updateCamTransform";

    const ::fwCom::Signals::SignalKeyType SMesh::s_CAM_UPDATED_SIG = "camUpdated";

    // {...}

    //------------------------------------------------------------------------------

    SMesh::SMesh() noexcept
    {
        newSlot(s_UPDATE_CAM_POSITION_SLOT, &SMesh::updateCamPosition, this);
        newSlot(s_UPDATE_CAM_TRANSFORM_SLOT, &SMesh::updateCamTransform, this);

        m_sigCamUpdated = newSignal<CamUpdatedSignalType>(s_CAM_UPDATED_SIG);
    }

    // {...}

    ::fwServices::IService::KeyConnectionsMap SMesh::getAutoConnections() const
    {
        // This is actually useless since the sub-service already listens to the data,
        // but this prevents a warning in fwServices from being raised.
        KeyConnectionsMap connections;
        connections.push(s_MESH_INPUT, ::fwData::Object::s_MODIFIED_SIG, s_UPDATE_SLOT);

        return connections;
    }

    // {...}

    void SMesh::updateCamPosition(::fwData::TransformationMatrix3D::sptr _transform)
    {
        m_cameraTransform->shallowCopy(_transform);
        m_cameraSrv->update().wait();
    }

    //------------------------------------------------------------------------------

    void SMesh::updateCamTransform()
    {
        {
            ::fwCom::Connection::Blocker block(m_sigCamUpdated->getConnection(this->slot(s_UPDATE_CAM_TRANSFORM_SLOT)));
            m_sigCamUpdated->asyncEmit(m_cameraTransform);
        }
    }

===
Run
===

To run the application, you must call the following line into the install or build directory:

.. tabs::

   .. group-tab:: Linux

        .. code::

            bin/tuto04signalslot

   .. group-tab:: Windows

        .. code::

            bin/tuto04signalslot.bat
