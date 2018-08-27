XML coding 
==========

.. rule :: uid name

    ``uid`` should have a semantic name. They must be written in lower camel case. Don't prefix the uid by `UID` (like `imageUID`). Moreover, avoid ``uid`` like ``myXXXXX`` or ``customXXXXX``.
    
    .. code-block :: xml

        <object uid="image" type="::fwData::Image" />

        <service uid="imageReader" type="::ioVTK::SImageReader">
            <inout key="image" uid="image" />
        </service>
        
Parameters
----------

.. rule :: Core parameters

    These parameters are given by launcher services and are capitalized

    * ``WID_PARENT``
    * ``AS_UID``
    
.. rule :: Data parameters (object)

    Objects ``uid`` should be written in lower camel case.
    
.. rule :: Other parameters

    Non-object parameters: channel, icon path, title, ...
    written in capitals.

Configuration
-------------

.. rule :: Configuration documentation

    The configuration must be documented.
    
    .. code-block :: xml
    
        <!--
        This config opens a window containing the editors managing the ModelSeries: show/hide reconstructions, change the color, ...

        Example:
        <service uid="..." type="::fwServices::SConfigController">
            <appConfig id="ModelSeriesManagerWindow" />
            <inout key="modelSeries" uid="..." />
            <parameter replace="ICON_PATH" by="my/icon/path.ico" />
            <parameter replace="WINDOW_TITLE"  by="" />
        </service>

        Object:
        - modelSeries (mandatory): uid of the ModelSeries to manage
        Other:
        - ICON_PATH (mandatory): window icon
        - WINDOW_TITLE(optional): window title
        -->
        <extension implements="::fwServices::registry::AppConfig">
            <!-- ... -->
        </extension>

.. rule :: Order

    The configuration file must be written following this order:
    
    #. objects
    #. GUI (frame, view, toolbar, menu, menubar)
    #. actions
    #. ConfigLauncher
    #. reader/writer
    #. editors
    #. extractors
    #. listener/sender
    #. algorithms
    #. renderers
    #. connections
    #. start
    #. update
    
    Each section should begin with an XML comment.
    
    .. code-block :: xml
    
        <!-- *************************************************** begin GUI ************************************************* -->
        <!-- ... frame, view, toolbar, menu and menubar services ... -->

.. rule :: Align the xml attributes

    The XML attributes should be aligned.
    
    .. code-block :: xml
    
        <service uid="cfgNegato1" type="::fwServices::SConfigController">
            <appConfig id="3DNegatoWithAcq" />
            <inout key="imageComposite" uid="${imageComposite}" />
            <inout key="modelSeries"    uid="${modelSeries}" />
            <inout key="landmarks"      uid="${landmarks}" />
            <parameter replace="orientation"              by="axial" />
            <parameter replace="WID_PARENT"               by="view_negato1" />
            <parameter replace="patient_name"             by="${patient_name}" />
            <parameter replace="PickingChannel"           by="pickerChannel" />
            <parameter replace="CrossTypeChannel"         by="crossTypeChannel" />
            <parameter replace="setSagittalCameraChannel" by="setSagittalCameraChannel" />
            <parameter replace="setFrontalCameraChannel"  by="setFrontalCameraChannel" />
            <parameter replace="setAxialCameraChannel"    by="setAxialCameraChannel" />
        </service>
        
.. rule :: Order the objects

    The objects should be ordered by type (ref, new and deferred), and by class.
    
    .. code-block :: xml
    
        <object uid="seriesDB"        type="::fwMedData::SeriesDB" src="ref" />
        <object uid="loadingSeriesDB" type="::fwMedData::SeriesDB" src="ref" />
        <object uid="imageRef"        type="::fwData::Image"       src="ref" />
        <object uid="imageSrc"        type="::fwData::Image"       src="ref" />

        <object uid="newSeriesDB" type="::fwMedData::SeriesDB" />
        <object uid="selections"  type="::fwData::Vector" />

        <object uid="currentActivity" type="::fwMedData::ActivitySeries" src="deferred" />
        <object uid="computedImage"   type="::fwData::Image"             src="deferred" />
        
