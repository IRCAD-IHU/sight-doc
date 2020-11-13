.. _TutorialsTuto03DataService:

************************************************
[*Tuto03DataService*] Display an image with menu
************************************************

The third tutorial is similar to the previous application, but we add gui services like menus.

.. figure:: ../media/tuto03DataService.png
    :scale: 25
    :align: center

=============
Prerequisites
=============

Before reading this tutorial, you should have seen :
 * :ref:`TutorialsTuto02DataServiceBasic`
 * :ref:`SADGUI`

=========
Structure
=========

----------------
Properties.cmake
----------------

This file describes the project information and requirements :

.. code-block:: cmake

    set( NAME Tuto03DataService )
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

        # Reader
        ioVTK

        # Services
        uiIO                    # Contains services to show dialogs for reader/writer selection
        visuBasic
    )

    moduleParam(
            appXml
        PARAM_LIST
            config
        PARAM_VALUES
            Tuto03DataService_AppCfg
    ) # Main application's configuration to launch

.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``: the file used to launch the application.

----------
plugin.xml
----------

This file is in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <plugin id="Tuto03DataService" version="@PROJECT_VERSION@" >

        <requirement id="servicesReg" />
        <requirement id="guiQt" />

        <extension implements="::fwServices::registry::AppConfig" >
            <id>Tuto03DataService_AppCfg</id>
            <config>

                <!-- ******************************* Objects declaration ****************************** -->

                <!-- The root data object in tutoDataService is a ::fwData::Image. -->
                <object uid="image" type="::fwData::Image" />

                <!-- ******************************* UI declaration *********************************** -->

                <!-- Frame service:
                    The frame creates a container for the rendering service and a menu bar.
                    In this tutorial, the gui services will automatically start the services they register using the
                    'start="yes"' attribute.
                -->
                <service uid="mainFrame" type="::gui::frame::SDefaultFrame" >
                    <gui>
                        <frame>
                            <name>Tuto03DataService</name>
                            <icon>Tuto03DataService-@PROJECT_VERSION@/tuto.ico</icon>
                            <minSize width="800" height="600" />
                        </frame>
                        <menuBar />
                    </gui>
                    <registry>
                        <menuBar sid="menuBarView" start="yes" />
                        <view sid="imageRenderSrv" start="yes" />
                    </registry>
                </service>

                <!--
                    Menu bar service:
                    This service defines the list of the menus displayed in the menu bar.
                    Here, we have only one menu: File
                    Each <menu> declared into the <layout> tag, must have its associated <menu> into the <registry> tag.
                    The <layout> tags defines the displayed information, whereas the <registry> tags defines the
                    services information.
                -->
                <service uid="menuBarView" type="::gui::aspect::SDefaultMenuBar" >
                    <gui>
                        <layout>
                            <menu name="File" />
                        </layout>
                    </gui>
                    <registry>
                        <menu sid="menuView" start="yes" />
                    </registry>
                </service>

                <!--
                    Menu service:
                    This service defines the actions displayed in the "File" menu.
                    Here, it registers two actions: "Open file", and "Quit".
                    As in the menu bar service, each <menuItem> declared into the <layout> tag, must have its
                    associated <menuItem> into the <registry> tag.

                    It's possible to associate specific attributes for <menuItem> to configure their style, shortcut...
                    In this tutorial, the attribute 'specialAction' has the value "QUIT". On MS Windows, there's no
                    impact, but on Linux this value installs the default 'Quit' system icon in the menuItem.
                -->
                <service uid="menuView" type="::gui::aspect::SDefaultMenu" >
                    <gui>
                        <layout>
                            <menuItem name="Open image" shortcut="Ctrl+O" />
                            <separator />
                            <menuItem name="Quit" specialAction="QUIT" shortcut="Ctrl+Q" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="openImageAct" start="yes" />
                        <menuItem sid="quitAct" start="yes" />
                    </registry>
                </service>

                <!-- ******************************* Actions ****************************************** -->

                <!--
                    Quit action:
                    The action service (::gui::action::SQuit) is a generic action that will close the application
                    when the user click on the menuItem "Quit".
                -->
                <service uid="quitAct" type="::gui::action::SQuit" />

                <!--
                    Open file action:
                    This service (::gui::action::SStarter) is a generic action, it starts and updates the
                    services given in the configuration when the user clicks on the action.
                    Here, the reader selector will be called when the action is clicked.
                -->
                <service uid="openImageAct" type="::gui::action::SStarter" >
                    <start uid="imageReaderSrv" />
                </service>

                <!-- ******************************* Services ***************************************** -->

                <!--
                    Reader selector dialog:
                    This is a generic service that shows a dialog to display all the readers or writers available for its
                    associated data. By default it is configured to show readers. (Note: if there is only one reading
                    service, it is directly selected without prompting.)
                    Here, the only reader available to read a ::fwData::Image is ::ioVTK::ImageReaderService (see
                    Tuto02DataServiceBasic), so the selector will not be displayed.
                    When the service is chosen, it is started, updated and stopped, so the data is read.
                -->
                <service uid="imageReaderSrv" type="::uiIO::editor::SIOSelector" >
                    <inout key="data" uid="image" />
                </service>

                <!--
                    3D visualization service of medical images:
                    Here, the service attribute 'autoConnect="yes"' allows the rendering to listen the modification of
                    the data image. So, when the image is loaded, the visualization will be updated.
                -->
                <service uid="imageRenderSrv" type="::visuBasic::SImage" >
                    <in key="image" uid="image" autoConnect="yes" />
                </service>

                <!-- ******************************* Start services ***************************************** -->

                <!--
                    Here, we only start the frame because all the others services are managed by the gui service:
                    - the frame starts the menu bar and the redering service
                    - the menu bar starts the menu services
                    - the menus starts the actions
                -->
                <start uid="mainFrame" />

            </config>
        </extension>
    </plugin>

The framework provides some gui services:

Frame (``::gui::frame::SDefaultFrame``)
    This service displays a frame and creates menu bar, tool bar and container for views, rendering service, ...

View (``::gui::view::SDefaultView``)
    This service creates sub-container and tool bar.

Menu bar (``::gui::aspect::SDefaultMenuSrv``)
    A menu bar displays menus.

Tool bar (``::gui::aspect::SDefaultToolBarSrv``)
    A tool bar displays actions, menus and editors.

Menu (``::gui::aspect::SDefaultMenuSrv``)
    A menu displays actions and sub-menus.

Action (inherited from ``::fwGui::IActionSrv`` )
    An action is a service inherited from ``::fwGui::IActionSrv``. It is called when the user clicks on the associated
    tool bar or menu.

Editors (inherited from ``::fwGui::editor::IEditor``)
    An editor is a service inherited from ``::fwGui::editor::IEditor``. It is used to creates your own gui container.

===
Run
===

To run the application, you must call the following line into the install or build directory:

.. tabs::

   .. group-tab:: Linux

        .. code::

            bin/tuto03dataservice

   .. group-tab:: Windows

        .. code::

            bin/tuto03dataservice.bat
