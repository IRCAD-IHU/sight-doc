Object-Service concept
======================

Introduction
------------

In the *Object Oriented Programming* (OOP) paradigm, an object is an instance of a class
which contains the data and the algorithms. For instance, a class Image contains the image buffer,
the size and format attributes, along with the methods to process the image,
such as reading, writing, visualizing, analyzing, etc.

This design works well, but has some drawbacks.
First the code of the class implementation can become very big
if you put everything in it, making collaborative work harder.
Then if you use different third-party libraries in your methods
(for instance DCMTK for I/O, VTK for visualization, OpenCV or ITK for the filtering methods),
your class becomes dependent of all of these libraries even if you only need one or two functionalities.
If we want something modular, that does not work.
Last, because of the two previous points, the maintenance of source code is quite tough.

Instead, Sight proposes an *Object-Service* paradigm where data and algorithms are separated into different code units.

Object
-------

Objects represent data used in the framework.
They can be simple (boolean, integer, string, etc.) or advanced structures (image, mesh, video, patient, etc.).
They are generic, which means they do not depend on the original input format or the future output format.
This way they can used with different third-party libraries,
and we provide helper methods to convert them into the corresponding formats.

These object classes contain only data features and their corresponding getter/setter methods.

For instance, the ``Image`` object:

- contains a buffer pointer, a buffer size, the image's dimension and origin,
- has public setter/getter methods to access these members,
- does not have methods such as reading or writing a buffer

The ``fwData`` library contains the standard simple and advanced data.
It is the main data library of Sight. There is also the ``fwMedData`` library which
contains several structures to store medical data.
A data list with a brief description is available in the appendixes.

Creating data
~~~~~~~~~~~~~

New data must be created as described below.

In the header file (MyData.hpp):

.. code-block:: cpp

    class MyData : public ::fwData::Object
    {

    public :
        fwCoreClassDefinitionsWithFactoryMacro( (MyData)(::fwData::Object),
            (()), ::fwData::factory::New< MyData >) ;


        // Private constructor, required for data factory
        MyData(::fwData::Object::Key key);

        /// Destructor, required for all data
        virtual ~MyData();

        /// Defines shallow copy, required for all data
        void shallowCopy( const Object::csptr& _source );

        /// Defines deep copy, required for all data
        void cachedDeepCopy(const Object::csptr& _source,
                                DeepCopyCacheType &cache);

    };

In the source file (MyData.cpp), this line must also be added to declare
``MyClass`` as data of the framework architecture :

.. code-block:: cpp

    fwDataRegisterMacro( MyData );

Service
-------

A service represents a functionality which uses or modifies data. **It
is associated with one or several objects**. For example, a service working on a
single image could be a reader, a writer, or a visualization service. A service working on two images could be a filtering service,
or a service working on a image and a mesh, a mesher.

Service type
~~~~~~~~~~~~

Some service categories exist in Sight. These categories are called *service
types* and are represented by an abstract class. The basic service types are:

- ``::fwIO::IReader``: base interface for reader services.
- ``::fwIO::IWriter``: base interface for writer services.
- ``::fwGui::IActionSrv``: base interface to manage action from a button or a
  menu in the GUI.
- ``::fwGui::editor::IEditor``:  base interface to create new widgets in the GUI.
- ``::fwRender::IRender``: base interface to create new visualization widgets in
  the GUI.
- ``::fwServices::IController``: does nothing in particular but can be considered as
  a default service type to be implemented by unclassified services.

All services require a type association and must inherit from an abstract
type service.

Service methods
~~~~~~~~~~~~~~~

Several methods exist to manipulate a service. The main methods are:
``configure``, ``start``, ``stop``, and ``update``.

- ``configure``: parses the service parameters and analyzes its
  configuration. For example, this method is used to configure an image file
  path on the file system for an image reader service.
- ``start``: initializes and launches the service (be careful,
  starting and instantiating a service is not the same thing. For
  example, for a visualization service, the ``start`` method instantiates all GUI
  widgets necessary to visualize the data but the service itself is
  instantiated before.).
- ``stop``: stops the service. For example, for a visualization
  service, this method detaches and destroys all GUI widgets previously
  instantiated earlier in the ``start`` method.
- ``update`` method is called to perform an action on the data associated with the
  service. For example, for an image reader service, the service reads the
  image, converts it and loads it into the associated data.

These methods are mandatory, but can be empty. This is because some services do
not need a configuration step, a start/stop process, or an update process.

Service states
~~~~~~~~~~~~~~

These methods must follow a calling sequence. For example, it is not possible to
stop a service before starting it. To secure the process, a state machine
has been implemented to control the calling sequence.

