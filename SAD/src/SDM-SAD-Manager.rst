Manager and updater services
==================================

In the Sight architecture, there are container objects like ``::fwData::Composite``, ``::fwData::Vector`` and
``::fwMedData::SeriesDB``. The ``::fwData::Composite`` is also an Object and represents a map
which associates a string with an Object.
The architecture provides a service to manage these objects ``::ctrlSelection::SManage``.

We also need services to extract the sub-objects from the containers
and add the object in the application configuration (AppConfig)

SManage
~~~~~~~

This service manages an object (add/swap/remove) into a container object (composite, vector, seriesDB) or into any
object's fields.

Available operations on composite are:

- Adding an object
- Swapping an object
- Removing an object
- Removing an object if present
- Adding or swapping an object

.. code-block:: xml

    <object uid="composite" type="::fwData::Composite" />
    <object uid="image" type="::fwData::Image" />

    <service uid="manager" type="::ctrlSelection::SManage">
        <inout key="object" uid="image" />
        <inout key="composite" uid="composite" />
        <compositeKey>myImage</compositeKey>
    </service>

    <!-- Add the image in the composite when it is modified -->
    <connect>
        <signal>image/modified</signal>
        <slot>manager/addOrSwap</slot>
    </connect>

    <start uid="manager" />

SObjFromSlots
~~~~~~~~~~~~~~

This service allows to add or remove an object in the :ref:`OSR` when calling the slots.

.. code-block:: xml

    <object uid="modelSeries" type="::fwMedData::ModelSeries" />
    <object uid="reconstruction" type="::fwData::Reconstruction" src="deferred" />

    <service uid="listOrganEditor" type="::uiMedDataQt::editor::SModelSeriesList" autoConnect="yes">
        <inout key="modelSeries" uid="modelSeries" />
    </service>


     <service uid="myUpdater" type="::ctrlSelection::updater::SObjFromSlot">
         <out key="object" uid="reconstruction" />
     </service>

    <!-- Add the selected reconstruction -->
    <connect>
        <signal>listOrganEditor/reconstructionSelected</signal>
        <slot>myUpdater/add</slot>
    </connect>

    <!-- Remove the reconstruction -->
    <connect>
        <signal>listOrganEditor/emptiedSelection</signal>
        <slot>myUpdater/remove</slot>
    </connect>

    <start uid="listOrganEditor" />
    <start uid="myUpdater" />

SExtractObj
~~~~~~~~~~~~

This service get objects from a source object and expose them as new objects. It uses "camp path" to extract the object
(see :ref:`campPath`).

.. code-block:: xml

    <object uid="composite" type="::fwData::Composite" />

    <object uid="image" type="::fwData::Image" src="deferred" />
    <object uid="mesh" type="::fwData::Mesh" src="deferred" />

    <service uid="extractor" type="::ctrlCamp::SExtractObj" >
       <inout key="source" uid="composite">
           <extract from="@values.myImage" />
           <extract from="@values.myMesh" />
       </inout>
       <out group="target">
           <key uid="image"/>
           <key uid="mesh"/>
       </out>
    </service>

    <start uid="extractor" />
    <update uid="extractor" />