.. rule :: Comment renderers

    Each scene and its adaptors must begin with an XML comment.
    
    .. code-block :: xml
    
        <!-- ************************************************ begin 3Dscene ************************************************ -->

        <service uid="3Dscene" type="::fwRenderVTK::SRender">
            <!-- ... -->
        </service>

        <service uid="adaptor1" type="::visuVTKAdaptor::SMesh" />
        <service uid="adaptor2" type="::visuVTKAdaptor::SMesh" />
        
    The starts of these adaptors must be preceded by a comment with the scene name
    
    .. code-block :: xml
    
        <!-- ************************************************* begin start ************************************************* -->

        <start uid="frame" />

        <!-- 3DScene adaptors-->
        <start uid="adaptor1" />
        <start uid="adaptor2" />

Example
-------

    .. code-block :: xml

        <!-- ************************************************** begin data ************************************************* -->

        <object uid="image" type="::fwData::Image" />

        <!-- *************************************************** begin GUI ************************************************* -->

        <service uid="frame" type="::gui::frame::SDefaultFrame">
            <gui>
                <frame>
                    <name>Application</name>
                    <icon>Application-@PROJECT_VERSION@/tuto.ico</icon>
                </frame>
            </gui>
            <registry>
                <view sid="mainView" start="yes" />
            </registry>
        </service>

        <service uid="mainView" type="::gui::view::SDefaultView">
            <gui>
                <layout type="::fwGui::CardinalLayoutManager">
                    <view align="center" />
                    <view align="bottom" minWidth="400" minHeight="30" />
                    <view align="bottom" minWidth="40" minHeight="30" />
                </layout>
            </gui>
            <registry>
                <view sid="3DScene"        start="yes" />
                <view sid="sliceEditor"    start="yes" />
                <view sid="snapshotEditor" start="yes" />
            </registry>
        </service>

        <!-- ************************************************ begin actions ************************************************ -->

        <service uid="actionOpenImage" type="::gui::action::SSlotCaller">
            <slots>
                <slot>imageReader/update</slot>
            </slots>
        </service>

        <!-- ******************************************** begin readers/writers ******************************************** -->

        <service uid="imageReader" type="::uiIO::editor::SIOSelector">
            <inout key="data" uid="imageUID" />
            <type mode="reader" />
        </service>

        <!-- *********************************************** begin editors ************************************************* -->

        <service uid="sliceEditor" type="::uiImageQt::SliceIndexPositionEditor" autoConnect="yes">
            <inout key="image" uid="imageUID" />
            <sliceIndex>axial</sliceIndex>
        </service>

        <service uid="snapshotEditor" type="::uiVisuQt::SnapshotEditor" />
        
        <!-- ************************************************ begin 3Dscene ************************************************ -->

        <service uid="3Dscene" type="::fwRenderVTK::SRender">
            <scene>
                <picker   id="myPicker" vtkclass="fwVtkCellPicker" />
                <renderer id="default"  background="0.0" />
        
                <adaptor uid="imageAdaptor" />
                <adaptor uid="snapshotAdaptor" />
            </scene>
        </service>

        <service uid="imageAdaptor" type="::visuVTKAdaptor::SNegatoMPR" autoConnect="yes">
            <inout key="image" uid="imageUID" />
            <config renderer="default" picker="myPicker" mode="3d" slices="3" sliceIndex="axial" />
        </service>

        <service uid="snapshotAdaptor" type="::visuVTKAdaptor::SSnapshot">
            <config renderer="default" />
        </service>

        <!-- ********************************************* begin connections *********************************************** -->

        <connect>
            <signal>snapshotEditor/snapped</signal>
            <slot>snapshotAdaptor/snap</slot>
        </connect>

        <!-- ************************************************* begin start ************************************************* -->

        <start uid="frame" />
        <start uid="actionOpenImage" />

        <!-- 3DScene adaptors-->
        <start uid="imageAdaptor" />
        <start uid="snapshotAdaptor" />

        <!-- ************************************************ begin update ************************************************* -->

        <update uid="actionOpenImage" />