The calling sequence to manage a service is:

.. code-block:: cpp

    MyData::sptr myData = MyData::New();
    MyService::sptr mySrv = ::fwService::add("MyService"); // create the service
    mySrv->registerInput(myInputData, "inputData");  // register the inputs
    mySrv->registerInOut(myInOutData, "modifiedData");

    mySrv->setConfiguration( ... ); // set parameters
    mySrv->configure(); // check parameters
    mySrv->start(); // start the service
    mySrv->update(); // update the service
    mySrv->stop(); // stop the service
    ::fwServices::ORS::unregisterService(mySrv); // destroy the service

.. note::
    Sight extensively uses `std::shared_ptr <http://en.cppreference.com/w/cpp/memory/shared_ptr>`_ to handle objects
    and services. The basic declaration macros of data and services define a typedef ``sptr`` as an alias to
    ``std::shared_ptr<this_class>`` and a typedef ``csptr`` as an alias to ``std::shared_ptr<const this_class>``.

Create a service
~~~~~~~~~~~~~~~~

A new service must be created as described below (see :ref:`HowTosServiceCreation`).

In the header file (MyService.hpp):

.. code-block:: cpp

    class MyService : public AbstractServiceType
    {
    public:

        // Macro to define few important parameters/functions used by the architecture
        fwCoreServiceClassDefinitionsMacro((MyService)(AbstractServiceType));

        // Service constructor
        MyService() noexcept() ;

        // Service destructor.
        virtual ~MyService() noexcept() ;

    protected:

        // To configure the service
        void configuring() override;

        // To start the service
        void starting() override;

        // To stop the service
        void stopping() override;

        // To update the service
        void updating() override;
    };

In the source file, the following lines must also be added to declare ``MyService`` as a service of the
framework architecture:

.. code-block:: cpp

    fwServicesRegisterMacro( AbstractServiceType, MyService );
    fwServicesRegisterObjectMacro( MyService, MyData )

.. note::
    These macros can be automatically generated by cmake in the file ``registerServices.cpp``. In this case you should
    write the correct doxygen of the service XML configuration

.. note::
    When a new service is created, the following functions must be overriden
    from IService class : ``configuring``, ``starting``, ``stopping`` and
    ``updating``.  The top level functions from IService class check the
    service state before any call to the overridden method.

Object and service factories
----------------------------

To instantiate an object or a service, the architecture requires the use of a
factory system. In class-based programming, the `factory method pattern`_ is a
creational pattern which uses factory methods to deal with the problem of
creating classes without specifying the exact class that will be created. This
is done by creating classes via a factory method, which is either specified in
an interface (abstract class) and implemented in child classes (concrete
classes) or implemented in a base class (optionally as a template method),
which can be overridden when inherited in derivative classes; rather than by a
constructor.

.. _`factory method pattern`: http://en.wikipedia.org/wiki/Factory_method_pattern

Object factory
~~~~~~~~~~~~~~

The ``fwData`` library has a factory to register and create all objects.
The registration is managed by two macros:

.. code-block:: cpp

    // in .hpp file
    fwCoreClassDefinitionsWithFactoryMacro( (MyData)(::fwData::Object),
        (()), ::fwData::factory::New< MyData >);

    // in .cpp file
    fwDataRegisterMacro( MyData );

Then, there data can be instantiated in two ways:

.. code-block:: cpp

    // Direct creation
    MyData::sptr obj = MyData::New();

    // Factory creation (here obj is an object of type
    // MyData, it is then possible to cast it dynamically)
    ::fwData::Object::sptr obj = ::fwData::factory::New("MyData");
    MyData::sptr myData = MyData::dynamicCast(obj);

Service factory
~~~~~~~~~~~~~~~

The ``fwService`` library has a factory to register and create all
services. The registration is managed by two macros:

.. code-block:: cpp

    // in .hpp file
    fwCoreServiceClassDefinitionsMacro ((MyService)(AbstractServiceType));

    // in .cpp file
    fwServicesRegisterMacro( AbstractServiceType, MyService );
    fwServicesRegisterObjectMacro( MyService, MyData )

The service must be created by the factory:

.. code-block:: cpp

    ::fwServices::registry::ServiceFactory::sptr srvFactory
            = ::fwServices::registry::ServiceFactory::getDefault();

    // Factory creation (here srv is a service of type MyService, it is possible to cast it)
    ::fwServices::IService::sptr srv = srvFactory->create("MyService");


.. _OSR:

Object-Service registry (OSR)
------------------------------

The Sight architecture is standardized thanks to:

- Abstract classes ``::fwData::Object`` and ``::fwService::IService``.
- The two factory systems.

