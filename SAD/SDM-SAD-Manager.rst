Concepts
--------

In the FW4SPL architecture, there is an object container which is often used:
``::fwData::Composite``. This container is also an Object and represents a map
which associates a string to an Object. The architecture provides two main
services to manage a Composite: a composite updater and a service manager.

Updater
~~~~~~~

The updater service extends service type ``::ctrlSelection::IUpdaterSrv`` and
work on a selection composite. This kind of service listens to specific events
from objects identified by their UID. When it receives an event, it performs an
operation on an object in the selection composite and notifies composite
listeners.

Available operations on composite are:

- Adding an object
- Swapping an object
- Removing an object
- Removing an object if present
- Adding or swapping an object
- Doing nothing

There are few generic updater services which listen all events sent by Objects,
and few other which works with particular Object events.

Manager
~~~~~~~

The manager services extend service type ``::ctrlSelection::IManagerSrv`` and
react to updater messages. This kind of service manages services on identified
data if they are present in a composite. There exist also few manager services,
but the most used is ``::ctrlSelection::manager::SwapperSrv``. This service
manages other services on objects stored in the composite. When this
manager gets notified, it can perform an action defined in the manager
configuration on concerned object such as :

- starting services of concerned object
- stopping services of concerned object
- create communication connection between new objects and/or new services

Implementation
--------------

Updater
~~~~~~~

An updater implementation must inherit of ``::ctrlSelection::IUpdaterSrv``
service.

In the example below, an updater is used to manage a
``::fwData::Reconstruction`` object identified with ``reconstruction`` key in a
selection composite. These ``::fwData::Reconstruction`` are stored in a
``::fwMedData::ModelSeries`` and we used a specific updater to listen events
and manage the structure.

It defines two scenarii, each if them belonging to a ``<update>`` XML tag:

- when the updater receives a ``NEW_RECONSTRUCTION_SELECTED`` event from the
  ``::fwMedData::ModelSeries`` object with uid ``model_uid``, it adds or swaps
  the ``rec`` object of the selection composite with object from which it
  received the event.
- when the updater receives a ``REMOVED_RECONSTRUCTIONS`` event from the
  ``::fwMedData::ModelSeries`` object with uid ``model_uid``, it removes the
  ``rec`` object of the selection composite if it is present.


Updater configuration example:

.. code:: xml

    <object id="model_uid" type="::fwMedData::ModelSeries" />

    <object type="::fwData::Composite">

      <service uid="updater_uid"
        impl="::ctrlSelection::updater::SReconstructionFromModelSeriesUpdater"
        type="::ctrlSelection::IUpdaterSrv">
        <update compositeKey="rec"
            onEvent="NEW_RECONSTRUCTION_SELECTED"
            fromUID="model_uid"
            actionType="ADD_OR_SWAP"
            />
        <update compositeKey="rec"
            onEvent="REMOVED_RECONSTRUCTIONS"
            fromUID="model_uid"
            actionType="REMOVE_IF_PRESENT"
            />
      </service>

    </object>

    <!-- connect updater to listen the model series -->
    <connect>
        <signal>model_uid/objectModified</signal>
        <slot>updater_uid/receive</slot>
    </connect>

Manager
~~~~~~~

Managers inherit from ``::ctrlSelection::IManagerSrv``. As explained earlier,
they manage tasks or services on objects which appear or disappear from the
composite on which they are working. For instance, the XML configuration below
manages a GUI to configure rendering options of a reconstruction from a reconstruction list thanks to
``::ctrlSelection::manager::SwapperSrv`` service.

In this configuration, the manager updates the services attached to ``rec``
object each time it is added, removed or swapped.

Manager configuration example

.. code:: xml

    <object type="::fwData::Composite">
      <service uid="manager_uid" impl="::ctrlSelection::manager::SwapperSrv"
            type="::ctrlSelection::IManagerSrv"
            autoConnect="yes" >
            <mode type="dummy" />
            <config>
                <object id="rec" type="::fwData::Reconstruction">
                    <service uid="organMaterialEditor"
                        impl="::uiReconstruction::OrganMaterialEditor" />
                    <service uid="representationEditor"
                        impl="::uiReconstruction::RepresentationEditor" />
                </object>
        </config>
      </service>
    </object>

.. note::
    Manager mode is *dummy* (``<mode type="dummy">``). With this configuration,
    if the ``::fwData::Reconstruction`` object is not present in the selection
    composite when the manager is starting, it will instantiate a new one. When
    the mode type is *stop*, the manager starts services when the object is
    present in the selection composite. When the mode type is *startAndUpdate*,
    the manager have the same comportment than the *stop* mode but also updates
    services