In an application, one of the problems is managing the life cycle of
a large number of object instances and their services.
This problem is solved by the class ``::fwServices::registry::ObjectService`` which maintains the relationship
between objects and services. This class concept is very simple :

.. code-block:: cpp

    // OSR is a singleton
    class ObjectService
    {
    public:
        // ...

        // Associates a service to an object
        void registerService ( ::fwData::Object::sptr obj,
                               const ::fwServices::IService::KeyType& objKey,
                               ::fwServices::IService::AccessType access,
                               ::fwServices::IService::sptr service);

        // Associates a service to an input object
        void registerServiceInput( const ::fwData::Object::csptr& object,
                                  const ::fwServices::IService::KeyType& objKey,
                                  const ::fwServices::IService::sptr& service)



        // Dissociates a service from an object
        void unregisterService ( const ::fwServices::IService::KeyType& objKey,
                                 ::fwServices::IService::AccessType access,
                                 IService::sptr service )
      // ...
    }

This registry manages the object-service relationships and guarantees
the non-destruction of an object while some services are still working on it.

Each object associated with the service must provide a **key** and an **access type**.
The **key** is used to retrieve the object in the service code, while the **access type**
tells how the object can be accessed: read, read/write or write.

Example:

.. code-block:: cpp

    ::fwData::Image::sptr image = ::fwData::Image::New();
    ::fwData::Mesh::sptr mesh = ::fwData::Mesh::New();
    ::fwServices::registry::ServiceFactory::sptr srvFactory
            = ::fwServices::registry::ServiceFactory::getDefault();

    ::fwServices::IService::sptr srv = srvFactory->create("MyService");

    ::fwServices::OSR::registerService(image, "image", ::fwServices::IService::AccessType::INOUT, srv);
    ::fwServices::OSR::registerService(mesh, "mesh", ::fwServices::IService::AccessType::INPUT, srv);

    // ....
    ::fwServices::OSR::unregisterService(srv);


To simplify, you can use an helper that calls this lines and register the inputs and inouts directly to the service:

.. code-block:: cpp

    #include <fwServices/op/Add.hpp>

    // ...
    ::fwData::Image::sptr image = ::fwData::Image::New();
    ::fwData::Mesh::sptr mesh = ::fwData::Mesh::New();
    ::fwServices::IService::sptr srv = ::fwServices::add("::myModule::MyService");
    srv->registerInOut(image, "image");
    srv->registerInput(mesh, "mesh");

    // ....
    ::fwServices::OSR::unregisterService(srv);

Object retrieval
~~~~~~~~~~~~~~~~~

We have two ways to retrieve the registered objects. We can retrieve them as a ``weak_ptr`` or as a ``locked_ptr``,
using one of the two "kind" of getter:

.. code-block :: cpp

    class IService
    {
    public:
      // ...
      ::fwData::mt::weak_ptr< CONST_DATATYPE > getWeakInput(const KeyType& key) const;
      ::fwData::mt::weak_ptr< DATATYPE > getWeakInOut(const KeyType& key) const;
      // ...
      ::fwData::mt::locked_ptr< CONST_DATATYPE > getLockedInput(const KeyType& key) const;
      ::fwData::mt::locked_ptr< DATATYPE > getLockedInOut(const KeyType& key) const;
    };

For instance, if we have a ``::fwData::Image`` registered as ``"image"`` key with ``INOUT`` access type,
and a ``::fwData::Mesh`` registered as ``"mesh"`` key with ``IN`` access
type we can retrieve there associated ``weak_ptr`` or ``locked_ptr`` in a method of the service this way:

.. code-block :: cpp

    auto mesh  = this->getLockedInput< ::fwData::Mesh >("mesh");
    auto image = this->getWeakInOut< ::fwData::Image >("image");


``weak_ptr`` or ``locked_ptr`` ?
*********************************

They both hold a pointer to the real data object, but mimic the behavior of ``std::weak_ptr``, ``std::shared_ptr``
and ``std::lock_guard``.
The purpose of this container classes, is to provide an out-of-the-box protection against concurrent access and
memory to disk dumping.

``weak_ptr`` use a hidden ``std::weak_ptr`` to point to the real data, but it can only be accessed by "locking"
the ``weak_ptr`` to get a ``locked_ptr``. The main purpose of the ``weak_ptr``, is to be used as a class member,
to spare the data lookup, or to control precisely the locking of the data. It is also the preferred way to retrieve
optional data, that are not yet registered. In that case, test for nullity the ``locked_ptr`` returned by
``weak_ptr::lock()``.

Once we have a ``locked_ptr``, we are protected from concurrent access and from dumping on disk. We can access to
the real pointer by calling ``locked_ptr::get_shared()`` or simply use ``->`` or ``*`` operators, like in:

.. code-block :: cpp

    auto weakInput   = this->getWeakInput< ::fwData::Integer >(s_INPUT);
    auto lockedInput = weakInput.lock();

    // now, the data referenced by s_INPUT is "read locked"
    // BUT beware, if the data is not yet registered, the locked_ptr can be NULL

    ....
    if( lockedInput )
    {
        const std::int64_t value = lockedInput->getValue();
        ...
    }
    ....

Or, even simpler:

.. code-block :: cpp

    auto lockedInOut = this->getLockedInOut< ::fwData::Integer >(s_INOUT);

    // after that the data referenced by s_INOUT is "write locked"
    // `getLockedInOut()` will raise an exception if the underlying data object is null.
    // If having a null object is a valid state, for instance for an optional data, please use `getWeakInOut()` instead.

    lockedInOut->setValue(666);
    ....

RAII mechanism will ensure that everything is unlocked once the ``locked_ptr`` is destroyed.
Once destroyed, you should not access anymore to the data objects as there is no more concurrent
access nor dumping protection.

.. note::

    ``locked_ptr`` are not "recursive", that means getting two ``locked_ptr`` in the same thread will
    lead to a dead lock.

Object access type
*******************

How to choose between the different access type for a given data ?

1. Read-only (*IN*)
    - If you don't modify the data and so that means you can deal with a const pointer on the data,
      then this is the right choice.

    - This also implies a "read" (or "shared") mutex with the associated ``locked_ptr``

2. Write-only (*OUT*)
    - This is a special case when the service will actually create the data.
      The data doesn't exist before the service creation.
      At some point, during ``start()``, or ``update()`` or elsewhere,
      the data is allocated, filled and registered in the OSR:

    - This also implies a "write" (or "exclusive") mutex with the associated ``locked_ptr``

.. code-block :: cpp

    ::IService::setOutput(const KeyType& key, const ::fwData::Object::sptr& object);
    //..
    ::fwData::Image::sptr image = ::fwData::Image::New();
    this->setOutput("outputImage", image);

3. Read-Write
    - The object already exists, and you need to modify it.
    - This also implies a "write" (or "exclusive") mutex with the associated ``locked_ptr``

This topic is explained more widely in the :ref:`AppConfig<SADAppConfig>` section.

.. _Object-Service_example:

Output objects
*********************************

When a service generates a new object, it can share it with other service by calling ``setOutput("<key>", <object>)``.
It can also remove the object by calling ``setOutput("<key>", nullptr)``.

.. warning::

    The service will be the maintainer of the output object: when the service is destroyed, its output
    will also be destroyed. It could be dangerous if other services work on this object. Thus, when a service is
    stopped, it must remove its outputs by calling ``setOutput("<key>", nullptr)``. The only exception of removing the
    output is when the output is extracted from an input object (and thus maintained by another service/object).

**When should I use inout or output ?**

inout
    InOuts are used when an object is accessed in reading and writing, or when it is a buffered object (array,
    image, mesh) that often changes and it would consume time by reallocating it every time.

output
    Outputs are used when the object is generated by the service, it can be a new object or an object extracted from
    an input object.


Object-Service concept example
*********************************

To conclude, the generic object-service concept is illustrated with this
example:

.. code-block:: cpp

    // Create an object
    ::fwData::Object::sptr obj = ::fwData::factory::New("::fwData::Image");

    // Create a reader and a view for this object
    ::fwServices::IService::sptr reader = ::fwServices::add("MyCustomImageReader");
    reader->registerInOut(obj, "data");
    ::fwServices::IService::sptr view = ::fwServices::add("MyCustomImageView");
    view->registerInput(obj, "object");

    // Configure and start services
    reader->setConfiguration ( /* ... */ );
    reader->configure();
    reader->start();

    view->setConfiguration ( /* ... */ );
    view->configure();
    view->start();

    // Execute services
    reader->update(); // Read image on filesystem
    view->update(); // Refresh visualization with the new image buffer

    // Stop services
    reader->stop();
    view->stop();

    // Destroy services
    ::fwServices::OSR::unregisterService(reader);
    ::fwServices::OSR::unregisterService(view);

This example shows the code to create a small application to read an image
and visualize it. You can easily transform this code to build an application
which reads and displays a 3D mesh by changing object and services
implementation strings only.

However, most applications made with Sight are not built this way. Instead, we use :ref:`AppConfig<SADAppConfig>`,
which allows to simplify the code above by a declarative approach based on XML files.
